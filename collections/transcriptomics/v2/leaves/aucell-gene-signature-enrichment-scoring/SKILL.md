---
name: aucell-gene-signature-enrichment-scoring
description: Use when single-cell expression data (SingleCellExperiment) must be converted
  into per-cell gene-signature enrichment scores, e.g. as classifier features, for
  state assignment, or for labeling Treg/CD4T/NKT/non-CD8 cells by curated signature
  enrichment.
when_to_use_negative:
- Input is not single-cell expression data or cannot be assembled as a SingleCellExperiment
  object (AUCell requires per-cell gene rankings)
- The goal is reference-map embedding rather than signature-score classification —
  the TILPRED README recommends ProjecTILs instead, since TILPRED is no longer maintained
- Cell types of interest fall outside CD8 T cell states / curated T-cell signatures
  — non-CD8 cells will only be labeled Treg, CD4T, NKT or Non-Tcell, not further resolved
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: AUCell
  role: Computes gene-ranking AUC enrichment scores of the 12 pairwise DEG signature
    sets per cell
  repo: https://bioconductor.org/packages/release/bioc/html/AUCell.html
- name: TILPRED
  role: Wraps AUCell scoring inside predictTilState to assign CD8 TIL states and filter
    non-CD8 T cells
  repo: https://github.com/carmonalab/TILPRED
- name: SingleCellExperiment
  role: Input/output container holding the expression matrix, predictedState labels
    and stateProbabilityMatrix
  repo: https://doi.org/doi:10.18129/B9.bioc.SingleCellExperiment
- name: doParallel
  role: Parallel backend for AUCell scoring across cells and signature sets
  repo: https://cran.r-project.org/web/packages/doParallel/index.html
- name: doRNG
  role: Reproducible parallel RNG for AUCell ranking computations
  repo: https://cran.r-project.org/web/packages/doRNG/index.html
- name: MAST
  role: Derives the 12 pairwise DEG signature gene sets across the four CD8 TIL states
    before AUCell scoring
provenance:
  source_task_ids:
  - task_006
  - task_008
  - task_004
  - task_005
  source_papers:
  - doi: 10.1080/2162402x.2020.1737369
    title: Deciphering the transcriptomic landscape of tumor-infiltrating CD8 lymphocytes
      in B16 melanoma tumors with single-cell RNA-Seq
  source_content_hash: sha256:c65adff2a8bc9e2c9c22ede199eb95902c717c8cb64f03868bc2c078eeb1b284
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
  - BiocManager
  - MAST
  - R
  - SingleCellExperiment
  - TILPRED
  - doParallel
  - doRNG
  - glmnet
  - remotes
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# AUCell gene-signature enrichment scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Score every cell against curated gene-signature sets using AUCell gene-ranking AUC to build a per-cell enrichment feature matrix — the core feature-engineering step behind TILPRED's classification of CD8 TIL states (exhausted, memory-like, naive, EM-like) and its filtering of non-CD8 T cells.

## When to use

Use when single-cell expression data (SingleCellExperiment) must be converted into per-cell gene-signature enrichment scores, e.g. as classifier features, for state assignment, or for labeling Treg/CD4T/NKT/non-CD8 cells by curated signature enrichment.

## When NOT to use

- Input is not single-cell expression data or cannot be assembled as a SingleCellExperiment object (AUCell requires per-cell gene rankings)
- The goal is reference-map embedding rather than signature-score classification — the TILPRED README recommends ProjecTILs instead, since TILPRED is no longer maintained
- Cell types of interest fall outside CD8 T cell states / curated T-cell signatures — non-CD8 cells will only be labeled Treg, CD4T, NKT or Non-Tcell, not further resolved

## Inputs

- SingleCellExperiment object containing single-cell expression matrix (mouse B16 CD8 TIL counts or human plate-based smart-seq2 counts, e.g. GSE86042)
- Gene-signature sets: 12 pairwise MAST DEG sets (6 state pairs x 2 directions) or curated non-CD8 T cell signatures (Treg, CD4T, NKT)
- Mode and threshold parameters: human=TRUE/FALSE, scoreThreshold (e.g. 0.3), cellCycleThreshold, filterCD8T

## Outputs

- Per-cell AUC feature matrix over all signature sets
- Per-cell predictedState assignments (exhausted, memory-like, naive, EM-like, or Treg/CD4T/NKT/Tcell_unknown/Non-Tcell)
- stateProbabilityMatrix stored in the returned SingleCellExperiment
- Serialized glmnet lasso classifier (tilpred_model.rds) when used in training mode

## How to apply

