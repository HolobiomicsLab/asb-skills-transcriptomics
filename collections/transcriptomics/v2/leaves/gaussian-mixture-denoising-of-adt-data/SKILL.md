---
name: gaussian-mixture-denoising-of-adt-data
description: Use when normalizing droplet-based protein (ADT) data from CITE-seq,
  ASAP-seq, or TEA-seq where ambient antibody noise and cell-to-cell technical variation
  confound protein values and raw/CLR values track empty-droplet signal.
when_to_use_negative:
- No empty droplets are available and no background matrix can be defined — dsb requires
  a background definition and 'defining a reasonable subset of barcodes as background
  droplets still requires care'; use the dedicated no-empty-droplets variant/vignette
  instead
- The dataset lacks isotype control antibodies and the workflow has not consulted
  the Supplementary Note / documentation caveats for the technical component in such
  experiments
- The protein is essentially unexpressed in the dataset (maximum raw UMI count < 5
  across all cells, e.g. CD34 was removed in two datasets) and should be filtered
  before normalization
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3577
tools:
- name: dsb
  role: Provides DSBNormalizeProtein, which implements the ambient correction, per-cell
    2-component Gaussian mixture fit, noise matrix construction, and technical-component
    regression
  repo: https://cran.r-project.org/package=dsb
- name: mclust
  role: Fits the 2-component Gaussian mixture model via expectation-maximization to
    each cell's transformed ADT values to estimate the background mean µ1
- name: limma
  role: Its removeBatchEffect() function is used by dsb for robust and efficient matrix
    decomposition during denoising
- name: R
  role: Implementation language; loading vectors from prcomp() are multiplied by the
    noise matrix to form the technical component
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:ee15d8d60f9d23e7721ccce231d8a0b65b4037a5e53949f48115d74605221df2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - dsb
  - limma
  - mclust
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# Gaussian mixture denoising of ADT data

## Summary

dsb's step II denoising: after ambient-noise correction with empty droplets, fit a 2-component Gaussian mixture (mclust EM) to each cell's log-transformed ADT values to extract the background mean µ1, build a noise matrix from µ1 values plus isotype controls, and regress out its PC1 (the dsb technical component) to remove cell-to-cell technical noise.

## When to use

Use when normalizing droplet-based protein (ADT) data from CITE-seq, ASAP-seq, or TEA-seq where ambient antibody noise and cell-to-cell technical variation confound protein values and raw/CLR values track empty-droplet signal.

## When NOT to use

- No empty droplets are available and no background matrix can be defined — dsb requires a background definition and 'defining a reasonable subset of barcodes as background droplets still requires care'; use the dedicated no-empty-droplets variant/vignette instead
- The dataset lacks isotype control antibodies and the workflow has not consulted the Supplementary Note / documentation caveats for the technical component in such experiments
- The protein is essentially unexpressed in the dataset (maximum raw UMI count < 5 across all cells, e.g. CD34 was removed in two datasets) and should be filtered before normalization

## Inputs

- Cell-containing ADT/protein count matrix (filtered Cell Ranger barcodes)
- Background/empty-droplet ADT matrix (remaining raw barcodes with protein reads)
- Isotype control antibody measurements (optional but recommended; use.isotype.control=TRUE)

## Outputs

- Denoised, scaled ADT matrix (dsb normalized values, residuals as standard R matrix)
- Per-cell fitted background means (µ1) and dsb technical component (PC1 of the noise matrix)

## How to apply

First log-transform ambient-corrected ADT counts (log(cell + 10)) and standardize by the background mean µN and standard deviation σN estimated from empty droplets. Then, per cell, fit a 2-component Gaussian mixture model (k = 2) via EM with mclust and take the fitted background subpopulation mean µ1. Combine all fitted µ1 values with isotype control antibody values across cells into a noise variable matrix and compute PC1 (prcomp) as the dsb technical component; the returned dsb values are the residuals after regressing this technical component out of the ambient-corrected values Y. Invoke this via DSBNormalizeProtein with denoise.counts=TRUE and use.isotype.control=TRUE, supplying the empty-droplet background matrix; validate downstream by showing normalized values no longer correlate with empty-droplet percentiles (e.g., dsb r≈0.13 vs CLR r≈0.8 with the 98th-percentile empty-droplet expression in the TEA-seq MAIT case).

