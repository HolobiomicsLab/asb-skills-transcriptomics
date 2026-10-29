---
name: lasso-multinomial-logistic-regression-training
description: 'Use when building a single-cell state classifier from gene-signature
  enrichment scores: cells are scored against several pairwise DEG signatures (e.g.
  via AUCell) and must be assigned to one of 3+ annotated states.'
when_to_use_negative:
- Input cells are not CD8 T cells — TILPRED's predictTilState filter steps (filterCD8T,
  Treg/CD4T/NKT classification by curated signatures) must run first; the model itself
  only predicts the four CD8 TIL states
- You need to embed/query new datasets against a maintained reference atlas — TILPRED
  is no longer maintained and the authors recommend ProjecTILs instead
- The classifier was trained on B16 murine melanoma smart-seq2 reference data; human
  samples require the `human` adaptation parameter and cross-species mapping, and
  performance is unguaranteed outside the mouse B16 CD8 TIL scope
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3308
tools:
- name: TILPRED
  role: R package whose model-building function trains and serializes the lasso multinomial
    classifier
  repo: https://github.com/carmonalab/TILPRED
- name: MAST
  role: runs pairwise differential expression across the four CD8 TIL states to define
    the 12 signature gene sets
- name: AUCell
  role: computes gene-ranking AUC enrichment scores of the 12 signatures per cell
    to build the feature matrix
  repo: https://bioconductor.org/packages/release/bioc/html/AUCell.html
- name: glmnet
  role: fits the multinomial logistic regression with lasso penalty (alpha=1) and
    cross-validated lambda
- name: doParallel
  role: parallelizes AUCell signature scoring during feature-matrix construction
- name: doRNG
  role: ensures reproducible parallel scoring of signatures
- name: SingleCellExperiment
  role: container format for the annotated reference CD8 TIL data and the scored feature
    matrix
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1080/2162402x.2020.1737369
    title: Deciphering the transcriptomic landscape of tumor-infiltrating CD8 lymphocytes
      in B16 melanoma tumors with single-cell RNA-Seq
  source_content_hash: sha256:b5b97212a99f4e9b6e0cdcff85e6470551d8ed4c24c31098ace4c456c1c09a17
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/carmonalab/TILPRED
  repo_url: https://github.com/carmonalab/TILPRED
  tools:
  - AUCell
  - MAST
  - R
  - SingleCellExperiment
  - TILPRED
  - doParallel
  - doRNG
  - glmnet
  edam_operation: http://edamontology.org/operation_2995
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# lasso-multinomial-logistic-regression-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains the TILPRED CD8 TIL state classifier: a multinomial logistic regression with lasso penalty (glmnet alpha=1) fit on per-cell AUCell AUC scores of 12 MAST-derived pairwise differential-expression signature gene sets, with lambda chosen by 10-fold cross-validation on a 70% training subset.

## When to use

Use when building a single-cell state classifier from gene-signature enrichment scores: cells are scored against several pairwise DEG signatures (e.g. via AUCell) and must be assigned to one of 3+ annotated states.

## When NOT to use

- Input cells are not CD8 T cells — TILPRED's predictTilState filter steps (filterCD8T, Treg/CD4T/NKT classification by curated signatures) must run first; the model itself only predicts the four CD8 TIL states
- You need to embed/query new datasets against a maintained reference atlas — TILPRED is no longer maintained and the authors recommend ProjecTILs instead
- The classifier was trained on B16 murine melanoma smart-seq2 reference data; human samples require the `human` adaptation parameter and cross-species mapping, and performance is unguaranteed outside the mouse B16 CD8 TIL scope

## Inputs

- Annotated SingleCellExperiment object of reference CD8 TIL profiles (B16CD8TIL_SCE) with state labels (exhausted, memory-like, naive, EM-like)
- 12 pairwise MAST DEG signature gene sets (6 state pairs x 2 directions)
- Per-cell AUCell gene-ranking AUC feature matrix (12 columns, one per signature)

## Outputs

- Fitted lasso multinomial logistic regression model (glmnet object) serialized as tilpred_model.rds
- TILPRED classifier object exposing a predictTilState-compatible structure assigning each cell to one of four CD8 TIL states with probability scores

## How to apply

Start from the annotated reference SingleCellExperiment (B16CD8TIL_SCE) and run MAST differential expression pairwise across the four reference CD8 TIL states (exhausted, memory-like, naive, EM-like) to derive 12 signature gene sets (6 state pairs x 2 directions). Score every cell against all 12 signatures with AUCell gene-ranking AUC, parallelized with doParallel/doRNG, to build the per-cell feature matrix. Train cv.glmnet with family='multinomial' and alpha=1 (lasso), selecting lambda by 10-fold cross-validation on a 70% training subset of cells. Serialize the fitted model as the TILPRED classifier object (tilpred_model.rds); the rationale for lasso is feature selection over the 12 correlated AUC features and probabilistic assignment of cells to one of the four states.

## Related tools

- **TILPRED** (R package whose model-building function trains and serializes the lasso multinomial classifier) — https://github.com/carmonalab/TILPRED
- **MAST** (runs pairwise differential expression across the four CD8 TIL states to define the 12 signature gene sets)
- **AUCell** (computes gene-ranking AUC enrichment scores of the 12 signatures per cell to build the feature matrix) — https://bioconductor.org/packages/release/bioc/html/AUCell.html
- **glmnet** (fits the multinomial logistic regression with lasso penalty (alpha=1) and cross-validated lambda)
- **doParallel** (parallelizes AUCell signature scoring during feature-matrix construction)
- **doRNG** (ensures reproducible parallel scoring of signatures)
- **SingleCellExperiment** (container format for the annotated reference CD8 TIL data and the scored feature matrix)

## Examples

```
model <- cv.glmnet(x = auCellScores_12sig, y = tilStates, family = "multinomial", alpha = 1, nfolds = 10); saveRDS(model, "tilpred_model.rds")
```

## Evaluation signals

- Reload tilpred_model.rds and confirm the object exposes a predictTilState-compatible predict interface
- The model assigns every scored cell to exactly one of the four reference states (exhausted, memory-like, naive, EM-like) with probability scores summing to 1
- Feature matrix has exactly 12 AUCell AUC columns (6 state pairs x 2 directions) matching the trained model's coefficients
- Lambda is chosen by 10-fold cross-validation on the 70% training subset; held-out 30% of cells should be classified at sensible accuracy before deployment

## Limitations

- Reference map is limited to the B16 melanoma model and a single mouse strain; the authors note future studies of additional mouse strains and cancer types are needed to refine the CD8 TIL landscape
- TILPRED is no longer maintained; the authors recommend ProjecTILs instead
- Training set is only 70% of cells, so performance depends on the AUCell AUC features of 12 pairwise signatures rather than raw expression

## Evidence

- [other] lasso training on AUCell features: [evidence span withheld — anchor status: not_found]
