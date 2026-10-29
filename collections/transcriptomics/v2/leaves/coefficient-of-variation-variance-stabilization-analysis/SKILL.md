---
name: coefficient-of-variation-variance-stabilization-analysis
description: Use when evaluating whether an imputation method reduces gene-expression
  variability across cells of the same type — i.e., you have unimputed and imputed
  count/log matrices plus cell-type annotations and need a variance-stabilization
  comparison.
when_to_use_negative:
- The analysis goal is expression recovery from subsampled (masked-matrix) data or
  clustering accuracy — those require different evaluations (e.g., AutoImpute's --masked_matrix_test
  masked-matrix recovery test), not intra-subpopulation CV.
- You want to assess cell-type separability across subpopulations rather than variance
  within a single randomly picked subpopulation; CV-based stabilization is a within-type
  metric.
- Only one imputation method or no unimputed baseline is available, making a per-method
  CV comparison impossible.
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: AutoImpute
  role: Overcomplete-autoencoder imputation method whose output matrix is evaluated
    for intra-subpopulation variance stabilization
  repo: github.com/divyanshu-talwar/AutoImpute
- name: numpy
  role: Computes per-gene standard deviation/mean (coefficient of variation) across
    cells of the selected subpopulation
- name: matplotlib
  role: Draws grouped boxplots of log-CV distributions per method, replicating the
    four Fig. 4 panels
- name: scikit-learn
  role: Python dependency of the AutoImpute model used to produce the imputed matrices
    under comparison
  repo: github.com/divyanshu-talwar/AutoImpute
- name: scImpute
  role: Comparison imputation method evaluated for variance stabilization
- name: MAGIC
  role: Comparison imputation method evaluated for variance stabilization
- name: drImpute
  role: Comparison imputation method evaluated for variance stabilization
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:2875bb9803a3d52a5603c405a88968c2c7d7503925841b11989192c55d6b364f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoImpute
  - MAGIC
  - drImpute
  - matplotlib
  - numpy
  - scImpute
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# coefficient-of-variation-variance-stabilization-analysis

## Summary

Compares how well scRNA-seq imputation methods (AutoImpute, scImpute, MAGIC, drImpute) stabilize intra-subpopulation gene-expression variance by computing per-gene coefficients of variation (CV = SD/mean) within a randomly chosen cell-type subpopulation, log-transforming the CVs, and visualizing them as grouped boxplots (Fig. 4 style).

## When to use

Use when evaluating whether an imputation method reduces gene-expression variability across cells of the same type — i.e., you have unimputed and imputed count/log matrices plus cell-type annotations and need a variance-stabilization comparison.

## When NOT to use

- The analysis goal is expression recovery from subsampled (masked-matrix) data or clustering accuracy — those require different evaluations (e.g., AutoImpute's --masked_matrix_test masked-matrix recovery test), not intra-subpopulation CV.
- You want to assess cell-type separability across subpopulations rather than variance within a single randomly picked subpopulation; CV-based stabilization is a within-type metric.
- Only one imputation method or no unimputed baseline is available, making a per-method CV comparison impossible.

## Inputs

- Pre-processed log-transformed scRNA-seq expression matrices (cells x genes) for Usoskin, Kolodziejczyk, Jurkat-293T and PBMC datasets (e.g., <dataset_name>.csv or .mat)
- Imputed expression matrices for AutoImpute, scImpute, MAGIC and drImpute on the same datasets
- Cell-type/subpopulation annotations for each dataset

## Outputs

- cv_summary.csv containing per-gene CV values for the unimputed data and every imputation method
- cv_boxplots.png with one panel per dataset (Usoskin, Kolodziejczyk, Jurkat-293T, PBMC) showing grouped log-CV boxplots per method

## How to apply

Load the pre-processed log-transformed expression matrices (Usoskin, Kolodziejczyk, Jurkat-293T, PBMC) alongside the corresponding matrices imputed by AutoImpute, scImpute, MAGIC and drImpute, plus cell-type annotations. Randomly pick one subpopulation per dataset and subset both unimputed and imputed matrices to those cells. For methods that consumed log-transformed input (e.g., AutoImpute, whose README states the input is a 'pre-processed count single cell matrix, with columns representing genes and rows representing cells'), first apply the anti-log operation to return to the linear expression scale before computing variability. With numpy, compute the coefficient of variation (standard deviation divided by mean) per gene across the selected cells for every method, then take the natural logarithm of the per-gene CVs to amplify fractional differences as done in the paper. Draw grouped boxplots of log-CV distributions per method for each dataset with matplotlib and export per-gene CVs to CSV.

## Related tools

- **AutoImpute** (Overcomplete-autoencoder imputation method whose output matrix is evaluated for intra-subpopulation variance stabilization) — github.com/divyanshu-talwar/AutoImpute
- **numpy** (Computes per-gene standard deviation/mean (coefficient of variation) across cells of the selected subpopulation)
- **matplotlib** (Draws grouped boxplots of log-CV distributions per method, replicating the four Fig. 4 panels)
- **scikit-learn** (Python dependency of the AutoImpute model used to produce the imputed matrices under comparison) — github.com/divyanshu-talwar/AutoImpute
- **scImpute** (Comparison imputation method evaluated for variance stabilization)
- **MAGIC** (Comparison imputation method evaluated for variance stabilization)
- **drImpute** (Comparison imputation method evaluated for variance stabilization)

## Examples

```
python AutoImpute.py --data jurkat.csv --hidden_units 2000 --iterations 7000 --imputed_save imputed_matrix
```

## Evaluation signals

- The output figure contains four dataset panels, each comparing log-CV distributions of all methods (unimputed baseline, AutoImpute, scImpute, MAGIC, drImpute).
- AutoImpute achieves the lowest median log-CV in the datasets where the paper claims it stabilizes intra-subpopulation variance the most (Jurkat, Preimplantation, PBMC).
- cv_boxplots.png exists as a valid PNG and cv_summary.csv contains per-gene CV rows for every method.
- Imputed matrices produced from log-transformed input (e.g., AutoImpute) were anti-logged back to the linear scale before CV computation.
- Per-gene CV values were log-transformed (natural log) before plotting, matching the paper's amplification of fractional differences.

## Limitations

- The paper evaluates only nine independent datasets, and AutoImpute was best in three of them while being outperformed by MAGIC in at least two, so results are dataset-dependent rather than universal.
- AutoImpute (per the README) runs on Python 2.7 with TensorFlow and expects a pre-processed count matrix (cells in rows, genes in columns) produced by the R pre-process.R script; raw or differently oriented matrices must be pre-processed first.
- Only one subpopulation is randomly picked per dataset, so conclusions depend on the subpopulation chosen.

## Evidence

- [results] AutoImpute stabilized the intra-subpopulation gene expression variance the most in three datasets: [evidence span withheld — anchor status: not_found]
- [readme] pre-processed count single cell matrix input format: "The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells."
- [abstract] autoencoder-based imputation of sparse gene expression matrix: "we developed an autoencoder-based sparse gene expression matrix imputation method. AutoImpute, which learns the inherent distribution of the input scRNA-seq data"
- [intro] evaluate variance stabilization among comparison criteria: "We compare the performance of AutoImpute with the existing imputation methods on nine independent datasets."
