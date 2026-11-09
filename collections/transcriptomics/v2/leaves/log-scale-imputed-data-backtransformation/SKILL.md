---
name: log-scale-imputed-data-backtransformation
description: Use when an imputation method (e.g. AutoImpute) was run on log-transformed
  count matrices but the downstream metric — like intra-subpopulation coefficient
  of variation for Fig. 4 variance stabilization — must be computed on linear-scale
  expression values.
when_to_use_negative:
- The imputation method already consumed and returned linear-scale counts (no anti-log
  step is needed and applying one would distort the data)
- The downstream analysis operates directly on log-scale values (e.g. log-scale visualization
  or clustering) where back-transformation is unnecessary
edam_operation: http://edamontology.org/operation_3230
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: AutoImpute
  role: Imputation method whose output is on the log-transformed scale and therefore
    requires anti-log back-transformation before variance (CV) computation
  repo: https://github.com/divyanshu-talwar/AutoImpute
- name: numpy
  role: Element-wise exponential (anti-log) transform and per-gene standard deviation/mean
    computation for the coefficient of variation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:7b91941b0784b9f52e3642330be1344bf7d9c7c61c846cbdf1ff2cb7cb0387c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/divyanshu-talwar/AutoImpute
  tools:
  - AutoImpute
  - matplotlib
  - numpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3230
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Log-scale imputed data back-transformation

## Summary

Anti-log (exponential) transform imputed scRNA-seq matrices that were produced by methods consuming log-transformed input (e.g. AutoImpute), so downstream variability statistics such as the per-gene coefficient of variation are computed on the linear expression scale.

## When to use

Use when an imputation method (e.g. AutoImpute) was run on log-transformed count matrices but the downstream metric — like intra-subpopulation coefficient of variation for Fig. 4 variance stabilization — must be computed on linear-scale expression values.

## When NOT to use

- The imputation method already consumed and returned linear-scale counts (no anti-log step is needed and applying one would distort the data)
- The downstream analysis operates directly on log-scale values (e.g. log-scale visualization or clustering) where back-transformation is unnecessary

## Inputs

- Log-transformed imputed gene-expression matrix from AutoImpute (cells x genes, .csv/.mat)
- Log-transformed pre-processed count matrix for the same dataset (cells x genes)
- Cell-type/subpopulation annotations used to subset cells before computing CV

## Outputs

- Back-transformed (linear-scale) imputed expression matrix with same dimensions as input
- Per-gene coefficient of variation values on the linear scale, exportable to cv_summary.csv

## How to apply

Identify which imputed matrices came from methods that consumed log-transformed input; for AutoImpute the pipeline takes the pre-processed single-cell matrix and returns an imputed matrix of identical dimensions (rows = cells, columns = genes). For those matrices only, apply the anti-log (exponential) operation element-wise to return to the linear expression scale before computing any variability measure. Keep the unimputed log-matrix and methods that operated on linear counts unchanged. Then compute per-gene statistics (standard deviation / mean) across the selected cells and, as done in the paper, take the natural logarithm of the per-gene CV values to amplify fractional differences. Skipping the back-transformation deflates CV for log-scaled methods and biases comparisons like the Jurkat-293T, Usoskin, Kolodziejczyk and PBMC variance-stabilization boxplots.

## Related tools

- **AutoImpute** (Imputation method whose output is on the log-transformed scale and therefore requires anti-log back-transformation before variance (CV) computation) — https://github.com/divyanshu-talwar/AutoImpute
- **numpy** (Element-wise exponential (anti-log) transform and per-gene standard deviation/mean computation for the coefficient of variation)

## Examples

```
import numpy as np; X_linear = np.expm1(X_log_imputed); cv = X_linear.std(axis=0) / X_linear.mean(axis=0); log_cv = np.log(cv)
```

## Evaluation signals

- Back-transformed matrix has identical dimensions (cells x genes) to the log-transformed input matrix
- All values in the back-transformed matrix are positive (linear expression scale), whereas log-scale inputs may contain non-positive values
- Per-gene CV computed on back-transformed data is finite (mean > 0) and larger than CV computed directly on the log-scale matrix
- In the Fig. 4 replication, the reported pattern shows AutoImpute achieving the lowest median log-CV in the Jurkat, Preimplantation and PBMC datasets
- cv_summary.csv contains per-gene CV rows for every method and cv_boxplots.png is a valid four-panel PNG

## Limitations

- Anti-log back-transformation is only correct for methods that consumed log-transformed input; applying it to methods operating on linear counts invalidates the CV comparison
- The AutoImpute README documents input as a pre-processed count single-cell matrix (columns = genes, rows = cells); any scale mismatch between raw, pre-processed, and imputed matrices must be resolved before back-transformation

## Evidence

- [readme] The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells. It's output is an imputed count matrix with the same dimension.: "The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells. It's output is an imputed count matrix with the same dimension."
- [intro] AutoImpute learns the inherent distribution of the input scRNA-seq data and imputes the missing values accordingly with minimal modification to the biologically silent gene expression values.: "AutoImpute learns the inherent distribution of the input scRNA-seq data and imputes the missing values accordingly with minimal modification to the biologically silent gene expression values."
- [other] For imputed matrices produced by methods that consumed log-transformed input (e.g. AutoImpute), first apply the anti-log operation to return to linear expression scale before computing variability.: [evidence span withheld — anchor status: not_found]
