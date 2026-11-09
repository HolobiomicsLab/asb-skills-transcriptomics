---
name: cross-validated-regularization-tuning
description: Use when training a penalized multinomial classifier (e.g. glmnet) on
  per-cell signature-score features and the regularization strength must be tuned
  by k-fold cross-validation rather than fixed a priori.
when_to_use_negative:
- Input is raw scRNA-seq counts rather than per-cell signature scores — the pairwise
  DEG signatures and AUCell scoring steps must be run first to build the feature matrix.
- 'Classifying non-CD8 T cells (Treg, CD4T, NKT): TILPRED routes these to curated
  signature-based classification, not the cross-validated glmnet CD8-state model.'
- A maintained alternative is needed — the TILPRED README states it is no longer maintained
  and recommends ProjecTILs instead.
edam_operation: http://edamontology.org/operation_3947
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: glmnet
  role: Fits the multinomial logistic regression with lasso penalty (alpha=1) and
    performs the 10-fold cross-validated lambda selection on the 70% training subset
- name: AUCell
  role: Produces the per-cell gene-ranking AUC feature matrix that the regularized
    model is trained on
- name: MAST
  role: Derives the 12 pairwise DEG signature gene sets across the four reference
    CD8 TIL states that define the model's features
- name: SingleCellExperiment
  role: Container for the annotated reference B16 CD8 TIL data (B16CD8TIL_SCE) used
    as training input
- name: doParallel
  role: Parallel backend for the AUCell scoring step during feature construction
- name: doRNG
  role: Reproducible parallel evaluation of AUCell scoring
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1080/2162402x.2020.1737369
    title: Deciphering the transcriptomic landscape of tumor-infiltrating CD8 lymphocytes
      in B16 melanoma tumors with single-cell RNA-Seq
  source_content_hash: sha256:9aa7290d46262ce005edc3de9a62be7fa80c73f004fd326b1555ad24c119aa10
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/carmonalab/TILPRED
  tools:
  - AUCell
  - MAST
  - R
  - SingleCellExperiment
  - TILPRED
  - doParallel
  - doRNG
  - glmnet
  edam_operation: http://edamontology.org/operation_3947
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cross-validated-regularization-tuning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a glmnet multinomial logistic regression with lasso (alpha=1) regularization on AUCell-derived feature matrices, selecting the penalty (lambda) by 10-fold cross-validation on a 70% training subset — the approach used to fit the TILPRED CD8 TIL state classifier from 12 pairwise MAST DEG signature scores.

## When to use

Use when training a penalized multinomial classifier (e.g. glmnet) on per-cell signature-score features and the regularization strength must be tuned by k-fold cross-validation rather than fixed a priori.

## When NOT to use

- Input is raw scRNA-seq counts rather than per-cell signature scores — the pairwise DEG signatures and AUCell scoring steps must be run first to build the feature matrix.
- Classifying non-CD8 T cells (Treg, CD4T, NKT): TILPRED routes these to curated signature-based classification, not the cross-validated glmnet CD8-state model.
- A maintained alternative is needed — the TILPRED README states it is no longer maintained and recommends ProjecTILs instead.

## Inputs

- SingleCellExperiment object of annotated reference CD8 TIL cells (B16CD8TIL_SCE)
- Per-cell AUCell AUC feature matrix computed from the 12 pairwise MAST DEG signature gene sets
- Cell-level state labels for the four reference CD8 TIL states (exhausted, memory-like, naive, EM-like)

## Outputs

- Fitted glmnet multinomial logistic regression model with lasso penalty and cross-validated lambda
- Serialized TILPRED classifier object (tilpred_model.rds) exposing a predictTilState-compatible structure

## How to apply

First build the feature matrix: run MAST differential expression pairwise across the four reference CD8 TIL states (exhausted, memory-like, naive, EM-like) to derive 12 signature gene sets (6 state pairs x 2 directions), then score every cell against all 12 signatures with AUCell gene-ranking AUC, parallelized with doParallel/doRNG. Split the cells into a 70% training subset and fit a multinomial logistic regression in glmnet with alpha=1 (lasso), letting 10-fold cross-validation on the training subset select lambda. Serialize the fitted model object (tilpred_model.rds) so it can be reloaded and used for prediction; the lasso penalty performs embedded feature selection over the 12 AUCell features while cross-validation guards against overfitting to the B16CD8TIL_SCE reference cells.

## Related tools

- **glmnet** (Fits the multinomial logistic regression with lasso penalty (alpha=1) and performs the 10-fold cross-validated lambda selection on the 70% training subset)
- **AUCell** (Produces the per-cell gene-ranking AUC feature matrix that the regularized model is trained on)
- **MAST** (Derives the 12 pairwise DEG signature gene sets across the four reference CD8 TIL states that define the model's features)
- **SingleCellExperiment** (Container for the annotated reference B16 CD8 TIL data (B16CD8TIL_SCE) used as training input)
- **doParallel** (Parallel backend for the AUCell scoring step during feature construction)
- **doRNG** (Reproducible parallel evaluation of AUCell scoring)

## Examples

```
library(glmnet); fit <- cv.glmnet(x = auc_feature_matrix, y = as.factor(states), family = "multinomial", alpha = 1, nfolds = 10); saveRDS(fit, "tilpred_model.rds")
```

## Evaluation signals

- The fitted glmnet object was fit with alpha=1 and a cross-validated lambda from 10-fold CV on the 70% training subset of cells.
- Feature matrix has exactly 12 columns (6 state pairs x 2 directions of MAST DEG signatures), one AUCell AUC score per cell per signature.
- Serialized model (tilpred_model.rds) reloads in R and exposes a predictTilState-compatible structure.
- predictTilState assigns each cell a probability score across the four CD8 TIL states (exhausted, memory-like, naive, EM-like) summing to one.
- Cross-validation predictions on the held-out 30% of reference cells assign cells to the correct annotated TIL state.

## Limitations

- The reference map and trained model are limited to the B16 melanoma model and a single mouse strain; 'Future studies of additional mouse strains and cancer types will help us further refine the CD8 TIL landscape'.
- TILPRED is no longer maintained; the README recommends ProjecTILs instead.
- Performance depends on the quality of the 12 MAST pairwise DEG signatures; the model operates on AUCell AUC scores, not raw expression, so signature choice dictates the feature space.
- TILPRED includes filtering parameters (filterCD8T, scoreThreshold, cellCycleThreshold) that must be considered so non-CD8 cells are not fed to the CD8-state classifier.

## Evidence

- [methods] training set of 70% of the cells with 10-fold CV lambda selection: [evidence span withheld — anchor status: not_found]
- [methods] glmnet alpha=1 with 10-fold cross-validation: [evidence span withheld — anchor status: not_found]
- [methods] AUCell feature construction step: [evidence span withheld — anchor status: not_found]
- [methods] serialization and validation of the fitted model: [evidence span withheld — anchor status: not_found]
- [readme] classifier assigns state probability scores from SingleCellExperiment input: [evidence span withheld — anchor status: not_found]