Assemble the expression matrix as a SingleCellExperiment object, then run AUCell gene-ranking AUC scoring in parallel with doParallel/doRNG over each signature set. In the TILPRED training pipeline, MAST pairwise differential expression across the four B16 CD8 TIL states (exhausted, memory-like, naive, EM-like) yields 12 signature gene sets (6 state pairs x 2 directions), and every cell is scored against all 12 to build the per-cell feature matrix fed to a glmnet multinomial lasso classifier (alpha=1, lambda chosen by 10-fold CV on a 70% training subset). At prediction time, predictTilState(data, nCores, human, scoreThreshold, cellCycleThreshold, filterCD8T) uses AUCell enrichment of curated signatures to auto-filter non-CD8 T cells (Treg/Foxp3, CD4T, NKT, Non-Tcell) before assigning states, with a scoreThreshold (e.g. 0.3 in the human GSE120575 analysis) gating state assignment. Ratios of AUC scores across the state signatures determine predictedState labels and the per-cell stateProbabilityMatrix returned in the SingleCellExperiment.

## Related tools

- **AUCell** (Computes gene-ranking AUC enrichment scores of the 12 pairwise DEG signature sets per cell) — https://bioconductor.org/packages/release/bioc/html/AUCell.html
- **TILPRED** (Wraps AUCell scoring inside predictTilState to assign CD8 TIL states and filter non-CD8 T cells) — https://github.com/carmonalab/TILPRED
- **SingleCellExperiment** (Input/output container holding the expression matrix, predictedState labels and stateProbabilityMatrix) — https://doi.org/doi:10.18129/B9.bioc.SingleCellExperiment
- **doParallel** (Parallel backend for AUCell scoring across cells and signature sets) — https://cran.r-project.org/web/packages/doParallel/index.html
- **doRNG** (Reproducible parallel RNG for AUCell ranking computations) — https://cran.r-project.org/web/packages/doRNG/index.html
- **MAST** (Derives the 12 pairwise DEG signature gene sets across the four CD8 TIL states before AUCell scoring)

## Examples

```
remotes::install_github("carmonalab/TILPRED"); data(B16CD8TIL_SCE); sce <- predictTilState(data=B16CD8TIL_SCE, nCores=4, human=FALSE, scoreThreshold=0.3, filterCD8T=TRUE)
```

## Evaluation signals

- On the 30% held-out B16CD8TIL_SCE test split, accuracy vs annotated labels reaches 0.94 (95% CI 0.92–0.95), mean specificity 0.98, mean sensitivity 0.90
- On independent Singer et al. GSE86042 smart-seq2 data, accuracy vs consensus clustering ground truth (SNN, K-means, hierarchical) is approximately 91%
- Reloaded tilpred_model.rds exposes a predictTilState-compatible structure assigning cells to one of the four CD8 TIL states
- Every cell receives a state probability row in the stateProbabilityMatrix and a predictedState label; cells below scoreThreshold (e.g. 0.3) are not assigned a confident state
- Human-mode run on GSE120575 reproduces per-patient EM-like proportions yielding one-sided Wilcoxon p = 0.0317 for responders vs non-responders (P1, P7, P28 vs P2, P3, P12, P20, ≥30 cells/sample)

## Limitations

- TILPRED (and its AUCell-based classifier) is no longer maintained; the README recommends ProjecTILs as the replacement
- Reference map/classifier is trained on the B16 melanoma model (wild-type C57BL/6 and PMEL) only — future studies of additional mouse strains and cancer types are needed to refine the CD8 TIL landscape
- Non-CD8 T cells can only be labeled by curated signature enrichment into Treg, CD4T, NKT or Non-Tcell; cell-cycle signal must be controlled via cellCycleThreshold
- Scoring depends on the training-set-derived signatures (12 MAST pairwise DEG sets); datasets from different sequencing technologies (plate-based smart-seq2 vs droplet 10X 5'/3') may need mode/threshold adjustment (human=TRUE, scoreThreshold=0.3 used for GSE120575)

## Evidence

- [methods] AUCell scoring in training pipeline: [evidence span withheld — anchor status: not_found]
- [methods] predictTilState signature and non-CD8 filtering: "predictTilState(data, nCores, human, scoreThreshold, cellCycleThreshold, filterCD8T)"
- [readme] TILPRED classifier description from README: "`TILPRED` is a logistic regression-based classifier that reads a [SingleCellExperiment](https://doi.org/doi:10.18129/B9.bioc.SingleCellExperiment) object containing CD8 T cell profiles"
- [methods] Non-CD8 T cell classification by curated signature enrichment: "Non CD8 T cells are classified based on curated gene signature enrichment into: _Treg_ (_Foxp3_ Regulatory T cells)"
- [discussion] TILPRED as tool for CD8 TIL classification: "Our new CD8 TIL classifier (https://github.com/carmonalab/TILPRED) might be an instrumental tool for this task."
- [abstract] Four reference CD8 TIL states the signatures distinguish: "Unsupervised clustering and gene-signature analysis revealed four distinct CD8 TIL states - exhausted, memory-like, naïve and effector memory-like (EM-like)"
