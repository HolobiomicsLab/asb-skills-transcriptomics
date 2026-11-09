---
name: mclust-em-algorithm-parameterization
description: Use when you need to model the distribution of dsb step-I ambient-corrected
  ADT values within each single cell as a Gaussian mixture and pick the number of
  components (k) per cell via BIC.
when_to_use_negative:
- Input is raw ADT UMI counts that have not been dsb step-I corrected (log + background
  standardization) — mixture components will reflect unnormalized noise
- The goal is general clustering of cells rather than modeling the within-cell protein-level
  background/positive bimodality
edam_operation: http://edamontology.org/operation_3891
edam_topics: []
tools:
- name: mclust
  role: Fits the Gaussian mixture model via the EM algorithm for each cell and returns
    BIC values used to select the number of components
- name: dsb
  role: Provides the example CITE-seq matrices (cells_citeseq_mtx, empty_drop_citeseq_mtx)
    and the step-I correction whose outputs are modeled; uses the k=2 mixture fit
    to extract µ1
  repo: https://cran.r-project.org/package=dsb
- name: R
  role: Analysis environment for running mclust and assembling the best-fit-k proportion
    table
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:bb3de444a762626460cf3373f9b9a958225a61caad9f2f3ef6e0951f710b8be6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - dsb
  - mclust
  edam_operation: http://edamontology.org/operation_3891
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# mclust-em-algorithm-parameterization

## Summary

Fit per-cell Gaussian mixture models with the R package mclust via the EM algorithm, selecting the optimal number of mixture components (k) by the mclust BIC convention (higher is better). In the dsb workflow this determines the fitted background protein subpopulation mean (µ1) and shows k=2 best fits the majority of single cells.

## When to use

Use when you need to model the distribution of dsb step-I ambient-corrected ADT values within each single cell as a Gaussian mixture and pick the best number of components (k) per cell via BIC.

## When NOT to use

- Input is raw ADT UMI counts that have not been dsb step-I corrected (log + background standardization) — mixture components will reflect unnormalized noise
- The goal is general clustering of cells rather than modeling the within-cell protein-level background/positive bimodality

## Inputs

- dsb step-I ambient-corrected protein matrix (log-transformed with pseudocount 10, standardized by empty-droplet background mean and SD), e.g. derived from dsb::cells_citeseq_mtx and dsb::empty_drop_citeseq_mtx
- Per-cell transformed ADT count vectors

## Outputs

- Per-cell BIC values for each mixture component count k = 1..6
- Per-cell optimal k (highest BIC) assignments
- Summary table of proportions of cells where each k is optimal, written to best_fit_k_proportions.csv
- Fitted background subpopulation mean µ1 per cell (when k=2 is used for dsb step II)

## How to apply

First compute dsb step-I values by log-transforming cell protein counts with pseudocount 10 (log(cell + 10)) and standardizing each protein by the empty-droplet background mean (µ_N) and standard deviation (σ_N). Then, for each cell, fit Gaussian mixture models with mclust using the EM algorithm, sweeping G = 1 through 6 mixture components, and extract the per-k BIC (mclust convention: higher BIC is better). Determine the optimal k per cell as the model with the highest BIC; the dsb method itself fits k = 2 components per cell to extract the background mean µ1 for step II. Summarize results as the proportion of cells where k=2 is optimal and compare against the paper's ~81% (batch-1) and 89% (external CITE-seq datasets) benchmarks.

## Related tools

- **mclust** (Fits the Gaussian mixture model via the EM algorithm for each cell and returns BIC values used to select the number of components)
- **dsb** (Provides the example CITE-seq matrices (cells_citeseq_mtx, empty_drop_citeseq_mtx) and the step-I correction whose outputs are modeled; uses the k=2 mixture fit to extract µ1) — https://cran.r-project.org/package=dsb
- **R** (Analysis environment for running mclust and assembling the best-fit-k proportion table)

## Examples

```
library(mclust); for (i in 1:ncol(dlog)) { fits <- mclust::mclustBIC(t(dlog[i, , drop=FALSE]), G=1:6); bic[i, ] <- fits[1, ] }
```

## Evaluation signals

- Proportion of cells with optimal k=2 should exceed 85% (paper reports ~81% in batch-1 cells and 89% averaged across four external CITE-seq datasets)
- BIC comparison uses the mclust convention that higher BIC indicates a better fit — verify the per-k BIC matrix is assembled per cell
- Sanity-check that fitted µ1 (background component mean) from the k=2 model is lower than µ2 within each cell
- Output CSV best_fit_k_proportions.csv should contain one row per dataset with the proportion of cells where each k is optimal

## Limitations

- mclust EM fits per cell can be slow for large datasets; the dsb README notes fast.km = TRUE can speed up dsb ~10-fold
- mclust may return warning/error on cells with degenerate distributions (e.g., nearly constant values), requiring careful handling
- Software version numbers for mclust/dsb are not stated in the article

## Evidence

- [discussion] EM algorithm with mclust and k=2 per-cell mixture: "dsb fits a Gaussian mixture model through the expectation-maximization algorithm implemented with the mclust51 R package"
- [methods] k=2 mixture components in dsb step II: "dsb fits a Gaussian mixture with 2 mixing components to model the ADT levels in each cell"
- [methods] log transformation with pseudocount 10: "# log transformation
dlog = log(cell + 10)"
- [methods] Standardization by background mean and SD: "Subtract background mean $\mu_{N}$ and divide by the background standard deviation $\sigma_{N}$"
