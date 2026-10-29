---
name: cd8-til-state-classification
description: Use when you have a SingleCellExperiment of CD8 T cell scRNA-seq profiles
  (mouse or human) and need to assign each cell a probability of belonging to a reference
  CD8 TIL state (exhausted, memory-like, naive, EM-like).
when_to_use_negative:
- TILPRED is explicitly no longer maintained; for new reference-based analyses of
  non-CD8 or multi-state data prefer its successor ProjecTILs (carmonalab/ProjecTILs)
- Input cells are not CD8 T lymphocytes or the goal is unbiased de novo clustering
  rather than mapping to the four reference CD8 TIL states (use unsupervised clustering,
  e.g. Seurat SNN with silhouette-based K selection)
- The task is TCR sequence reconstruction rather than state classification (use TraCeR
  for reconstructing TCR sequences from single-cell RNA-seq)
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: TILPRED
  role: Logistic regression classifier invoked via predictTilState() to assign CD8
    TIL state probabilities and labels
  repo: https://github.com/carmonalab/TILPRED
- name: AUCell
  role: Bioconductor package wrapped by TILPRED/scAUCscore to compute per-cell AUC
    signature enrichment scores over the top 1500 ranked genes
  repo: https://bioconductor.org/packages/release/bioc/html/AUCell.html
- name: SingleCellExperiment
  role: Bioconductor container for the input expression data and the object returned
    by predictTilState()
  repo: https://doi.org/doi:10.18129/B9.bioc.SingleCellExperiment
- name: doParallel
  role: Parallel backend required by TILPRED for multi-core scoring (nCores argument)
  repo: https://cran.r-project.org/web/packages/doParallel/index.html
- name: doRNG
  role: Reproducible parallel RNG dependency of TILPRED
  repo: https://cran.r-project.org/web/packages/doRNG/index.html
- name: remotes
  role: Used to install TILPRED directly from its Git repository
  repo: https://cran.r-project.org/web/packages/remotes/index.html
- name: testTILPRED
  role: R Notebook repository demonstrating TILPRED usage
  repo: https://github.com/carmonalab/testTILPRED
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_001
  - task_007
  - task_002
  source_papers:
  - doi: 10.1080/2162402x.2020.1737369
    title: Deciphering the transcriptomic landscape of tumor-infiltrating CD8 lymphocytes
      in B16 melanoma tumors with single-cell RNA-Seq
  source_content_hash: sha256:a5cf6c6f93ac1a7462d5bd3eb22185af44d1f2dea04bf56ef37e080dc8e1a76c
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
  - BiocManager
  - DESeq2
  - R
  - Salmon
  - SingleCellExperiment
  - TILPRED
  - Trimmomatic
  - clusterProfiler
  - doParallel
  - doRNG
  - ggradar
  - remotes
  - smart-seq2
  - testTILPRED
  - tximport
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5294117647058824
n_claims: 17
---

# cd8-til-state-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies single-cell CD8 tumor-infiltrating lymphocyte (TIL) profiles into four reference transcriptomic states — exhausted, memory-like, naive and effector memory-like (EM-like) — using the TILPRED multinomial logistic regression classifier, which scores each cell via AUCell-based gene-signature enrichment.

## When to use

Use when you have a SingleCellExperiment of CD8 T cell scRNA-seq profiles (mouse or human) and need to assign each cell a probability of belonging to a reference CD8 TIL state (exhausted, memory-like, naive, EM-like).

## When NOT to use

- TILPRED is explicitly no longer maintained; for new reference-based analyses of non-CD8 or multi-state data prefer its successor ProjecTILs (carmonalab/ProjecTILs)
- Input cells are not CD8 T lymphocytes or the goal is unbiased de novo clustering rather than mapping to the four reference CD8 TIL states (use unsupervised clustering, e.g. Seurat SNN with silhouette-based K selection)
- The task is TCR sequence reconstruction rather than state classification (use TraCeR for reconstructing TCR sequences from single-cell RNA-seq)

## Inputs

- SingleCellExperiment object with CD8 T cell scRNA-seq count matrix (mouse or human gene symbols), e.g. the shipped B16CD8TIL_SCE example dataset
- Plate-based smart-seq2 or droplet-based (10X 5' and 3') count matrices, e.g. GSE86042 (Singer et al.) or GSE116390 (B16 CD8 TILs)
- Optional held-out reference state labels (exhausted, memory-like, naive, EM-like) for benchmarking

## Outputs

