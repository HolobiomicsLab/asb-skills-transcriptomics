---
name: penalized-regression-competitor-evaluation
description: Use when evaluating a high-dimensional regression method (e.g. SuffPCR)
  against feature-selecting penalized competitors on transcriptomic data where correlated
  features and small selected-feature counts matter.
when_to_use_negative:
- The goal is method development or new theory, not head-to-head benchmarking of feature-selecting
  sparse regression methods on real datasets
- Data are not high-dimensional regression/classification tasks with correlated omics
  features — e.g. low-dimensional data where OLS suffices
- Only a single train/test split is available, so validation-fold lambda selection
  at the 40/30/30 protocol cannot be implemented reproducibly
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: suffpcr
  role: Method under evaluation; fits row-sparse PCA followed by OLS/logistic regression
    via suffPCR(X, y, d=1:3), and supplies the five benchmark datasets
  repo: https://github.com/dajmcdon/suffpcr
- name: remotes
  role: Installs the suffpcr package from GitHub to run the benchmark and load its
    data
  repo: https://github.com/dajmcdon/suffpcr
- name: R
  role: Environment in which the splits, suffPCR fits, competitor fits, and MSE/feature-count
    aggregation are executed
  repo: https://github.com/dajmcdon/suffpcr
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbac033
    title: Sufficient principal component regression for pattern discovery in transcriptomic
      data
  source_content_hash: sha256:e3a6db66d8c69d88f54e2732d540b2421aa535cba2626412449fbbcadf330572
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
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# penalized-regression-competitor-evaluation

## Summary

Benchmarks SuffPCR (row-sparse PCA + OLS/logistic regression) against feature-selecting sparse regression alternatives (SPC, Lasso, Elastic Net, SPCA, DSPCA) on high-dimensional microarray datasets using a shared 40/30/30 train/validation/test splitting and validation-fold lambda-selection protocol, to compare average test MSE and number of selected features.

## When to use

Use when evaluating a high-dimensional regression method (e.g. SuffPCR) against feature-selecting penalized competitors on transcriptomic data where correlated features and small selected-feature counts matter.

## When NOT to use

- The goal is method development or new theory, not head-to-head benchmarking of feature-selecting sparse regression methods on real datasets
- Data are not high-dimensional regression/classification tasks with correlated omics features — e.g. low-dimensional data where OLS suffices
- Only a single train/test split is available, so validation-fold lambda selection at the 40/30/30 protocol cannot be implemented reproducibly

## Inputs

- suffpcr microarray datasets: aml, dlbcl, miller, nsclc, vantveer
- Feature matrix X (n x p, p >> n gene expression) and response vector y for each dataset
- Random-split specification: 40/30/30 train/validation/test, 10 repetitions per dataset

## Outputs

- Per-method, per-dataset average test MSE across 10 repetitions
- Per-method, per-dataset average number of selected features
- Table 2 style benchmark summary (e.g. table2_benchmark_results.csv)

## How to apply

Install the suffpcr package with remotes::install_github("dajmcdon/suffpcr") and load the five bundled microarray datasets: aml, dlbcl, miller, nsclc, and vantveer. For each dataset, generate 10 random 40/30/30 train/validation/test splits. On each split, fit suffPCR with d=1:3 components over a grid of log-spaced lambda values (e.g. 5 values), selecting lambda on the validation fold; run the competitor methods (lasso, elastic net, SPC, SPCA, DSPCA) under the identical splitting and lambda-selection protocol so comparisons are fair. Average test MSE and the average number of selected features across the 10 repetitions per method per dataset, then write the Table 2 style summary (e.g. to table2_benchmark_results.csv). Validation criterion: the reproduced ranking should show SuffPCR attaining the lowest average test MSE on 4 of the 5 datasets while selecting comparatively few features.

## Related tools

- **suffpcr** (Method under evaluation; fits row-sparse PCA followed by OLS/logistic regression via suffPCR(X, y, d=1:3), and supplies the five benchmark datasets) — https://github.com/dajmcdon/suffpcr
- **remotes** (Installs the suffpcr package from GitHub to run the benchmark and load its data) — https://github.com/dajmcdon/suffpcr
- **R** (Environment in which the splits, suffPCR fits, competitor fits, and MSE/feature-count aggregation are executed) — https://github.com/dajmcdon/suffpcr

## Examples

```
library(suffpcr); out <- suffPCR(X, y, d=1:3)
```

## Evaluation signals

- SuffPCR attains the lowest average test MSE on 4 of the 5 datasets (aml, dlbcl, miller, nsclc, vantveer), matching the paper's Table 2 ranking pattern
- SuffPCR's average number of selected features is comparatively small relative to feature-selecting alternatives
- All methods use identical 10-repetition 40/30/30 splits and validation-fold lambda selection, so per-dataset averages are computed over the same 10 repetitions
- Output CSV (table2_benchmark_results.csv) contains both metrics (MSE and feature count) for every method-dataset pair
- suffPCR calls succeed with d=1:3 components over 5 log-spaced lambda values without errors across all splits

## Limitations

- Existing sparse linear approaches can fail to select the correct features, predict poorly relative to non-sparse alternatives, or ignore unknown feature groupings for the features
- Near-optimal performance holds 'when the model assumptions are satisfied'; deviations from the assumed low-rank sparse structure may degrade SuffPCR's advantage
- Benchmark conclusions are specific to the five microarray datasets and the 10-repetition 40/30/30 protocol; other datasets or split protocols may change the ranking

## Evidence

- [results] SuffPCR achieves lower average test MSE than all feature-selecting alternative methods on 4 of the 5 real microarray datasets, while selecting a comparatively small number of features.: [evidence span withheld — anchor status: not_found]
- [readme] suffPCR API and data: "The goal of suffpcr is to estimate sufficient principal component regression by implementing row-sparse PCA followed by ordinary least squares or logistic regression."
- [methods] Installation method: "remotes::install_github("dajmcdon/suffpcr")"
- [abstract] SuffPCR scope and competitor limitations: "These methods can fail to select the correct features, predict poorly relative to non-sparse alternatives or ignore any unknown grouping structures for the features."
- [abstract] Near-optimality caveat: "SuffPCR works well on a variety of simulated and experimental transcriptomic data, performing nearly optimally when the model assumptions are satisfied."
