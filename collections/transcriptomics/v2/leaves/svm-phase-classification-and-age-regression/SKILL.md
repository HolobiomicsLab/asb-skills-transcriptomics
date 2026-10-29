---
name: svm-phase-classification-and-age-regression
description: Use when benchmarking feature selection methods on proteomic-imaging
  single-cell data with ground truth labels, to test whether a top-p feature subset
  (e.g. p=30) preserves biologically meaningful signals like cell cycle phase and
  continuous cell age.
when_to_use_negative:
- Ground truth phase/age annotations are unavailable — the classifier/regressor evaluation
  requires labeled data; use unsupervised metrics (NMI, pseudotime correlation) instead
- Evaluating a single fixed feature set rather than comparing across feature selection
  methods — the nested-CV SVM benchmark is designed for method comparison with matched
  top-p feature counts
- Data is scRNA-seq rather than proteomic-imaging features with continuous age annotations
  — the age-regression RMSE metric assumes a quantitative time axis
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0091
tools:
- name: scikit-learn
  role: Trains the SVM classifier and SVM regressor with nested 10-fold cross-validation
    and grid-searched hyperparameters to score selected feature sets
- name: delve_benchmark
  role: Provides the `fs` class and the twelve feature selection functions (run_delve_fs,
    laplacian_score_fs, hotspot_fs, etc.) whose top-30 selected features feed the
    SVM evaluation
  repo: https://github.com/jranek/delve_benchmark
- name: DELVE
  role: Feature selection method whose output features are benchmarked; expected to
    achieve highest median SVM phase classification accuracy (0.960)
  repo: https://github.com/jranek/delve
- name: anndata
  role: Reads and validates the adata_RPE.h5ad annotated data object (cells x features)
    including obs annotations
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:154365355544255298526947788c252f1638d11299c7e99a883c9cb3a3b5bc26
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jranek/delve_benchmark
  tools:
  - DELVE
  - Hotspot
  - Python
  - Slingshot
  - anndata
  - delve_benchmark
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# svm-phase-classification-and-age-regression

## Summary

Evaluates feature selection methods on the RPE 4i cell-cycle dataset by training an SVM classifier (nested 10-fold CV, grid-searched hyperparameters) to predict cell cycle phase and an SVM regressor to predict cell cycle age, recording median classification accuracy and RMSE in hours per method.

## When to use

Use when benchmarking feature selection methods on proteomic-imaging single-cell data with ground truth labels, to test whether a top-p feature subset (e.g. p=30) preserves biologically meaningful signals like cell cycle phase and continuous cell age.

## When NOT to use

- Ground truth phase/age annotations are unavailable — the classifier/regressor evaluation requires labeled data; use unsupervised metrics (NMI, pseudotime correlation) instead
- Evaluating a single fixed feature set rather than comparing across feature selection methods — the nested-CV SVM benchmark is designed for method comparison with matched top-p feature counts
- Data is scRNA-seq rather than proteomic-imaging features with continuous age annotations — the age-regression RMSE metric assumes a quantitative time axis

## Inputs

- Preprocessed RPE 4i annotated data object (adata_RPE.h5ad, 2759 cells x 241 features) from Zenodo record 10534873 with cell cycle phase and age annotations in adata.obs
- Ranked feature lists (top p=30 features) from each of the twelve feature selection methods
- Ground truth cell cycle phase labels and cell cycle age values per cell

## Outputs

- Median SVM cell cycle phase classification accuracy per feature selection method
- SVM regressor RMSE (hours) for cell cycle age prediction per feature selection method
- Summary CSV aggregating median metric values per method

## How to apply

Load the preprocessed RPE 4i dataset (adata_RPE.h5ad, 2759 cells x 241 features) from Zenodo record 10534873 with anndata, verifying ground truth cell cycle phase and age annotations in adata.obs. For each of the twelve feature selection strategies (random forest, DELVE with k=10, num_subsamples=1000, n_clusters=5 across 20 random trials, dynamic seed, Laplacian Score, neighborhood variance, Hotspot, MCFS, SCMER, max variance, HVG, all features, random), select the top p=30 features using the delve_benchmark `fs` class (`select()` / `evaluate_select()`). Train an SVM classifier with nested 10-fold cross-validation and grid-searched hyperparameters on the selected features to predict cell cycle phase, recording median accuracy; train an SVM regressor with nested 10-fold CV to predict cell cycle age and compute RMSE in hours. Expected DELVE results: median classification accuracy 0.960 (vs 0.957 for the supervised random forest on ground truth annotations) and age-regression RMSE 1.806 h.

## Related tools

- **scikit-learn** (Trains the SVM classifier and SVM regressor with nested 10-fold cross-validation and grid-searched hyperparameters to score selected feature sets)
- **delve_benchmark** (Provides the `fs` class and the twelve feature selection functions (run_delve_fs, laplacian_score_fs, hotspot_fs, etc.) whose top-30 selected features feed the SVM evaluation) — https://github.com/jranek/delve_benchmark
- **DELVE** (Feature selection method whose output features are benchmarked; expected to achieve highest median SVM phase classification accuracy (0.960)) — https://github.com/jranek/delve
- **anndata** (Reads and validates the adata_RPE.h5ad annotated data object (cells x features) including obs annotations)

## Examples

```
from delve_benchmark.fs import fs; model = fs(adata=adata, fs_method=run_delve_fs, fs_method_params={'k':10,'num_subsamples':1000,'n_clusters':5}, eval_method=svm_classification, feature_threshold=30, labels_key='phase'); features, score_df = model.evaluate_select()
```

## Evaluation signals

- Input adata dimensions match 2759 cells x 241 features with phase and age keys present in adata.obs
- DELVE's median SVM phase classification accuracy reaches 0.960, within ~0.003 of the supervised random-forest reference (0.957) and above all other unsupervised methods and the all-features baseline
- DELVE's age-regression RMSE is 1.806 hours, lower than competing unsupervised feature selection methods
- Each method contributes exactly the top p=30 features, enabling a matched-threshold comparison across methods
- Summary CSV contains one median accuracy and one RMSE per method across the twelve strategies

## Limitations

- Supervised evaluation with nested 10-fold CV is computationally expensive when repeated across 12 methods x 20 DELVE random trials
- SVM performance reflects predictive signal for phase/age, not preservation of trajectory topology — the article notes feature selection can bias interpretation of the underlying cellular trajectory, so complementary metrics (p@30, NMI, pseudotime correlation) should be reported
- Results depend on the p=30 threshold; the skill is tuned to the benchmark's matched top-p comparison rather than arbitrary feature set sizes

## Evidence

- [discussion] Benchmarked twelve feature selection methods on identifying biologically relevant features for trajectory analysis: "we benchmarked twelve feature selection methods29,49,53,55–58 on their ability to identify biologically relevant features for trajectory analysis"
- [readme] fs class select/evaluate API: [evidence span withheld — anchor status: external_doc]
- [readme] feature_threshold parameter: [evidence span withheld — anchor status: external_doc]
- [results] DELVE achieves highest phase classification accuracy: [evidence span withheld — anchor status: not_found]