## Related tools

- **dsb** (Provides DSBNormalizeProtein, which implements the ambient correction, per-cell 2-component Gaussian mixture fit, noise matrix construction, and technical-component regression) — https://cran.r-project.org/package=dsb
- **mclust** (Fits the 2-component Gaussian mixture model via expectation-maximization to each cell's transformed ADT values to estimate the background mean µ1)
- **limma** (Its removeBatchEffect() function is used by dsb for robust and efficient matrix decomposition during denoising)
- **R** (Implementation language; loading vectors from prcomp() are multiplied by the noise matrix to form the technical component)

## Examples

```
dsb_rescaled <- DSBNormalizeProtein(cell_protein_matrix = cells_citeseq_mtx, empty_drop_matrix = empty_drop_citeseq_mtx, denoise.counts = TRUE, use.isotype.control = TRUE)
```

## Evaluation signals

- Median dsb-normalized protein values across cells show near-zero Pearson correlation with the 98th percentile of that protein's expression in empty droplets (expected r≈0.13, p≈0.39), whereas CLR values correlate strongly (r≈0.8, p≈2.4e-7)
- After denoising, biologically meaningful populations are unmasked: in the TEA-seq case, cluster 14 (WNN, dims.list = list(1:30, 1:18), FindClusters algorithm=3, resolution=2) is the only TCR-Va7.2-positive cluster under dsb
- Each per-cell mixture fit used exactly k = 2 mixture components, and the noise matrix contained all fitted µ1 values plus isotype controls with PC1 as the single technical component regressed out
- Output is a residual matrix of the same dimensions as the input ADT matrix, compatible with Seurat/SimpleCellExperiment/AnnData containers

## Limitations

- dsb normalized values are sensitive to how background droplets are defined; defining a reasonable subset of barcodes as background requires care
- Caveats for datasets without isotype control antibody measurements are deferred to the Supplementary Note and CRAN documentation
- Batch normalization behavior depends on the extent of batch uniformity
- Software version numbers are not stated in the paper; outlier behavior can be mitigated via quantile.clipping (quantile.clip = c(0.001, 0.9995))

## Evidence

- [discussion] dsb fits a Gaussian mixture model through the expectation-maximization algorithm implemented with the mclust51 R package to the transformed count of each cell from step 1 with k = 2 mixture components: "dsb fits a Gaussian mixture model through the expectation-maximization algorithm implemented with the mclust51 R package to the transformed count of each cell from step 1 with k = 2 mixture components"
- [discussion] A noise variable matrix is then constructed by combining all the fitted µ1 values with the isotype control values for all cells. dsb then calculates principal component 1: "A noise variable matrix is then constructed by combining all the fitted µ1 values with the isotype control values for all cells. dsb then calculates principal component 1"
- [discussion] dsb uses a function from the limma52 package removeBatchEffect() for robust and efficient matrix decomposition: "dsb uses a function from the limma52 package removeBatchEffect() for robust and efficient matrix decomposition"
- [methods] # log transformation
dlog = log(cell + 10): "dlog = log(cell + 10)"
- [discussion] Proteins with very low raw data signal (a maximum UMI count <5 across all cells) were removed prior to normalization: "Proteins with very low raw data signal (a maximum UMI count <5 across all cells) were removed prior to normalization, resulting in removal of the CD34 protein from two datasets."
- [discussion] defining a reasonable subset of barcodes as background droplets still requires care: "defining a reasonable subset of barcodes as background droplets still requires care"
- [discussion] The dsb method is implemented via the R package “dsb [https://cran.r-project.org/package=dsb]” through a single function call to DSBNormalizeProtein(): "The dsb method is implemented via the R package “dsb [https://cran.r-project.org/package=dsb]” through a single function call to DSBNormalizeProtein()"
- [methods] denoise.counts = FALSE: "denoise.counts = FALSE"
