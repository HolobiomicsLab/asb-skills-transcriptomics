---
name: repeated-train-validation-test-split-resampling
description: Use when benchmarking predictive performance of sparse high-dimensional
  regression methods on small-n/large-p omics data, where a single train/test split
  would make MSE and feature counts unstable.
when_to_use_negative:
- When the dataset is large enough that a single train/test split gives stable estimates
  — the 10x repetition is designed for small-n microarray settings.
- When the goal is unsupervised pattern discovery (PCA/clustering) with no response
  y — this skill requires a prediction target to compute test MSE.
- When methods cannot be tuned on a validation fold (no lambda or comparable hyperparameter),
  since selection-on-validation is part of the protocol.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3518
tools:
- name: suffpcr
  role: R package providing the suffPCR() estimator (row-sparse PCA followed by OLS
    or logistic regression) and the five benchmark microarray datasets being resampled
  repo: https://github.com/dajmcdon/suffpcr
- name: R
  role: Language in which the splits are generated, models are fit, and average MSE/feature
    counts are aggregated
- name: remotes
  role: Used to install the suffpcr package from GitHub before running the resampling
    benchmark
  repo: https://github.com/dajmcdon/suffpcr
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbac033
    title: Sufficient principal component regression for pattern discovery in transcriptomic
      data
  source_content_hash: sha256:2f13647483fa9580ce7d1772dbc8ed71a0a0c5cd9d6a7bea63759df57c49a71b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dajmcdon/suffpcr
  tools:
  - R
  - remotes
  - suffpcr
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# repeated-train-validation-test-split-resampling

## Summary

Repeatedly partitions a high-dimensional microarray dataset into random train/validation/test splits so that prediction error (test MSE) and sparsity (number of selected features) of methods like suffPCR can be compared against competitors (SPC, Lasso, Elastic Net, SPCA, DSPCA) with averaged, less split-dependent results.

## When to use

Use when benchmarking predictive performance of sparse high-dimensional regression methods on small-n/large-p omics data, where a single train/test split would make MSE and feature counts unstable.

## When NOT to use

- When the dataset is large enough that a single train/test split gives stable estimates — the 10x repetition is designed for small-n microarray settings.
- When the goal is unsupervised pattern discovery (PCA/clustering) with no response y — this skill requires a prediction target to compute test MSE.
- When methods cannot be tuned on a validation fold (no lambda or comparable hyperparameter), since selection-on-validation is part of the protocol.

## Inputs

- n x p gene-expression (microarray) matrices with continuous response y, e.g. aml, dlbcl, miller, nsclc, vantveer from the suffpcr R package
- Random split specification: 10 repetitions of 40% train / 30% validation / 30% test

## Outputs

- Per-method, per-dataset average test MSE across the 10 repetitions
- Per-method, per-dataset average number of selected features
- Table 2 style benchmark summary (e.g. table2_benchmark_results.csv)

## How to apply

For each dataset (e.g., the suffpcr package microarray datasets aml, dlbcl, miller, nsclc, vantveer), generate 10 random 40/30/30 train/validation/test splits. On each split, fit every method — suffPCR with d=3 components over 5 log-spaced lambda values via suffPCR(X, y, d=1:3), plus competitors (lasso, elastic net, SPC, SPCA, DSPCA) — selecting the tuning parameter (lambda) on the validation fold only. Refit on train+validation is then possible, and test-set prediction MSE is computed on the held-out 30%. Average test MSE and average number of selected features per method per dataset across the 10 repetitions to produce a Table 2 style summary (written e.g. to table2_benchmark_results.csv). The rationale is that averaging over repeated splits removes dependence on any single partition, so the ranking pattern (e.g., SuffPCR attaining the lowest average test MSE on 4 of 5 datasets) is a robust comparison, not a split artifact.

## Related tools

- **suffpcr** (R package providing the suffPCR() estimator (row-sparse PCA followed by OLS or logistic regression) and the five benchmark microarray datasets being resampled) — https://github.com/dajmcdon/suffpcr
- **R** (Language in which the splits are generated, models are fit, and average MSE/feature counts are aggregated)
- **remotes** (Used to install the suffpcr package from GitHub before running the resampling benchmark) — https://github.com/dajmcdon/suffpcr

## Examples

```
remotes::install_github("dajmcdon/suffpcr"); library(suffpcr); out <- suffPCR(X, y, d=1:3)
```

## Evaluation signals

- Each of the 5 datasets is split exactly 10 times into 40/30/30 train/validation/test proportions.
- Lambda (or equivalent tuning parameter) is selected on the validation fold only; the test fold is untouched until final MSE computation.
- Averages are computed over all 10 repetitions per method per dataset, not a single split.
- Reproduced ranking pattern should show SuffPCR attaining the lowest average test MSE on 4 of the 5 datasets, with a comparatively small number of selected features.
- Summary is written in Table 2 style (method x dataset grid of average test MSE and selected feature counts).

## Limitations

- Results depend on the random split seeds; 10 repetitions reduce but do not eliminate split-to-split variance.
- Competitor methods must be given an equivalent tuning protocol (same splitting and validation-based lambda selection) or the comparison is unfair.
- With only 5 log-spaced lambda values and d=1:3 components, suffPCR may be under-tuned relative to a denser tuning grid.

## Evidence

- [methods] For each dataset, generate 10 random 40/30/30 train/validation/test splits.: [evidence span withheld — anchor status: not_found]
- [methods] Compute average test MSE and average number of selected features per method per dataset across the 10 repetitions.: [evidence span withheld — anchor status: not_found]
- [results] SuffPCR achieves lower average test MSE than all feature-selecting alternative methods on 4 of the 5 real microarray datasets, while selecting a comparatively small number of features.: [evidence span withheld — anchor status: not_found]
- [readme] The goal of suffpcr is to estimate sufficient principal component regression: "The goal of suffpcr is to estimate sufficient principal component regression by implementing row-sparse PCA followed by ordinary least squares or logistic regression."