- Per-cell predictedState assignments restricted to {exhausted, memory-like, naive, EM-like} (auto-filtered non-CD8 T cells get Treg/CD4T/NKT/Non-Tcell labels)
- stateProbabilityMatrix with a probability score per cell for each reference CD8 TIL state
- Optional confusion-matrix metrics (accuracy with 95% CI, mean specificity, mean sensitivity) e.g. written to tilpred_test_metrics.csv

## How to apply

Install TILPRED from GitHub (remotes::install_github('carmonalab/TILPRED')) together with its Bioconductor dependencies AUCell, SingleCellExperiment, doParallel and doRNG, and load the B16CD8TIL_SCE example SingleCellExperiment to validate the setup. Build a SingleCellExperiment containing the CD8 T cell expression matrix with mouse (or human) gene symbols, then call predictTilState(data, nCores, human, scoreThreshold, cellCycleThreshold, filterCD8T); with mouse data, TILPRED auto-filters non-CD8 T cells (Treg/Foxp3, CD4T, NKT, Non-Tcell) via curated gene-signature enrichment before scoring states. The classifier wraps AUCell, ranking genes per cell and computing AUC over the top-ranked window (aucMaxRank at the top 1500 expressed genes), then applies a multinomial logistic regression over signature scores to return predictedState labels and a stateProbabilityMatrix. Benchmark against held-out labels by building a confusion matrix over the four states and computing accuracy with 95% CI, mean specificity and mean sensitivity; the paper's model achieved accuracy 0.94 (95% CI 0.92–0.95), specificity 0.98, sensitivity 0.90 on a 30% test split, and 91% accuracy versus consensus clustering (SNN, K-means, hierarchical) on the independent Singer et al. GSE86042 smart-seq2 dataset. Note that TILPRED is no longer maintained and its successor is ProjecTILs.

## Related tools

- **TILPRED** (Logistic regression classifier invoked via predictTilState() to assign CD8 TIL state probabilities and labels) — https://github.com/carmonalab/TILPRED
- **AUCell** (Bioconductor package wrapped by TILPRED/scAUCscore to compute per-cell AUC signature enrichment scores over the top 1500 ranked genes) — https://bioconductor.org/packages/release/bioc/html/AUCell.html
- **SingleCellExperiment** (Bioconductor container for the input expression data and the object returned by predictTilState()) — https://doi.org/doi:10.18129/B9.bioc.SingleCellExperiment
- **doParallel** (Parallel backend required by TILPRED for multi-core scoring (nCores argument)) — https://cran.r-project.org/web/packages/doParallel/index.html
- **doRNG** (Reproducible parallel RNG dependency of TILPRED) — https://cran.r-project.org/web/packages/doRNG/index.html
- **remotes** (Used to install TILPRED directly from its Git repository) — https://cran.r-project.org/web/packages/remotes/index.html
- **testTILPRED** (R Notebook repository demonstrating TILPRED usage) — https://github.com/carmonalab/testTILPRED

## Examples

```
remotes::install_github("carmonalab/TILPRED"); library(TILPRED); data(B16CD8TIL_SCE); sce <- predictTilState(data = B16CD8TIL_SCE, human = FALSE)
```

## Evaluation signals

- On the B16CD8TIL_SCE held-out 30% test split, expect accuracy ≈ 0.94 (95% CI 0.92–0.95), mean specificity 0.98 and mean sensitivity 0.90 across the four states
- Against consensus clustering (SNN, K-means, hierarchical) ground truth on GSE86042, expect ≈ 91% overall classification accuracy
- Every classified CD8 T cell receives exactly one label from {exhausted, memory-like, naive, EM-like}; non-CD8 T cells are diverted to Treg/CD4T/NKT/Non-Tcell via signature-enrichment filtering
- Predicted probabilities from stateProbabilityMatrix are valid probability scores per state; naive cells should be Tcf7/Lef1/Il7r-high with no cytotoxicity/activation markers, exhausted cells Pdcd1/Havcr2-high
- Mouse-mode runs should recover the known biology, e.g. EM-like cells cytotoxicity-high (Gzmk/Gzmb, Ly6c2) with low-to-intermediate PD-1 and no PD-1 expression distinguishing them from exhausted cells

## Limitations

- TILPRED is no longer maintained; the authors recommend ProjecTILs instead
- The reference map is limited to the B16 melanoma model and a single mouse strain (C57BL/6); future studies of additional mouse strains and cancer types are needed to refine the CD8 TIL landscape
- Cycling cells can confound signature averages and are removed before cluster-level phenotypic profiling (cellCycleThreshold parameter)
- Non-CD8 T cells (Treg, CD4T, NKT, Non-Tcell) are classified only approximately via curated gene-signature enrichment rather than dedicated models

## Evidence

- [readme] reference states: [evidence span withheld — anchor status: not_found]
