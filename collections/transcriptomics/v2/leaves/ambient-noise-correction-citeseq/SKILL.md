---
name: ambient-noise-correction-citeseq
description: Use when raw ADT/protein UMI counts from droplet-based single cell assays
  (CITE-seq, ASAP-seq, TEA-seq) show substantial background noise from unbound antibodies
  and per-cell technical variation, and you need denoised, normalized protein values
  for downstream clustering or multimodal analysis.
when_to_use_negative:
- The dataset has no empty droplets and no isotype controls — use the 'no_empty_drops'
  dsb vignette workflow instead, since dsb step I requires a background definition
  and caveats for missing isotypes are deferred to the Supplementary Note
- Proteins have very low signal (maximum UMI count < 5 across all cells) and were
  not pre-filtered — these should be removed before normalization
- You need raw protein counts or a sample-barcode demultiplexing method (MULTI-seq,
  demuxlet) rather than ADT noise correction
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3168
tools:
- name: dsb
  role: Core R package implementing step I ambient correction, Gaussian-mixture background
    modeling, technical component regression via DSBNormalizeProtein() and ModelNegativeADTnorm()
  repo: https://cran.r-project.org/package=dsb
- name: mclust
  role: Fits the 2-component Gaussian mixture model by EM to each cell's transformed
    counts to estimate the background mean µ1
- name: limma
  role: Provides removeBatchEffect() for robust and efficient matrix decomposition
    of the noise matrix into technical components
- name: R
  role: Environment for the workflow; loading vectors computed with prcomp() are multiplied
    by the noise matrix
- name: Seurat
  role: 'Downstream container: dsb outputs integrate with Seurat objects for WNN clustering
    with 30 mRNA principal components'
  repo: https://github.com/satijalab/seurat-data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:ff2fbedc921177a78766e136091f7d7fbe1c77035f68bd2e1a49c092bda55d78
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
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# ambient-noise-correction-citeseq

## Summary

Two-step ADT normalization for droplet-based protein data (CITE-seq and related assays): dsb step I removes protein-specific ambient antibody noise using empty droplets, and step II models per-cell background with a Gaussian mixture to remove cell-intrinsic technical noise via a technical component regressed out of the data.

## When to use

Use when raw ADT/protein UMI counts from droplet-based single cell assays (CITE-seq, ASAP-seq, TEA-seq) show substantial background noise from unbound antibodies and per-cell technical variation, and you need denoised, normalized protein values for downstream clustering or multimodal analysis.

## When NOT to use

- The dataset has no empty droplets and no isotype controls — use the 'no_empty_drops' dsb vignette workflow instead, since dsb step I requires a background definition and caveats for missing isotypes are deferred to the Supplementary Note
- Proteins have very low signal (maximum UMI count < 5 across all cells) and were not pre-filtered — these should be removed before normalization
- You need raw protein counts or a sample-barcode demultiplexing method (MULTI-seq, demuxlet) rather than ADT noise correction

## Inputs

- Raw ADT/protein UMI count matrix for cell-containing droplets (e.g. filtered Cell Ranger barcodes)
- Raw ADT/protein UMI count matrix for background/empty droplets (raw barcodes minus cells, or dsb::empty_drop_citeseq_mtx)
- Isotype control antibody measurements (rows 67:70 in dsb example data; optional — usage without isotypes is covered in the Supplementary Note)

## Outputs

- Normalized, denoised ADT matrix (residual values after step I ambient correction and step II technical-component regression)
- Per-cell fitted background mean µ1 values and the technical component (PC1 of the noise matrix)
- Matrix compatible with Seurat, SingleCellExperiment, or AnnData containers

## How to apply

Load the raw cell protein matrix (e.g. dsb::cells_citeseq_mtx) and a background droplet matrix (dsb::empty_drop_citeseq_mtx, or all raw barcodes minus filtered cells). For step I, apply a log(x+10) transform to both matrices, then for each protein subtract the empty-droplet mean µ_N and divide by the empty-droplet standard deviation σ_N to correct ambient antibody noise. For step II, fit a 2-component Gaussian mixture (mclust EM) to each cell's step-I-corrected values and extract the background subpopulation mean µ1; combine all fitted µ1 values with isotype control values into a noise variable matrix and compute principal component 1 (the technical component). Regress this technical component out of the ambient-corrected values and return the residuals, optionally with quantile clipping (quantile.clip = c(0.001, 0.9995)). Before normalization, remove proteins with maximum raw UMI count < 5 across all cells (e.g. CD34 was removed from two datasets). The whole procedure is available through a single call to DSBNormalizeProtein(); set fast.km = TRUE for ~10-fold speedup on large datasets.

