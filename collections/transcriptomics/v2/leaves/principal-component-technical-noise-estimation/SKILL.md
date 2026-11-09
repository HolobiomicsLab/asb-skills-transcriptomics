---
name: principal-component-technical-noise-estimation
description: 'Use when normalizing CITE-seq/ASAP-seq/TEA-seq ADT counts where per-cell
  technical variation must be removed: isotype controls and the background population
  mean correlate across single cells, and that shared variance should be captured
  and regressed out.'
when_to_use_negative:
- The dataset has no isotype control antibody measurements — caveats and alternative
  definitions of the technical component are deferred to the dsb documentation and
  Supplementary Note
- The analysis goal is removing protein-specific ambient noise only — that is handled
  by dsb step I from empty droplets and does not require the PC1 technical component
- Input is raw UMI counts without prior ambient correction and log transformation
  — the mixture model and PC1 assume step-I standardized values
edam_operation: http://edamontology.org/operation_2939
edam_topics: []
tools:
- name: dsb
  role: R package whose DSBNormalizeProtein() implements the technical component regression
    as step II of the method
  repo: https://cran.r-project.org/package=dsb
- name: mclust
  role: Fits the 2-component Gaussian mixture model per cell to extract the background
    mean µ1
- name: limma
  role: Provides removeBatchEffect() used to regress the technical component out of
    the step-I matrix
- name: R
  role: prcomp() computes PC1 loading vectors applied to the noise matrix
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:470ada49b6097501b1ff8204a80224fd1ce349831f896e5d8c20c3d6c53a584d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - dsb
  - limma
  - mclust
  edam_operation: http://edamontology.org/operation_2939
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# principal-component-technical-noise-estimation

## Summary

Estimates a per-cell technical noise component in droplet-based protein (ADT) data by computing PC1 over a noise matrix combining per-cell Gaussian-mixture background means (µ1) and isotype control antibodies, as in step II of the dsb method.

## When to use

Use when normalizing CITE-seq/ASAP-seq/TEA-seq ADT counts where per-cell technical variation must be removed: isotype controls and the background population mean correlate across single cells, and that shared variance should be captured and regressed out.

## When NOT to use

- The dataset has no isotype control antibody measurements — caveats and alternative definitions of the technical component are deferred to the dsb documentation and Supplementary Note
- The analysis goal is removing protein-specific ambient noise only — that is handled by dsb step I from empty droplets and does not require the PC1 technical component
- Input is raw UMI counts without prior ambient correction and log transformation — the mixture model and PC1 assume step-I standardized values

## Inputs

- Ambient-corrected (step I) ADT matrix: log-transformed cell protein counts standardized by empty-droplet background mean and SD
- Empty droplet protein count matrix (e.g. dsb::empty_drop_citeseq_mtx)
- Isotype control antibody count rows (e.g. Mouse IgG2bk, MouseIgG1kappa, MouseIgG2akappa, RatIgG2bk)

## Outputs

- Per-cell technical component scores (PC1 of the µ1 + isotype noise matrix)
- Denoised ADT matrix: residuals after regressing the technical component out of ambient-corrected values

## How to apply

After dsb step I (ambient correction using empty droplets: standardize each cell's log(cell + 10) ADT values with per-protein background mean µ_N and SD σ_N), fit a 2-component Gaussian mixture model per cell with mclust::Mclust(G=2) and extract the background mean µ1. Build a noise matrix by binding the fitted µ1 values for all cells with the isotype control rows (e.g. Mouse IgG2bk, MouseIgG1kappa, MouseIgG2akappa, RatIgG2bk) and compute prcomp(tmat, scale = TRUE); the loading vectors times the noise matrix yield the per-cell PC1 score, the dsb technical component. Regress this component out of the ambient-corrected values using limma::removeBatchEffect(covariates = technical component); the returned residuals are the final denoised values. The rationale is that the shared variance of isotype controls and the µ1 background average reflects droplet/cell-specific noise rather than biology.

## Related tools

- **dsb** (R package whose DSBNormalizeProtein() implements the technical component regression as step II of the method) — https://cran.r-project.org/package=dsb
- **mclust** (Fits the 2-component Gaussian mixture model per cell to extract the background mean µ1)
- **limma** (Provides removeBatchEffect() used to regress the technical component out of the step-I matrix)
- **R** (prcomp() computes PC1 loading vectors applied to the noise matrix)

## Examples

```
tmat <- cbind(mu1, cells_citeseq_mtx[67:70, ]); pc <- prcomp(t(tmat), scale = TRUE); denoised <- limma::removeBatchEffect(step1_adt, covariates = pc$x[, 1])
```

## Evaluation signals

- Reproduced denoised matrix matches DSBNormalizeProtein() output: all.equal(denoised_adt, denoised_adt_3) returns TRUE
- Technical component scores correlate across single cells with both the isotype controls and the per-cell background mean µ1
- Each cell's Gaussian mixture fit converges with exactly 2 components and a valid background mean µ1
- Final residuals show reduced correlation with the technical component compared with step-I ambient-corrected values

## Limitations

- Defining a reasonable subset of barcodes as background droplets still requires care; dsb normalized values are sensitive to how background droplets are defined
- Datasets without isotype control antibody measurements require the alternative technical component definition described only in the dsb documentation and Supplementary Note
- Batch normalization behavior could be dependent on the extent of uniformity among the batches

## Evidence

- [discussion] mixture model and PC1 basis: "dsb fits a Gaussian mixture model through the expectation-maximization algorithm implemented with the mclust51 R package"
- [discussion] noise matrix and PC1: "A noise variable matrix is then constructed by combining all the fitted µ1 values with the isotype control values for all cells. dsb then calculates principal component 1"
- [discussion] regression of technical component: "the dsb technical component for each cell is then regressed out of the ambient-noise-corrected values “Y” from part 1; the values returned by dsb are the residuals"
- [abstract] shared variance rationale: "isotype control antibodies and the background protein population average in each cell exhibit significant correlations across single cells, we thus use their shared variance to correct for"
- [discussion] limma removeBatchEffect: "dsb uses a function from the limma52 package removeBatchEffect() for robust and efficient matrix decomposition"
