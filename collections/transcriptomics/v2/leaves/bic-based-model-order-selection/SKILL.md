---
name: bic-based-model-order-selection
description: Use when single-cell CITE-seq / ADT protein count distributions (after
  dsb step-I log(pseudocount 10) transformation and empty-droplet standardization)
  must be modeled with Gaussian mixtures and the number of mixture components needs
  a principled, per-cell choice.
when_to_use_negative:
- Input is already fully dsb-denoised residual output (step-II output with technical
  component regressed out) rather than step-I ambient-corrected values — the mixture
  model is defined on the step-I transformed count of each cell.
- No empty droplets are available to define the background mean and standard deviation
  needed for step-I standardization, unless using the documented no-empty-drops variant.
- The goal is a single global distribution across cells (e.g., population-level clustering)
  rather than modeling the within-cell protein count distribution.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: dsb
  role: Provides the step-I ambient correction (DSBNormalizeProtein via empty-droplet
    background mean/sd) that produces the distributions being modeled
  repo: https://cran.r-project.org/package=dsb
- name: mclust
  role: Fits per-cell Gaussian mixture models via the EM algorithm over G = 1–6 and
    returns BIC values for model-order selection
- name: R
  role: Implementation environment for the dsb and mclust workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:4ac218cfe1ba8ca5c577553500f5e52db1925dd49929d489798ebf77d36587a2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - dsb
  - mclust
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# BIC-based model order selection

## Summary

Selects the optimal number of Gaussian mixture components (k) for modeling the distribution of dsb step-I ambient-corrected ADT values within each single cell by sweeping G = 1–6 with mclust and comparing BIC values per cell.

## When to use

Use when single-cell CITE-seq / ADT protein count distributions (after dsb step-I log(pseudocount 10) transformation and empty-droplet standardization) must be modeled with Gaussian mixtures and the number of mixture components needs a principled, per-cell choice.

## When NOT to use

- Input is already fully dsb-denoised residual output (step-II output with technical component regressed out) rather than step-I ambient-corrected values — the mixture model is defined on the step-I transformed count of each cell.
- No empty droplets are available to define the background mean and standard deviation needed for step-I standardization, unless using the documented no-empty-drops variant.
- The goal is a single global distribution across cells (e.g., population-level clustering) rather than modeling the within-cell protein count distribution.

## Inputs

- dsb step-I ambient-corrected ADT matrix derived from dsb::cells_citeseq_mtx (cells x proteins, log-transformed with pseudocount 10 and standardized by empty-drop background mean/sd)
- dsb::empty_drop_citeseq_mtx (empty-droplet protein matrix used to compute background mean and sd per antibody)

## Outputs

- Per-cell per-k BIC table (cells x G=1..6) from mclustBIC
- Per-cell optimal-k assignments (model with highest BIC)
- best_fit_k_proportions.csv: summary table of proportions of cells in which each k (especially k = 2) is optimal

## How to apply

First compute dsb step-I values: log-transform cell ADT counts with pseudocount 10 (dlog = log(cell + 10)), then standardize each protein by the empty-droplet background mean and standard deviation. For each cell independently, fit Gaussian mixture models via the EM algorithm with mclust over G = 1 through 6 mixing components, and record each model's BIC (mclust convention: higher BIC is better). The optimal k per cell is the model with the highest BIC; the dsb method itself fixes k = 2 to extract the background protein subpopulation mean mu1, and the paper justifies this because k = 2 had the best BIC fit in the majority of cells (81% in batch-1; ~89% average across four external CITE-seq datasets). Finally, tabulate the proportion of cells for which each k is optimal and compare against the expected dominance of k = 2.

## Related tools

- **dsb** (Provides the step-I ambient correction (DSBNormalizeProtein via empty-droplet background mean/sd) that produces the distributions being modeled) — https://cran.r-project.org/package=dsb
- **mclust** (Fits per-cell Gaussian mixture models via the EM algorithm over G = 1–6 and returns BIC values for model-order selection)
- **R** (Implementation environment for the dsb and mclust workflow)

## Examples

```
library(mclust); BIC <- mclustBIC(t(dlog_scaled), G = 1:6); best_k <- apply(BIC, 1, function(b) which.max(b))
```

## Evaluation signals

- Proportion of cells whose highest-BIC model is k = 2 should exceed 85% (paper reports 81% in batch-1 cells and 89% average across four external CITE-seq datasets)
- Per-cell BIC matrix must cover all candidate orders G = 1 through 6 fitted by mclust EM; missing cells or partial G coverage indicates incorrect fitting
- Per-dataset proportions of optimal k must sum to 1 (each cell contributes exactly one best-fit k)
- Remember mclust BIC convention: higher BIC is better — argmax, not argmin, selects the optimal k
- Step-I values used as model input must reflect log(cell + 10) transformation followed by subtraction of background mean and division by background standard deviation per protein

## Limitations

- The k = 2 justification is empirical: a minority of cells (up to ~19% in batch-1) prefer other orders, so a fixed k = 2 (as in dsb step II) is an approximation per cell
- Software version numbers for dsb/mclust are not stated in the paper, so exact BIC reproducibility may depend on package versions
- The distribution being modeled depends on how background droplets are defined — 'defining a reasonable subset of barcodes as background droplets still requires care'

## Evidence

- [methods] dsb fits a Gaussian mixture with 2 mixing components to model the ADT levels in each cell: "dsb fits a Gaussian mixture with 2 mixing components to model the ADT levels in each cell"
- [discussion] mclust EM with k = 2 on step-I transformed counts: "dsb fits a Gaussian mixture model through the expectation-maximization algorithm implemented with the mclust51 R package"
- [discussion] Step I defines the ambient background being standardized against: "In step I, protein counts in empty droplets are used to estimate the expected ambient background noise for each antibody"
- [methods] Step-I log transformation with pseudocount 10: "# log transformation
dlog = log(cell + 10)"
- [readme] dsb scope: CITE-seq and related ADT assays: "dsb: Normalize and denoise antibody-derived-tag data from CITE-seq, ASAP-seq, TEA-seq and related assays."
