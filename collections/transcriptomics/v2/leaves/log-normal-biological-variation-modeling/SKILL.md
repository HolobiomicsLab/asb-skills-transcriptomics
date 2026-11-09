---
name: log-normal-biological-variation-modeling
description: Use when a known-truth negative-control dataset is needed to test whether
  an scRNA-seq method (e.g., Normalisr) falsely detects co-expression or nonlinear
  library-size confounders when none exist.
when_to_use_negative:
- The simulated matrix should not be treated as real biological data — by construction
  it contains no co-expression and represents a homogeneous gRNA-free K562-like cell
  null
- Normalisr (and this null-model validation scheme) was only evaluated on UMI-based
  datasets; full-length platforms may need revalidation of the nonlinear cellular
  summary covariates
- Not suitable for hypothesis testing of arbitrary nonlinear associations, for which
  Normalisr is explicitly not designed
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Normalisr
  role: 'Framework whose null-model validation uses this simulator: the co-expression-free
    synthetic data tests detection of nonlinear confounders arising from library size
    at mean and variance levels'
  repo: github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:649374374583883a5e51e673559f749b6133e65e97df33e30b105c4af33a5c28
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Normalisr
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# log-normal-biological-variation-modeling

## Summary

Constructs a synthetic null scRNA-seq read-count matrix of homogeneous gRNA-free K562-like cells by applying log-normal biological variation to empirically sampled gene expression proportions and multinomial read sampling with sampled library sizes, yielding a co-expression-free null with ground-truth log expression proportions.

## When to use

Use when a known-truth negative-control dataset is needed to test whether an scRNA-seq method (e.g., Normalisr) falsely detects co-expression or nonlinear library-size confounders when none exist.

## When NOT to use

- The simulated matrix should not be treated as real biological data — by construction it contains no co-expression and represents a homogeneous gRNA-free K562-like cell null
- Normalisr (and this null-model validation scheme) was only evaluated on UMI-based datasets; full-length platforms may need revalidation of the nonlinear cellular summary covariates
- Not suitable for hypothesis testing of arbitrary nonlinear associations, for which Normalisr is explicitly not designed

## Inputs

- Real scRNA-seq read-count matrix (e.g., MARS-seq melanoma dataset) providing empirical library-size and gene expression-fraction distributions
- Empirical distribution of observed per-cell library sizes (sequencing depths)
- Empirical distribution of observed per-gene expression proportions

## Outputs

- synthetic_null_read_counts.csv — co-expression-free null read-count matrix (cells × genes, integer counts)
- ground_truth_log_proportions.csv — ground-truth log expression proportions, one row per simulated cell

## How to apply

Load a real scRNA-seq read-count matrix (e.g., the MARS-seq melanoma dataset used by Normalisr) as the empirical source of sequencing-depth and expression distributions. Sample per-cell total read counts from the empirical distribution of observed library sizes and per-gene read proportions from the empirical distribution of observed gene expression fractions. Apply log-normal biological variation to the sampled proportions to introduce realistic cell-to-cell variability under the homogeneous-cell null model. Generate final per-cell gene counts by multinomial read sampling, using the sampled library sizes as the number of trials and the log-normal-varied proportions as event probabilities, so that no true co-expression exists in the resulting null read-count matrix. Save the simulated counts as synthetic_null_read_counts.csv and the ground-truth log expression proportions as ground_truth_log_proportions.csv.

## Related tools

- **Normalisr** (Framework whose null-model validation uses this simulator: the co-expression-free synthetic data tests detection of nonlinear confounders arising from library size at mean and variance levels) — github.com/lingfeiwang/normalisr

## Examples

```
import numpy as np; counts = np.array([np.random.multinomial(n, p) for n, p in zip(lib_sizes, lognormal_varied_props)])
```

## Evaluation signals

- Both output files (synthetic_null_read_counts.csv and ground_truth_log_proportions.csv) exist and load as numeric matrices
- The two matrices have matching dimensions (cells × genes)
- ground_truth_log_proportions.csv has exactly one row per simulated cell
- Simulated per-gene counts are non-negative integers consistent with multinomial sampling given the sampled library sizes
- Residual co-expression estimated on the synthetic data should be consistent with the co-expression-free null (no true co-expression exists by construction)

## Limitations

- The null assumes homogeneous gRNA-free K562-like cells, so biological heterogeneity (cell states, trajectories) is deliberately absent and cannot be evaluated with this simulator
- The simulation parameters are tied to the empirical library-size and expression distributions of the chosen real dataset (e.g., MARS-seq); changing the source dataset changes the null
- Validation on full-length platforms was not performed — only UMI-based datasets were analyzed, and read-count distributions from full-length platforms may differ

## Evidence

- [other] Synthetic null generation workflow: [evidence span withheld — anchor status: not_found]
- [abstract] Nonlinear confounder detection goal: "By systematically detecting and removing nonlinear confounders arising from library size at mean and variance levels, Normalisr achieves high sensitivity, specificity, speed, and generalizability"
- [readme] Tool scope and non-parametric nature: [evidence span withheld — anchor status: external_doc]
