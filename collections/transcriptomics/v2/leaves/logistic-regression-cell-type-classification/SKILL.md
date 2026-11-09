---
name: logistic-regression-cell-type-classification
description: Use when a selected marker gene set (e.g., 42 genes from GreedyPC on
  IPF) must be validated by training a logistic regression classifier on the same
  data subset used for feature selection and evaluating macro-F1 on left-out cells.
when_to_use_negative:
- The goal is pseudo-bulk mixture deconvolution via ν-SVR (ν ∈ {0.25, 0.5, 0.75},
  lowest RMSE) rather than classifying individual cells into phenotypes
- Input is raw unnormalized count data that has not undergone gene/cell filtering,
  normalization, and log transformation
- Only a ranked feature list is needed with no downstream classifier-based validation
  of discriminative power
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: phenotype-cover
  role: Provides GreedyPC and CEMPC to select the marker feature set that the logistic
    regression is subsequently trained on
  repo: https://github.com/euxhenh/phenotype-cover
- name: scikit-learn
  role: Trains the LogisticRegression classifier and computes macro-F1 on left-out
    cells; also supplies decision tree and mutual-information baseline selectors
- name: mRMR (mrmr_selection)
  role: Baseline minimum-redundancy maximum-relevance feature selector (mrmr_classif)
    whose selected features are evaluated with the same logistic regression protocol
  repo: https://github.com/smazzanti/mrmr
- name: numpy
  role: Holds and preprocesses the scRNA-seq count matrices (filtering, normalization,
    log transform) fed to selection and classification
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:492240d0d8e1c9d74e4559ac374132b09f164f17219e290ee46ee5f0bd2e64ed
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/euxhenh/phenotype-cover
  repo_url: https://github.com/euxhenh/phenotype-cover
  tools:
  - GreedyPC
  - Python
  - mRMR (mrmr_selection)
  - numpy
  - phenotype-cover
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# logistic-regression-cell-type-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark discriminative marker sets (e.g., G-PC, CEM-PC, scGeneFit, mRMR, TopDE, ReliefF, decision trees, mutual information) by training a scikit-learn logistic regression on the selected genes and scoring macro-F1 on left-out cells of scRNA-seq phenotype datasets (IPF lung, mouse cortex, human cell atlas).

## When to use

Use when a selected marker gene set (e.g., 42 genes from GreedyPC on IPF) must be validated by training a logistic regression classifier on the same data subset used for feature selection and evaluating macro-F1 on left-out cells.

## When NOT to use

- The goal is pseudo-bulk mixture deconvolution via ν-SVR (ν ∈ {0.25, 0.5, 0.75}, lowest RMSE) rather than classifying individual cells into phenotypes
- Input is raw unnormalized count data that has not undergone gene/cell filtering, normalization, and log transformation
- Only a ranked feature list is needed with no downstream classifier-based validation of discriminative power

## Inputs

- Preprocessed log-transformed scRNA-seq expression matrix (numpy array) with cell phenotype labels for IPF (Adams et al.), mouse cortex (Zeisel et al.), or HCA (He et al.)
- Selected marker gene indices/names per method (e.g., 42 genes from GreedyPC on IPF)
- Identical train/test cell splits shared across all feature-selection methods

## Outputs

- Trained scikit-learn LogisticRegression model per method-dataset pair
- Macro-F1 score for every method-dataset pair, written to macro_f1_results.csv
- Deviation flags when reproduced scores differ from reported values (G-PC 0.70 IPF / 0.94–0.95 MC / 0.68 HCA)

## How to apply

Load and preprocess the scRNA-seq count matrices (IPF lung Adams et al., mouse cortex Zeisel et al., HCA He et al.) with gene/cell filtering, normalization, and log transformation — the phenotype-cover README notes you may need to log-transform X when working with raw counts. Run each selector (GreedyPC/CEMPC via phenotype-cover at the reported operating point, e.g., 42 genes on IPF; mrmr_classif from mrmr_selection; scGeneFit, TopDE, ReliefF, decision trees, mutual-information ranking from scikit-learn) on identical data splits. For each method, select feature set S using a subset of the data, train a scikit-learn LogisticRegression on that same subset, and evaluate macro-F1 on left-out cells. Compare reproduced scores against reported values (G-PC 0.70 on IPF with 42 genes, 0.94–0.95 on MC, 0.68 on HCA; scGeneFit 0.65, CEM-PC 0.61 on IPF) and write every method-dataset macro-F1 to macro_f1_results.csv, flagging deviations.

## Related tools

- **phenotype-cover** (Provides GreedyPC and CEMPC to select the marker feature set that the logistic regression is subsequently trained on) — https://github.com/euxhenh/phenotype-cover
- **scikit-learn** (Trains the LogisticRegression classifier and computes macro-F1 on left-out cells; also supplies decision tree and mutual-information baseline selectors)
- **mRMR (mrmr_selection)** (Baseline minimum-redundancy maximum-relevance feature selector (mrmr_classif) whose selected features are evaluated with the same logistic regression protocol) — https://github.com/smazzanti/mrmr
- **numpy** (Holds and preprocesses the scRNA-seq count matrices (filtering, normalization, log transform) fed to selection and classification)

## Examples

```
from phenotype_cover import GreedyPC; gpc = GreedyPC(); gpc.fit(X, y); features = gpc.select(100); from sklearn.linear_model import LogisticRegression; LogisticRegression().fit(X[:, features], y_train)
```

## Evaluation signals

- Reproduced G-PC macro-F1 is within tolerance of reported values: 0.70 on IPF with 42 genes, 0.94–0.95 on MC, 0.68 on HCA
- Baseline ordering reproduced on IPF: G-PC (0.70) > scGeneFit (0.65) > CEM-PC (0.61); decision trees reach 0.70 on IPF
- The logistic regression is trained on exactly the same data subset used for feature selection, and F1 is computed only on left-out cells
- All selectors run on identical data splits so macro-F1 differences are attributable to the marker sets
- One macro-F1 row per method-dataset pair exists in macro_f1_results.csv with deviations flagged

## Limitations

- G-PC does not provide an optimal solution to the phenotype cover problem, so marker sets fed to the classifier are approximations
- G-PC worked well for the data analyzed in the paper but its generalization to other datasets is not guaranteed
- Raw scRNA-seq counts must be log-transformed before use, and results depend on the chosen operating point (e.g., 42 genes on IPF)

## Evidence

- [results] Reported baseline comparison: [evidence span withheld — anchor status: not_found]
