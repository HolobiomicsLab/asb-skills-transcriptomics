---
name: gaussian-mixture-model-fitting-per-cell
description: Use when normalizing CITE-seq/ASAP-seq/TEA-seq ADT data with dsb step
  II, or when validating that k=2 Gaussian mixtures fit per-cell protein distributions
  via BIC.
when_to_use_negative:
- 'When raw ADT counts have not been ambient-corrected (step I applied): fitting mixtures
  on unstandardized raw counts conflates protein-specific ambient noise with cell-intrinsic
  noise'
- When the goal is mRNA expression normalization (e.g., CLR/LogNormalize for gene
  counts) rather than antibody-derived tag denoising
- When empty droplets or a background definition are unavailable without careful redefinition
  — the article notes defining a reasonable background subset 'still requires care'
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3318
tools:
- name: dsb
  role: Provides the step-I ambient-corrected matrix and example data (cells_citeseq_mtx,
    empty_drop_citeseq_mtx); DSBNormalizeProtein() wraps the full two-step procedure
  repo: https://cran.r-project.org/package=dsb
- name: mclust
  role: Fits the per-cell 2-component Gaussian mixture via EM; supplies BIC values
    for model selection and the fitted µ1 parameter
- name: R
  role: Language in which the per-cell fitting loop and downstream correlation/PC1
    computations are implemented
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:48822242fa70faadb44ac31627e41d41adeb1288160da0f1d0dcf904e2e03878
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - dsb
  - mclust
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# gaussian-mixture-model-fitting-per-cell

## Summary

Fits a two-component Gaussian mixture model (via mclust EM) to the dsb step-I ambient-corrected protein values of each single cell to extract the background mean µ1, which captures cell-intrinsic technical noise.

## When to use

Use when normalizing CITE-seq/ASAP-seq/TEA-seq ADT data with dsb step II, or when validating that k=2 Gaussian mixtures best fit per-cell protein distributions via BIC.

## When NOT to use

- When raw ADT counts have not been ambient-corrected (step I applied): fitting mixtures on unstandardized raw counts conflates protein-specific ambient noise with cell-intrinsic noise
- When the goal is mRNA expression normalization (e.g., CLR/LogNormalize for gene counts) rather than antibody-derived tag denoising
- When empty droplets or a background definition are unavailable without careful redefinition — the article notes defining a reasonable background subset 'still requires care'

## Inputs

- dsb step-I ambient-corrected per-cell ADT matrix (log(x+10) transformed, standardized by empty-droplet µ_N and σ_N), e.g. from dsb::cells_citeseq_mtx
- empty-droplet background matrix (dsb::empty_drop_citeseq_mtx) for step-I standardization
- isotype control rows (Mouse IgG2bk, MouseIgG1kappa, MouseIgG2akappa, RatIgG2bk; rows 67:70) for downstream technical component construction

## Outputs

- Per-cell fitted background mean µ1 (one value per single cell)
- Per-cell BIC statistics for G = 1–6 mixture components (for model-selection validation)
- Noise variable matrix of µ1 values plus isotype controls feeding the PC1 technical component

## How to apply

First compute dsb step-I values: log-transform both cell and empty-droplet ADT matrices with pseudocount 10 (log(x + 10)), then standardize each protein by the empty-droplet background mean µ_N and standard deviation σ_N. For each cell independently, fit a Gaussian mixture model with mclust::Mclust(G = 2) using the expectation-maximization algorithm, and extract the fitted background subpopulation mean µ1. Model choice is supported by BIC comparisons across G = 1–6, which showed k=2 fit best in ~81% of batch-1 cells and ~89% of cells across external CITE-seq datasets. The per-cell µ1 values are then combined with isotype control rows (e.g., rows 67:70: Mouse IgG2bk, MouseIgG1kappa, MouseIgG2akappa, RatIgG2bk) into a noise matrix whose PC1 (prcomp, scale = TRUE) is the dsb technical component λ regressed out via limma::removeBatchEffect.

## Related tools

- **dsb** (Provides the step-I ambient-corrected matrix and example data (cells_citeseq_mtx, empty_drop_citeseq_mtx); DSBNormalizeProtein() wraps the full two-step procedure) — https://cran.r-project.org/package=dsb
- **mclust** (Fits the per-cell 2-component Gaussian mixture via EM; supplies BIC values for model selection and the fitted µ1 parameter)
- **R** (Language in which the per-cell fitting loop and downstream correlation/PC1 computations are implemented)

## Examples

```
mu1 <- apply(dlog_std, 2, function(x) mclust::Mclust(x, G = 2)$parameters$mean[1])
```

## Evaluation signals

- Per-cell BIC comparison over G = 1–6 should select k=2 for the large majority of cells (paper reports ~81% in batch-1 and ~89% across external CITE-seq datasets, exceeding the >85% validation threshold in batch-1)
- Extracted µ1 values must correlate with the mean of the four isotype controls at r ≈ 0.59 across single cells (vs r ≈ 0.33 for individual isotypes)
- µ1 from k=2 fits should be highly concordant (Pearson r near 1) with µ1 from k=3 fits
- The full dsb pipeline incorporating these µ1 values should reproduce DSBNormalizeProtein() output (all.equal(denoised_adt, denoised_adt_3))

## Limitations

- Model-fit proportions and the bimodality assumption are dataset-dependent; the article notes batch normalization behavior depends on the extent of uniformity among batches
- For datasets without isotype control measurements, usage and definition of the technical component require care — caveats are deferred to the dsb documentation and Supplementary Note
- dsb normalized values are sensitive to how background droplets are defined; a reasonable background subset must be chosen carefully

## Evidence

- [discussion] dsb fits a Gaussian mixture model through the expectation-maximization algorithm implemented with the mclust51 R package: "dsb fits a Gaussian mixture model through the expectation-maximization algorithm implemented with the mclust51 R package"
- [methods] dsb fits a Gaussian mixture with 2 mixing components to model the ADT levels in each cell: "dsb fits a Gaussian mixture with 2 mixing components to model the ADT levels in each cell"
- [discussion] A noise variable matrix is then constructed by combining all the fitted µ1 values with the isotype control values for all cells.: "A noise variable matrix is then constructed by combining all the fitted µ1 values with the isotype control values for all cells. dsb then calculates principal component 1"
- [abstract] isotype control antibodies and the background protein population average in each cell exhibit significant correlations across single cells, we thus use their shared variance to correct for: "isotype control antibodies and the background protein population average in each cell exhibit significant correlations across single cells, we thus use their shared variance to correct for"
- [readme] dsb: Normalize and denoise antibody-derived-tag data from CITE-seq, ASAP-seq, TEA-seq and related assays.: "dsb: Normalize and denoise antibody-derived-tag data from CITE-seq, ASAP-seq, TEA-seq and related assays."