## Related tools

- **dsb** (Core R package implementing step I ambient correction, Gaussian-mixture background modeling, technical component regression via DSBNormalizeProtein() and ModelNegativeADTnorm()) — https://cran.r-project.org/package=dsb
- **mclust** (Fits the 2-component Gaussian mixture model by EM to each cell's transformed counts to estimate the background mean µ1)
- **limma** (Provides removeBatchEffect() for robust and efficient matrix decomposition of the noise matrix into technical components)
- **R** (Environment for the workflow; loading vectors computed with prcomp() are multiplied by the noise matrix)
- **Seurat** (Downstream container: dsb outputs integrate with Seurat objects for WNN clustering with 30 mRNA principal components) — https://github.com/satijalab/seurat-data

## Examples

```
install.packages('dsb'); dsb_rescaled = DSBNormalizeProtein(cell_protein_matrix = dsb::cells_citeseq_mtx, empty_drop_matrix = dsb::empty_drop_citeseq_mtx, denoise.counts = TRUE, quantile.clipping = TRUE, quantile.clip = c(0.001, 0.9995))
```

## Evaluation signals

- Per-cell background mean µ1 correlates with mean isotype control value across cells at r ≈ 0.59 (vs r ≈ 0.33 for individual isotypes; isotype-to-isotype r ≈ 0.11)
- Mean isotype value correlates with protein library size at r ≈ 0.45
- µ1 from k=2 vs k=3 Gaussian mixture fits is highly concordant (Pearson near 1), showing robustness of the background definition
- Normalized values should reveal biologically meaningful cell populations (e.g. WNN clustering with 30 mRNA PCs + dsb protein values) better than CLR normalization
- Residual output matrix has the same dimensions/proteins as the input cell matrix after removing proteins with max UMI < 5

## Limitations

- dsb normalized values are sensitive to how background droplets are defined; defining a reasonable subset of barcodes as background requires care
- Caveats for using dsb on datasets without isotype control antibody measurements are deferred to the Supplementary Note, including guidance on the technical component in such experiments
- Batch normalization behavior depends on the extent of batch uniformity
- Software version numbers are not stated in the methods section

## Evidence

- [discussion] In step I, protein counts in empty droplets are used to estimate the expected ambient background noise for each antibody: "In step I, protein counts in empty droplets are used to estimate the expected ambient background noise for each antibody"
- [discussion] Gaussian mixture model via mclust EM with k=2 components: "dsb fits a Gaussian mixture model through the expectation-maximization algorithm implemented with the mclust51 R package to the transformed count of each cell from step 1 with k = 2 mixture components"
- [discussion] Noise matrix construction and PC1 technical component: "A noise variable matrix is then constructed by combining all the fitted µ1 values with the isotype control values for all cells. dsb then calculates principal component 1"
- [abstract] Isotype controls and background mean shared variance used to correct technical noise: "isotype control antibodies and the background protein population average in each cell exhibit significant correlations across single cells, we thus use their shared variance to correct for"
- [discussion] Low-signal protein filtering before normalization: "Proteins with very low raw data signal (a maximum UMI count <5 across all cells) were removed prior to normalization, resulting in removal of the CD34 protein from two datasets."
- [discussion] Background definition sensitivity caveat: "defining a reasonable subset of barcodes as background droplets still requires care"
- [discussion] Background droplets defined from raw Cell Ranger output: "Cells were defined as barcodes in the filtered Cell Ranger output, and background drops were defined as after removing the cells from the from the raw Cell Ranger output"
- [readme] dsb available via CRAN install and vignettes: "The dsb R package is available on [**CRAN: latest dsb release**](https://CRAN.R-project.org/package=dsb)  
To install in R use `install.packages('dsb')`"
