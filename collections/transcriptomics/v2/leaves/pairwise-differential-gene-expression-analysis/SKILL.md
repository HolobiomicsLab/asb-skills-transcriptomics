---
name: pairwise-differential-gene-expression-analysis
description: Use when a SingleCellExperiment of CD8 T cells has been assigned to reference
  states and you need gene signatures per state pair (or per state) to score/query
  cells or train a state classifier.
when_to_use_negative:
- TILPRED is no longer maintained — for projecting new datasets into a reference atlas
  use ProjecTILs instead
- Input contains non-CD8 T cells (Treg, CD4T, NKT) — first filter/classify them via
  predictTilState's filterCD8T and curated non-CD8 signatures before running state-pair
  DEG analysis
- Input is already a per-cell signature-score feature matrix — proceed directly to
  glmnet training rather than recomputing DEG signatures
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: MAST
  role: Pairwise differential expression testing across the four reference CD8 TIL
    states to derive signature gene sets
- name: AUCell
  role: Gene-ranking AUC scoring of every cell against the 12 pairwise signatures
    to build the classifier feature matrix
  repo: https://bioconductor.org/packages/release/bioc/html/AUCell.html
- name: glmnet
  role: Multinomial logistic regression with lasso penalty (alpha=1), lambda selected
    by 10-fold CV on 70% of cells
- name: SingleCellExperiment
  role: Container format for the reference B16CD8TIL_SCE expression data consumed
    by TILPRED
  repo: https://doi.org/doi:10.18129/B9.bioc.SingleCellExperiment
- name: doParallel
  role: Parallel backend for AUCell scoring across cells
  repo: https://cran.r-project.org/web/packages/doParallel/index.html
- name: doRNG
  role: Reproducible parallel foreach evaluation during AUCell scoring
  repo: https://cran.r-project.org/web/packages/doRNG/index.html
- name: TILPRED
  role: R package whose reference states, DEG-derived signatures and trained classifier
    object this pipeline constructs
  repo: https://github.com/carmonalab/TILPRED
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1080/2162402x.2020.1737369
    title: Deciphering the transcriptomic landscape of tumor-infiltrating CD8 lymphocytes
      in B16 melanoma tumors with single-cell RNA-Seq
  source_content_hash: sha256:0a2e4f74ab7b798d7e2bb7cced1c59d77fc5198310984d7d3453ba70e1165bad
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 20
---

# pairwise-differential-gene-expression-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Derive state-defining signature gene sets by running pairwise differential expression across the four reference CD8 TIL states (exhausted, memory-like, naive, EM-like) of the B16CD8TIL_SCE reference, then score single cells against them with AUCell to build the feature matrix that trains the TILPRED lasso classifier.

## When to use

Use when a SingleCellExperiment of CD8 T cells has been assigned to reference states and you need gene signatures per state pair (or per state) to score/query cells or train a state classifier.

## When NOT to use

- TILPRED is no longer maintained — for projecting new datasets into a reference atlas use ProjecTILs instead
- Input contains non-CD8 T cells (Treg, CD4T, NKT) — first filter/classify them via predictTilState's filterCD8T and curated non-CD8 signatures before running state-pair DEG analysis
- Input is already a per-cell signature-score feature matrix — proceed directly to glmnet training rather than recomputing DEG signatures

## Inputs

- Annotated SingleCellExperiment object of CD8 T cell profiles (e.g. B16CD8TIL_SCE reference) with state labels for exhausted, memory-like, naive and EM-like cells
- Raw count/normalized expression matrix per cell (smart-seq2 plate-based or 10X droplet data)

## Outputs

- 12 pairwise DEG signature gene sets (6 state pairs x 2 directions)
- Per-cell AUCell AUC feature matrix (one column per signature)
- Fitted multinomial lasso glmnet model serialized as tilpred_model.rds with predictTilState-compatible structure

## How to apply

Load the annotated reference B16 CD8 TIL SingleCellExperiment (data(B16CD8TIL_SCE)) and run MAST differential expression pairwise across the four reference states — exhausted, memory-like, naive, EM-like — yielding 12 signature gene sets (6 state pairs x 2 directions). Score every cell against all 12 signatures with AUCell gene-ranking AUC, parallelized with doParallel/doRNG, to build the per-cell feature matrix. Train a multinomial logistic regression with lasso penalty (glmnet, alpha=1), selecting lambda by 10-fold cross-validation on a 70% training subset, and serialize the fitted model as tilpred_model.rds. The pairwise-direction design matters because each state is defined relative to every other state, not by a one-vs-rest contrast.

## Related tools

- **MAST** (Pairwise differential expression testing across the four reference CD8 TIL states to derive signature gene sets)
- **AUCell** (Gene-ranking AUC scoring of every cell against the 12 pairwise signatures to build the classifier feature matrix) — https://bioconductor.org/packages/release/bioc/html/AUCell.html
- **glmnet** (Multinomial logistic regression with lasso penalty (alpha=1), lambda selected by 10-fold CV on 70% of cells)
- **SingleCellExperiment** (Container format for the reference B16CD8TIL_SCE expression data consumed by TILPRED) — https://doi.org/doi:10.18129/B9.bioc.SingleCellExperiment
- **doParallel** (Parallel backend for AUCell scoring across cells) — https://cran.r-project.org/web/packages/doParallel/index.html
- **doRNG** (Reproducible parallel foreach evaluation during AUCell scoring) — https://cran.r-project.org/web/packages/doRNG/index.html
- **TILPRED** (R package whose reference states, DEG-derived signatures and trained classifier object this pipeline constructs) — https://github.com/carmonalab/TILPRED

## Examples

```
remotes::install_github("carmonalab/TILPRED"); library(TILPRED); library(SingleCellExperiment); data(B16CD8TIL_SCE)
```

## Evaluation signals

- The pairwise design yields exactly 12 signature sets (6 state pairs x 2 directions), not 4 one-vs-rest sets
- The AUCell feature matrix has one AUC value per cell per signature for all cells scored
- glmnet is fit with alpha=1 (lasso) and lambda chosen by 10-fold cross-validation on the 70% training subset
- The serialized tilpred_model.rds reloads in R and exposes a predictTilState-compatible structure that assigns each cell one of the four states (exhausted, memory-like, naive, EM-like)

## Limitations

- TILPRED is no longer maintained; the authors recommend ProjecTILs (https://github.com/carmonalab/ProjecTILs) as the replacement
- The reference map is limited to the B16 melanoma model in C57BL/6 (wild-type and PMEL) mice; additional mouse strains and cancer types are needed to refine the CD8 TIL landscape
- Classifier assigns probabilities only over the four reference CD8 TIL states; non-CD8 cells (Treg, CD4T, NKT) must be filtered out first via curated signature enrichment

## Evidence
