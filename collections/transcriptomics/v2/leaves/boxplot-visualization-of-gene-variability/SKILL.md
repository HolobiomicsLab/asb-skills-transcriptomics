---
name: boxplot-visualization-of-gene-variability
description: Use when comparing imputation methods on single-cell RNA-seq data by
  measuring which one reduces gene-wise variability across cells of the same type,
  replicating the variance-stabilization analysis (Fig. 4) of the AutoImpute paper.
when_to_use_negative:
- The goal is to compare methods on expression recovery from subsampled data or cell-clustering
  accuracy rather than intra-subpopulation variance stabilization
- The data are bulk RNA-seq or already aggregated, so there is no intra-subpopulation
  (cell-to-cell) variance to stabilize
- Cell-type annotations are unavailable, making it impossible to isolate a single
  subpopulation for the CV calculation
edam_operation: http://edamontology.org/operation_0572
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_3170
tools:
- name: AutoImpute
  role: Imputation method whose imputed matrix is evaluated for intra-subpopulation
    variance stabilization alongside competing methods
  repo: github.com/divyanshu-talwar/AutoImpute
- name: numpy
  role: Computes per-gene coefficient of variation (standard deviation / mean) across
    the cells of the selected subpopulation
- name: matplotlib
  role: Draws grouped boxplots of log-CV distributions per method for each dataset
    panel
- name: scikit-learn
  role: Supports the AutoImpute autoencoder model pipeline that produced the imputed
    matrices being compared
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:7e3b52f22f4a72bb13d4d0723586067b59de4e281e747b9bf28b6ef178cbcc9a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoImpute
  - matplotlib
  - numpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0572
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# boxplot-visualization-of-gene-variability

## Summary

Quantifies per-gene coefficient of variation (CV) within a single randomly picked cell-type subpopulation and visualizes log-transformed CV distributions as grouped boxplots to compare how well imputation methods (AutoImpute, scImpute, MAGIC, drImpute) stabilize intra-subpopulation variance.

## When to use

Use when comparing imputation methods on single-cell RNA-seq data by measuring which one reduces gene-wise variability across cells of the same type, replicating the variance-stabilization analysis (Fig. 4) of the AutoImpute paper.

## When NOT to use

- The goal is to compare methods on expression recovery from subsampled data or cell-clustering accuracy rather than intra-subpopulation variance stabilization
- The data are bulk RNA-seq or already aggregated, so there is no intra-subpopulation (cell-to-cell) variance to stabilize
- Cell-type annotations are unavailable, making it impossible to isolate a single subpopulation for the CV calculation

## Inputs

- Pre-processed log-transformed gene-by-cell expression matrices (.csv or .mat) for Jurkat-293T, Usoskin, Kolodziejczyk and PBMC datasets
- Imputed expression matrices from AutoImpute, scImpute, MAGIC and drImpute for each dataset
- Cell-type annotations per dataset defining subpopulations

## Outputs

- cv_summary.csv with per-gene CV values for every method and dataset
- cv_boxplots.png with four dataset panels of grouped log-CV boxplots comparing all methods

## How to apply

Load the pre-processed log-transformed expression matrices for the Jurkat-293T, Usoskin, Kolodziejczyk and PBMC datasets together with their AutoImpute, scImpute, MAGIC and drImpute imputed counterparts and cell-type annotations. Randomly select one subpopulation per dataset and subset both unimputed and imputed matrices to those cells; for imputed matrices produced from log-transformed input (e.g., AutoImpute), first apply the anti-log operation to return to linear scale. With numpy, compute CV = standard deviation / mean for each gene across the selected cells, then take the natural logarithm of the per-gene CV values to amplify fractional differences as in the paper. Draw grouped matplotlib boxplots of the log-CV distributions per method, one panel per dataset, replicating the four panels of Fig. 4.

## Related tools

- **AutoImpute** (Imputation method whose imputed matrix is evaluated for intra-subpopulation variance stabilization alongside competing methods) — github.com/divyanshu-talwar/AutoImpute
- **numpy** (Computes per-gene coefficient of variation (standard deviation / mean) across the cells of the selected subpopulation)
- **matplotlib** (Draws grouped boxplots of log-CV distributions per method for each dataset panel)
- **scikit-learn** (Supports the AutoImpute autoencoder model pipeline that produced the imputed matrices being compared)

## Examples

```
python AutoImpute.py --data blakeley.csv --hidden_units 2000 --iterations 7000 --imputed_save imputed_matrix  # then compute per-gene CV = np.std(X, axis=0) / np.mean(X, axis=0) on the selected subpopulation
```

## Evaluation signals

- cv_boxplots.png exists and is a valid PNG containing four dataset panels (Usoskin, Kolodziejczyk, Jurkat-293T, PBMC), each comparing all methods
- cv_summary.csv contains per-gene CV rows for every method (unimputed, AutoImpute, scImpute, MAGIC, drImpute)
- AutoImpute shows the lowest median log-CV in the three datasets where the paper claims it stabilizes variance the most (Jurkat, Preimplantation and PBMC)
- Imputed matrices from log-input methods were anti-logged before CV computation, so CV values are on linear expression scale
- Per-gene log-CV values are finite and non-degenerate (means are not zero after subsetting to the chosen subpopulation)

## Limitations

- Only one subpopulation is randomly picked per dataset, so the result depends on which cell type is sampled and may not generalize to all subpopulations
- The paper reports AutoImpute stabilized variance the most in three datasets but performed better than all methods except MAGIC in two datasets, so superiority is not universal
- Imputed matrices from methods consuming log-transformed input must be anti-logged before CV computation or the comparison is invalid
- README notes preprocessing depends on a chain of R packages (Matrix, irlba, dynamicTreeCut, etc.) and the model runs on Python 2.7 with TensorFlow

## Evidence

- [abstract] AutoImpute performed competitively wrt., the existing single-cell imputation methods, on the grounds of expression recovery from subsampled data, cell-clustering accuracy, variance stabilization and: "AutoImpute performed competitively wrt., the existing single-cell imputation methods, on the grounds of expression recovery from subsampled data, cell-clustering accuracy, variance stabilization"
- [intro] Compare on nine independent datasets: "We compare the performance of AutoImpute with the existing imputation methods on nine independent datasets."
- [readme] README input/output description: "The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells. It's output is an imputed count matrix with the same dimension."
- [readme] README dependencies: "* For Python (2.7):
    > numpy, scikit-learn, tensorflow, matplotlib."
