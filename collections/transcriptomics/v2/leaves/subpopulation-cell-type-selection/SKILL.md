---
name: subpopulation-cell-type-selection
description: 'Use when comparing imputation methods on scRNA-seq data by intra-subpopulation
  variance stabilization: cell-type annotations and raw/imputed log-transformed count
  matrices (cells x genes) are available for datasets like Jurkat-293T, Usoskin, Kolodziejczyk
  or PBMC.'
when_to_use_negative:
- When the evaluation target is expression recovery from subsampled data or cell-clustering
  accuracy rather than intra-subpopulation variance stabilization
- When no cell-type annotations exist and a specific subpopulation cannot be isolated
  from the cells
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: AutoImpute
  role: Autoencoder-based imputation method whose imputed matrix is anti-logged and
    compared against other methods on per-gene CV within a selected subpopulation
  repo: github.com/divyanshu-talwar/AutoImpute
- name: numpy
  role: Computes per-gene coefficient of variation (standard deviation / mean) across
    the cells of the chosen subpopulation
- name: matplotlib
  role: Draws the grouped log-CV boxplots replicating the four dataset panels of Fig.
    4
- name: scikit-learn
  role: Supporting dependency for the AutoImpute model pipeline (per README)
  repo: github.com/divyanshu-talwar/AutoImpute
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:acb274fbbe46e5f874a93a39345111e428283fc9aa0335c3b24a9ac5641d6db2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoImpute
  - matplotlib
  - numpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# subpopulation-cell-type-selection

## Summary

Selects one randomly chosen cell-type subpopulation from a scRNA-seq dataset and computes per-gene coefficient of variation (CV) across its cells on both unimputed and imputed expression matrices, to quantify which imputation method (AutoImpute, scImpute, MAGIC, drImpute) best stabilizes intra-subpopulation variance as in Fig. 4 of the AutoImpute paper.

## When to use

Use when comparing imputation methods on scRNA-seq data by intra-subpopulation variance stabilization: cell-type annotations and raw/imputed log-transformed count matrices (cells x genes) are available for datasets like Jurkat-293T, Usoskin, Kolodziejczyk or PBMC.

## When NOT to use

- When the evaluation target is expression recovery from subsampled data or cell-clustering accuracy rather than intra-subpopulation variance stabilization
- When no cell-type annotations exist and a specific subpopulation cannot be isolated from the cells

## Inputs

- Pre-processed log-transformed scRNA-seq expression matrix (cells x genes) for each dataset (e.g. Usoskin, Kolodziejczyk, Jurkat-293T, PBMC)
- Imputed expression matrices from AutoImpute, scImpute, MAGIC, and drImpute for the same datasets
- Cell-type annotations assigning each cell to a subpopulation

## Outputs

- Per-gene CV values (and natural-log CV) for each method within the selected subpopulation (cv_summary.csv)
- Grouped boxplots of log-CV distributions per method per dataset replicating Fig. 4 (cv_boxplots.png)

## How to apply

Load the pre-processed log-transformed expression matrices for each dataset (e.g. Usoskin, Kolodziejczyk, Jurkat-293T, PBMC) together with cell-type annotations and the matrices imputed by AutoImpute, scImpute, MAGIC and drImpute. Randomly pick one subpopulation (cell type) per dataset and subset both unimputed and imputed matrices to those cells using the annotations. Because AutoImpute consumed log-transformed input, anti-log its imputed matrix back to linear scale before computing variability, matching the paper's preprocessing. With numpy, compute CV (standard deviation / mean) per gene across the selected cells for the raw and each imputed matrix, then take the natural logarithm of the per-gene CV values to amplify fractional differences as done in the paper. Draw grouped boxplots of log-CV distributions per method for each dataset with matplotlib (the four panels of Fig. 4), and write per-gene CVs to cv_summary.csv.

## Related tools

- **AutoImpute** (Autoencoder-based imputation method whose imputed matrix is anti-logged and compared against other methods on per-gene CV within a selected subpopulation) — github.com/divyanshu-talwar/AutoImpute
- **numpy** (Computes per-gene coefficient of variation (standard deviation / mean) across the cells of the chosen subpopulation)
- **matplotlib** (Draws the grouped log-CV boxplots replicating the four dataset panels of Fig. 4)
- **scikit-learn** (Supporting dependency for the AutoImpute model pipeline (per README)) — github.com/divyanshu-talwar/AutoImpute

## Examples

```
python AutoImpute.py --data blakeley.csv --hidden_units 2000 --iterations 7000 --imputed_save imputed_matrix
```

## Evaluation signals

- cv_boxplots.png exists as a valid PNG and contains four dataset panels (Usoskin, Kolodziejczyk, Jurkat-293T, PBMC), each comparing all imputation methods
- cv_summary.csv contains per-gene CV rows for every method (unimputed, AutoImpute, scImpute, MAGIC, drImpute)
- AutoImpute achieves the lowest median log-CV in the datasets where the paper claims it stabilizes intra-subpopulation variance the most (Jurkat, Preimplantation and PBMC)
- AutoImpute imputed values were anti-logged to linear scale before CV computation (only methods fed log-transformed input need this)
- Per-gene CV values equal standard deviation / mean computed over cells of a single annotated subpopulation only, not the full dataset

## Limitations

- Comparison depends on which single subpopulation is randomly selected per dataset; results may vary with a different pick
- AutoImpute input must be a pre-processed count matrix with columns as genes and rows as cells; log/linear scale mismatches across methods bias CV comparisons
- The paper reports AutoImpute was better than all methods except MAGIC in two datasets, so lowest-median-CV is not expected universally

## Evidence

- [readme] README: input format: "The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells."
- [abstract] Abstract: variance stabilization benchmark: "AutoImpute performed competitively wrt., the existing single-cell imputation methods, on the grounds of expression recovery from subsampled data, cell-clustering accuracy, variance stabilization"
- [intro] Intro: benchmark scope: "We compare the performance of AutoImpute with the existing imputation methods on nine independent datasets."
- [abstract] Abstract: imputation philosophy: "AutoImpute learns the inherent distribution of the input scRNA-seq data"
