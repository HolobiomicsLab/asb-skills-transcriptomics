---
name: test-mse-and-feature-count-aggregation
description: Use when comparing feature-selecting sparse regression methods on high-dimensional
  omics data and you need a reproducible summary of average test MSE and selected
  feature counts across repeated random splits.
when_to_use_negative:
- The goal is feature selection or subspace recovery on a single dataset without a
  benchmarking comparison — use suffPCR directly (suffPCR(X, y, d=1:3)) rather than
  the aggregation protocol
- Data are low-dimensional or features are uncorrelated, where sparse PCA-based regression
  offers no advantage over standard regression
- You lack a held-out test fold — averaging test MSE requires a 40/30/30-style train/validation/test
  split; validation-fold lambda selection plus test-fold evaluation is essential
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: suffpcr
  role: Fit suffPCR (row-sparse PCA followed by OLS or logistic regression) on each
    training fold; provides the aml, dlbcl, miller, nsclc, vantveer microarray datasets
    being benchmarked
  repo: https://github.com/dajmcdon/suffpcr
- name: R
  role: Runtime environment for the benchmark script and aggregation of test MSE /
    feature-count summaries
  repo: https://github.com/dajmcdon/suffpcr
- name: remotes
  role: Install the suffpcr package from GitHub into the R environment
  repo: https://github.com/dajmcdon/suffpcr
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbac033
    title: Sufficient principal component regression for pattern discovery in transcriptomic
      data
  source_content_hash: sha256:c460e5201dc32e5ad241eeef8c5cbae950c08766ce21f873e36b0f1e1e548882
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# test-mse-and-feature-count-aggregation

## Summary

Benchmark protocol for evaluating sparse regression methods (SuffPCR vs. SPC, Lasso, Elastic Net, SPCA, DSPCA) on high-dimensional microarray data by averaging test-set prediction MSE and the number of selected features across repeated random train/validation/test splits, as in Table 2 of the SuffPCR paper.

## When to use

Use when comparing feature-selecting sparse regression methods on high-dimensional omics data and you need a reproducible summary of average test MSE and selected feature counts across repeated random splits.

## When NOT to use

- The goal is feature selection or subspace recovery on a single dataset without a benchmarking comparison — use suffPCR directly (suffPCR(X, y, d=1:3)) rather than the aggregation protocol
- Data are low-dimensional or features are uncorrelated, where sparse PCA-based regression offers no advantage over standard regression
- You lack a held-out test fold — averaging test MSE requires a 40/30/30-style train/validation/test split; validation-fold lambda selection plus test-fold evaluation is essential

## Inputs

- Five microarray datasets shipped with the suffpcr package: aml, dlbcl, miller, nsclc, vantveer
- High-dimensional feature matrix X (genes/features by samples) and response vector y (regression or classification target)
- Fit objects from suffPCR and competitor sparse regression methods (lasso, elastic net, SPC, SPCA, DSPCA) fit under a common split/lambda-selection protocol

## Outputs

- Per-method, per-dataset average test-set prediction MSE across 10 repetitions
- Per-method, per-dataset average number of selected features across 10 repetitions
- Table 2-style benchmark summary (e.g., table2_benchmark_results.csv)

## How to apply

Install the suffpcr package from GitHub with remotes::install_github("dajmcdon/suffpcr") and load the five packaged microarray datasets (aml, dlbcl, miller, nsclc, vantveer). For each dataset, generate 10 random 40/30/30 train/validation/test splits; on each split fit suffPCR with d=1:3 components over a grid of log-spaced lambda values (e.g., 5 values), selecting lambda on the validation fold. Fit each competitor sparse regression method (lasso, elastic net, SPC, SPCA, DSPCA) under the identical split and lambda-selection protocol so comparisons are fair. For each method and dataset, average test MSE and the number of selected features over the 10 repetitions and write a Table 2-style summary (e.g., to CSV). Apply the skill correctly when the aggregated ranking reproduces the paper's pattern: SuffPCR attains the lowest average test MSE on 4 of the 5 datasets while selecting comparatively few features.

## Related tools

- **suffpcr** (Fit suffPCR (row-sparse PCA followed by OLS or logistic regression) on each training fold; provides the aml, dlbcl, miller, nsclc, vantveer microarray datasets being benchmarked) — https://github.com/dajmcdon/suffpcr
- **R** (Runtime environment for the benchmark script and aggregation of test MSE / feature-count summaries) — https://github.com/dajmcdon/suffpcr
- **remotes** (Install the suffpcr package from GitHub into the R environment) — https://github.com/dajmcdon/suffpcr

## Examples

```
remotes::install_github("dajmcdon/suffpcr"); library(suffpcr); out <- suffPCR(X, y, d=1:3)
```

## Evaluation signals

- Reproduced summary shows SuffPCR with the lowest average test MSE on 4 of the 5 microarray datasets, matching the paper's reported ranking pattern
- Average selected feature counts for SuffPCR are comparatively small relative to competing sparse methods
- Every method-dataset cell is based on exactly 10 repetitions of independently drawn 40/30/30 splits
- Lambda for every method on every split was chosen on the validation fold only, never on the test fold
- Output CSV contains both metrics (average test MSE and average number of selected features) per method per dataset

## Limitations

- Results depend on the specific random splits; the paper uses 10 repetitions of 40/30/30 splits, so a single split may not reproduce the reported ranking
- The suffpcr repository has no changelog, so package-version drift may affect reproducibility of the benchmark
- Near-optimal performance of SuffPCR relies on model assumptions (e.g., sparse principal components) being satisfied; violations may degrade prediction or feature selection

## Evidence

- [results] SuffPCR attains lowest average test MSE on 4 of 5 datasets: [evidence span withheld — anchor status: not_found]
- [readme] Package installation from GitHub: "remotes::install_github("dajmcdon/suffpcr")"
- [intro] Method definition: sparse PCA then linear model: "The goal of suffpcr is to estimate sufficient principal component regression by implementing row-sparse PCA followed by ordinary least squares or logistic regression."
- [methods] Datasets shipped with the package: "- `aml (suffpcr)` - `dlbcl (suffpcr)` - `miller (suffpcr)` - `nsclc (suffpcr)` - `vantveer (suffpcr)`"
- [methods] Core API call: "out <- suffPCR(X, y, d=1:3)"
