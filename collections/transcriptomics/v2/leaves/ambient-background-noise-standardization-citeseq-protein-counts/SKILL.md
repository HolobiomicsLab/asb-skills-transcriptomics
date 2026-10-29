---
name: ambient-background-noise-standardization-citeseq-protein-counts
description: Use when normalizing raw ADT/protein UMI matrices from CITE-seq, ASAP-seq,
  or TEA-seq droplet experiments where ambient unbound antibody noise and per-cell
  technical noise must be removed before clustering.
when_to_use_negative:
- Input is a dataset with no empty droplets available — use the dsb variant without
  empty drops (vignette 'Using dsb when empty droplets are not available') instead
  of the standard empty-droplet Step I
- Datasets lacking isotype control antibody measurements — the technical component
  definition requires caveats described in the dsb documentation and Supplementary
  Note
- Proteins with very low signal (maximum raw UMI count < 5 across all cells) should
  be removed prior to normalization rather than passed through the pipeline
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3577
tools:
- name: dsb
  role: Core R package implementing the two-step normalization via DSBNormalizeProtein();
    provides example matrices cells_citeseq_mtx and empty_drop_citeseq_mtx
  repo: https://cran.r-project.org/package=dsb
- name: mclust
  role: Fits the 2-component Gaussian mixture model per cell (Mclust(G=2)) to extract
    the background mean mu1
- name: limma
  role: removeBatchEffect() is used to regress the per-cell technical component out
    of the ambient-corrected values
- name: Seurat
  role: Downstream container; dsb output integrates with Seurat for joint mRNA+protein
    WNN clustering
  repo: https://github.com/satijalab/seurat-data
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:ae4f95b99b5073d844271027209402314ccf45b4445e61dbf8328be71f6c6415
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
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# ambient-background-noise-standardization-citeseq-protein-counts

## Summary

Two-step normalization of droplet-based protein (ADT) counts: Step I removes protein-specific ambient antibody noise using empty-droplet statistics, and Step II removes cell-intrinsic technical noise via a Gaussian-mixture-derived technical component regressed out with limma. This is the dsb method (Mulè, Martins & Tsang, Nat Commun 2022).

## When to use

Use when normalizing raw ADT/protein UMI matrices from CITE-seq, ASAP-seq, or TEA-seq droplet experiments where ambient unbound antibody noise and per-cell technical noise must be removed before clustering.

## When NOT to use

- Input is a dataset with no empty droplets available — use the dsb variant without empty drops (vignette 'Using dsb when empty droplets are not available') instead of the standard empty-droplet Step I
- Datasets lacking isotype control antibody measurements — the technical component definition requires caveats described in the dsb documentation and Supplementary Note
- Proteins with very low signal (maximum raw UMI count < 5 across all cells) should be removed prior to normalization rather than passed through the pipeline

## Inputs

- raw ADT/protein UMI count matrix for cell-containing droplets (filtered Cell Ranger barcodes)
- raw ADT count matrix for background/empty droplets (raw barcodes minus cells)
- isotype control antibody rows (e.g. Mouse IgG2bk, MouseIgG1kappa, MouseIgG2akappa, RatIgG2bk)

## Outputs

- denoised and scaled protein expression matrix (residuals after technical component regression)
- per-cell dsb technical component (PC1 of the noise variable matrix)

## How to apply

Load the cell protein matrix and the background/empty-droplet matrix (e.g. dsb::cells_citeseq_mtx and dsb::empty_drop_citeseq_mtx) and log transform both with pseudocount 10, log(cell + 10). Step I: compute per-protein mean (mu_N) and SD (sigma_N) of the log-transformed empty droplets and standardize each cell's values as (x - mu_N)/sigma_N. Step II part I: fit a 2-component Gaussian mixture model per cell with mclust::Mclust(G=2) on the step-I values and extract the background mean mu1; build a noise matrix by binding mu1 with the isotype control rows (e.g. Mouse IgG2bk, MouseIgG1kappa, MouseIgG2akappa, RatIgG2bk) and take PC1 from prcomp(tmat, scale=TRUE) as the per-cell technical component. Regress this technical component out of the step-I matrix using limma::removeBatchEffect(covariates = technical component) to obtain the final denoised residuals; optionally apply quantile clipping with quantile.clip = c(0.001, 0.9995). Define cells as barcodes in the filtered Cell Ranger output and background drops as the remaining raw barcodes with protein reads.

## Related tools

- **dsb** (Core R package implementing the two-step normalization via DSBNormalizeProtein(); provides example matrices cells_citeseq_mtx and empty_drop_citeseq_mtx) — https://cran.r-project.org/package=dsb
- **mclust** (Fits the 2-component Gaussian mixture model per cell (Mclust(G=2)) to extract the background mean mu1)
- **limma** (removeBatchEffect() is used to regress the per-cell technical component out of the ambient-corrected values)
- **Seurat** (Downstream container; dsb output integrates with Seurat for joint mRNA+protein WNN clustering) — https://github.com/satijalab/seurat-data

## Examples

```
denoised_adt <- DSBNormalizeProtein(cell_protein_matrix = cells_citeseq_mtx, empty_drop_matrix = empty_drop_citeseq_mtx, denoise.counts = TRUE)
```

## Evaluation signals

- Reconstructed output matches DSBNormalizeProtein() output, verified with all.equal(denoised_adt, denoised_adt_3)
- Step-I standardized values have each protein centered at the empty-droplet mean mu_N and scaled by sigma_N
- Technical component PC1 shows significant correlation with isotype control antibodies and per-cell background protein population average
- Final denoised values are residuals with the technical component regressed out; downstream clustering better unmasks biologically meaningful cell populations compared to CLR normalization

## Limitations

- dsb normalized values are sensitive to how background droplets are defined; defining a reasonable subset of barcodes as background requires care
- Caveats for using dsb on datasets without isotype control antibody measurements are deferred to the Supplementary Note and CRAN documentation
- Proteins with maximum raw UMI count < 5 across all cells (e.g. CD34 in two datasets) must be removed prior to normalization
- Standard scRNAseq QC on mRNA content, mitochondrial read proportion and protein library size should be tuned per dataset before normalization

## Evidence

- [discussion] Step I ambient correction with empty droplets: "In step I, protein counts in empty droplets are used to estimate the expected ambient background noise for each antibody"
- [discussion] Gaussian mixture per cell: "dsb fits a Gaussian mixture model through the expectation-maximization algorithm implemented with the mclust51 R package to the transformed count of each cell from step 1 with k = 2 mixture components"
- [discussion] Noise matrix and PC1: "A noise variable matrix is then constructed by combining all the fitted µ1 values with the isotype control values for all cells. dsb then calculates principal component 1"
- [discussion] limma regression of technical component: "the dsb technical component for each cell is then regressed out of the ambient-noise-corrected values “Y” from part 1; the values returned by dsb are the residuals"
- [methods] Log transformation pseudocount: "# log transformation
dlog = log(cell + 10)"
- [discussion] Background droplet definition caveat: "defining a reasonable subset of barcodes as background droplets still requires care"
- [discussion] Low-signal protein filter: "Proteins with very low raw data signal (a maximum UMI count <5 across all cells) were removed prior to normalization, resulting in removal of the CD34 protein from two datasets."
- [abstract] Shared variance of isotypes and background mean: "isotype control antibodies and the background protein population average in each cell exhibit significant correlations across single cells, we thus use their shared variance to correct for"
