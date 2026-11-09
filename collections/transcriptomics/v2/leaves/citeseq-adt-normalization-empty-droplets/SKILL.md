---
name: citeseq-adt-normalization-empty-droplets
description: Use when normalizing droplet-based single-cell protein (ADT) counts —
  e.g. CITE-seq, ASAP-seq, TEA-seq — where raw and filtered Cell Ranger matrices are
  available so empty droplets can estimate ambient antibody background.
when_to_use_negative:
- Empty droplets are not available (no raw matrix) — use the alternative dsb workflow/vignette
  'Using dsb when empty droplets are not available' instead of the empty-droplet background
  definition
- The dataset lacks isotype control antibody measurements — caveats on defining the
  technical component are deferred to the dsb documentation and Supplementary Note;
  do not blindly apply the default denoising
- The data are already normalized ADT values (e.g. CLR-normalized) rather than raw
  UMI counts — dsb expects raw ADT counts
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: dsb
  role: Core normalization/denoising via a single call to DSBNormalizeProtein(), performing
    background subtraction and technical-component regression
  repo: https://cran.r-project.org/package=dsb
- name: mclust
  role: Fits the per-cell 2-component Gaussian mixture model (expectation-maximization)
    to extract the background mean mu1
- name: limma
  role: Provides removeBatchEffect() used for robust and efficient regression of the
    dsb technical component
- name: Seurat
  role: Downstream clustering and RunUMAP embedding of dsb-normalized protein values;
    dsb interfaces easily with Seurat
  repo: https://github.com/satijalab/seurat-data
- name: R
  role: Implementation environment; prcomp() computes loading vectors for the noise
    matrix technical component
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:e57b7ce6a18555402fcf62b9243d5cdb64688772f9652d8416e8c1f1d831774e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - dsb
  - limma
  - mclust
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# citeseq-adt-normalization-empty-droplets

## Summary

Normalize and denoise CITE-seq ADT (protein) counts with the dsb two-step method: remove ambient antibody noise using empty-droplet background, then regress out droplet/cell-specific noise captured by a 2-component Gaussian mixture background mean plus isotype controls. This yields biologically interpretable protein values suitable for clustering and biaxial plots.

## When to use

Use when normalizing droplet-based single-cell protein (ADT) counts — e.g. CITE-seq, ASAP-seq, TEA-seq — where raw and filtered Cell Ranger matrices are available so empty droplets can estimate ambient antibody background.

## When NOT to use

- Empty droplets are not available (no raw matrix) — use the alternative dsb workflow/vignette 'Using dsb when empty droplets are not available' instead of the empty-droplet background definition
- The dataset lacks isotype control antibody measurements — caveats on defining the technical component are deferred to the dsb documentation and Supplementary Note; do not blindly apply the default denoising
- The data are already normalized ADT values (e.g. CLR-normalized) rather than raw UMI counts — dsb expects raw ADT counts

## Inputs

- Raw Cell Ranger UMI matrix (Gene Expression + Antibody Capture) containing empty droplets
- Filtered Cell Ranger UMI matrix (Gene Expression + Antibody Capture) defining cell barcodes
- Empty-droplet ADT count matrix (e.g. dsb::empty_drop_citeseq_mtx as example)
- Isotype control antibody measurements (for the technical component)
- List of target protein/ADT feature names

## Outputs

- dsb-normalized, denoised ADT matrix (residuals after technical component regression)
- Per-cell fitted background means (mu1) usable for noise-model correlation checks
- Seurat clusters and UMAP embedding computed on dsb-normalized protein values
- Biaxial plots (e.g. CD14 vs CD4) on normalized values
- Summary metrics: Cor 1 (mu1 vs isotype) and Cor 2 (mu1 vs library size lambda)

## How to apply

