---
name: high-dimensional-microarray-regression-benchmarking
description: Use when a high-dimensional regression or classification task on correlated
  omics/microarray feature data needs a quantitative benchmark of sparse linear methods
  — e.g. reproducing the paper's Table 2 test-MSE and feature-count comparison across
  the five public microarray datasets.
when_to_use_negative:
- The goal is prediction from an already-fitted model or a low-dimensional (small
  p) feature table where sparse PCA offers no advantage over ordinary regression
- A single train/test evaluation is sufficient — the skill's ranking pattern is defined
  over 10 repeated 40/30/30 splits, so a one-split comparison cannot reproduce the
  paper's benchmark
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0203
tools:
- name: suffpcr
  role: Implements row-sparse PCA followed by OLS/logistic regression; provides the
    suffPCR(X, y, d=1:3) API and the five microarray datasets used as the benchmark
    inputs
  repo: https://github.com/dajmcdon/suffpcr
- name: remotes
  role: Installs the suffpcr package from GitHub for the benchmarking workflow
  repo: https://github.com/dajmcdon/suffpcr
- name: R
  role: Execution environment for running the suffpcr benchmark and competitor fits
  repo: https://github.com/dajmcdon/suffpcr
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbac033
    title: Sufficient principal component regression for pattern discovery in transcriptomic
      data
  source_content_hash: sha256:0a514ee19c052f389b6650ea4ff2baa55e179d922a73fef50d95030dff313c47
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
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# high-dimensional-microarray-regression-benchmarking

## Summary

Benchmark SuffPCR against feature-selecting sparse regression alternatives (SPC, Lasso, Elastic Net, SPCA, DSPCA) on the five suffpcr microarray datasets (aml, dlbcl, miller, nsclc, vantveer), comparing average test-set prediction MSE and number of selected features under a shared 40/30/30 train/validation/test splitting and lambda-selection protocol.

## When to use

Use when a high-dimensional regression or classification task on correlated omics/microarray feature data needs a quantitative benchmark of sparse linear methods — e.g. reproducing the paper's Table 2 test-MSE and feature-count comparison across the five public microarray datasets.

## When NOT to use

- The goal is prediction from an already-fitted model or a low-dimensional (small p) feature table where sparse PCA offers no advantage over ordinary regression
- A single train/test evaluation is sufficient — the skill's ranking pattern is defined over 10 repeated 40/30/30 splits, so a one-split comparison cannot reproduce the paper's benchmark

## Inputs

- Five microarray datasets shipped with the suffpcr package: aml, dlbcl, miller, nsclc, vantveer (high-dimensional X with correlated genes, continuous y)
- Competitor sparse regression method implementations: lasso, elastic net, SPC, SPCA, DSPCA

## Outputs

- Per-dataset, per-method average test-set MSE across 10 repetitions
- Per-dataset, per-method average number of selected features across 10 repetitions
- Table 2 style benchmark summary (e.g. table2_benchmark_results.csv)

## How to apply

Install the suffpcr package from GitHub via remotes::install_github("dajmcdon/suffpcr") and load its five microarray datasets: aml, dlbcl, miller, nsclc, and vantveer. For each dataset, generate 10 random 40/30/30 train/validation/test splits so every method is evaluated on identical partitions. On each split, fit suffPCR with d=1:3 components over 5 log-spaced lambda values via suffPCR(X, y, d=1:3), selecting lambda on the validation fold, then fit the competitor methods (lasso, elastic net, SPC, SPCA, DSPCA) under the same splitting and lambda-selection protocol. Average test MSE and average number of selected features per method per dataset across the 10 repetitions, and write a Table 2 style summary (e.g. table2_benchmark_results.csv). The benchmark is judged correct if it reproduces the paper's ranking pattern: SuffPCR attains the lowest average test MSE on 4 of the 5 datasets while selecting comparatively few features.

## Related tools

- **suffpcr** (Implements row-sparse PCA followed by OLS/logistic regression; provides the suffPCR(X, y, d=1:3) API and the five microarray datasets used as the benchmark inputs) — https://github.com/dajmcdon/suffpcr
- **remotes** (Installs the suffpcr package from GitHub for the benchmarking workflow) — https://github.com/dajmcdon/suffpcr
- **R** (Execution environment for running the suffpcr benchmark and competitor fits) — https://github.com/dajmcdon/suffpcr

## Examples

```
remotes::install_github("dajmcdon/suffpcr"); library(suffpcr); out <- suffPCR(X, y, d=1:3)
```

## Evaluation signals

- Reproduced summary attains the paper's ranking pattern: SuffPCR has lower average test MSE than all feature-selecting alternatives on 4 of the 5 datasets
- SuffPCR selects a comparatively small number of features relative to competitors
- SuffPCR is fitted with d=1:3 components over 5 log-spaced lambda values, with lambda chosen on the validation fold
- Every method is evaluated on the same 10 random 40/30/30 train/validation/test splits per dataset
- Output CSV (table2_benchmark_results.csv) contains average test MSE and average selected-feature counts per method per dataset across the 10 repetitions

## Limitations

- SuffPCR performs nearly optimally only when the model assumptions are satisfied (single-index structure with a sparse loading vector); existing sparse linear approaches can otherwise fail to select correct features or predict poorly relative to non-sparse alternatives
- The five microarray datasets (aml, dlbcl, miller, nsclc, vantveer) are fixed package datasets, so benchmark results are specific to those inputs
- Averaging over only 10 random splits means the reported ranking pattern is sensitive to split generation

## Evidence

- [results] SuffPCR achieves lower average test MSE: [evidence span withheld — anchor status: not_found]
- [readme] Package installation: "remotes::install_github("dajmcdon/suffpcr")"
- [readme] suffPCR API example: "out <- suffPCR(X, y, d=1:3)"
- [abstract] Method definition: "SuffPCR first estimates sparse principal components and then estimates a linear model on the recovered subspace."