Define cells as barcodes present in the filtered Cell Ranger output and background droplets as the remaining raw-output barcodes with ADT reads, retaining a large set of empty droplets (>50,000 in the external 10X NextGem ~5K-cell 29-protein replication). Call DSBNormalizeProtein with dsb defaults: log(cell + 10) transformation, subtraction of background mean mu_N and SD sigma_N estimated from empty droplets (step I), then step II fitting a 2-component Gaussian mixture (mclust) per cell to extract the background mean mu1. Build a noise matrix of fitted mu1 values plus isotype controls, take PC1 via prcomp as the technical component, and regress it out with limma::removeBatchEffect; the returned residuals are the denoised values. Optionally remove proteins with maximum raw UMI count <5 across all cells (as done for CD34 in two datasets), and optionally apply quantile.clipping with quantile.clip = c(0.001, 0.9995). Feed the normalized matrix into Seurat for clustering and RunUMAP, and validate by checking that mu1 correlates positively with isotype levels and with protein library size lambda.

## Related tools

- **dsb** (Core normalization/denoising via a single call to DSBNormalizeProtein(), performing background subtraction and technical-component regression) — https://cran.r-project.org/package=dsb
- **mclust** (Fits the per-cell 2-component Gaussian mixture model (expectation-maximization) to extract the background mean mu1)
- **limma** (Provides removeBatchEffect() used for robust and efficient regression of the dsb technical component)
- **Seurat** (Downstream clustering and RunUMAP embedding of dsb-normalized protein values; dsb interfaces easily with Seurat) — https://github.com/satijalab/seurat-data
- **R** (Implementation environment; prcomp() computes loading vectors for the noise matrix technical component)

## Examples

```
dsb_rescaled = DSBNormalizeProtein(cell_protein_matrix = cells_citeseq_mtx, empty_drop_matrix = empty_drop_citeseq_mtx)
```

## Evaluation signals

- The fitted background mean mu1 (cellwise_background_mean) shows a significant positive correlation with isotype control levels (Cor 1) across single cells
- The fitted background mean mu1 shows a significant positive correlation with protein library size lambda (Cor 2), replicating the noise-model assumptions (Fig. 2i)
- Clustering and UMAP on dsb-normalized protein values recover biologically interpretable populations, and biaxial plots (e.g. CD14 vs CD4) match the expected structure
- Proteins with maximum raw UMI count <5 across all cells were removed prior to normalization (e.g. CD34)
- Background droplet set is large and derived from the EmptyDrops/Cell Ranger barcode rank (e.g. >50,000 empty droplets retained)

## Limitations

- dsb normalized values are sensitive to how background droplets are defined; defining a reasonable subset of barcodes as background droplets still requires care
- Caveats for using dsb on datasets without isotype control antibody measurements are deferred to the Supplementary Note and CRAN documentation
- Batch normalization behavior depends on the extent of batch uniformity among batches
- Software version numbers for tools are not stated in the publication; antibody list is only in supplementary Data 1

## Evidence

- [methods] log transformation of ADT counts: "dlog = log(cell + 10)"
- [methods] standardization by background statistics: "Subtract background mean $\mu_{N}$ and divide by the background standard deviation $\sigma_{N}$"
- [discussion] Gaussian mixture background mean: "dsb fits a Gaussian mixture model through the expectation-maximization algorithm implemented with the mclust51 R package"
- [discussion] technical component definition: "A noise variable matrix is then constructed by combining all the fitted µ1 values with the isotype control values for all cells. dsb then calculates principal component 1"
- [discussion] cell/background definition: "Cells were defined as barcodes in the filtered Cell Ranger output, and background drops were defined as after removing the cells from the from the raw Cell Ranger output"
- [discussion] low-signal protein filter: "Proteins with very low raw data signal (a maximum UMI count <5 across all cells) were removed prior to normalization, resulting in removal of the CD34 protein from two datasets."
- [readme] repository availability: "The dsb R package is available on [**CRAN: latest dsb
release**](https://CRAN.R-project.org/package=dsb)"
- [readme] ambient background rationale: [evidence span withheld — anchor status: external_doc]
