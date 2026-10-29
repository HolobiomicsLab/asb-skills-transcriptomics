---
name: classifier-accuracy-benchmarking
description: Use when you have a trained cell-state classifier (e.g. TILPRED) and
  an independent scRNA-seq dataset with consensus labels from multiple clustering
  methods (SNN, K-means, hierarchical), and need a quantitative accuracy estimate
  of the classifier's agreement with that ground truth.
when_to_use_negative:
- Input is already a cluster/partition result and no classifier predictions are being
  evaluated — this skill benchmarks classifier agreement, not clustering quality
- Data are human scRNA-seq profiles where the mouse CD8 TIL reference states do not
  apply without appropriate human-mode settings (predictTilState supports a human
  flag) — verify species mapping before comparing to ground truth
- 'A maintained classifier is needed: the TILPRED README states it is no longer maintained
  and recommends ProjecTILs instead'
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: TILPRED
  role: Logistic regression-based CD8 TIL state classifier whose predictions are benchmarked
    against clustering consensus
  repo: https://github.com/carmonalab/TILPRED
- name: remotes
  role: Installs TILPRED directly from its Git repository
- name: BiocManager
  role: Installs the Bioconductor dependencies (AUCell, SingleCellExperiment)
- name: AUCell
  role: Gene-signature enrichment scoring used by TILPRED to auto-filter non-CD8 T
    cells and score states
- name: SingleCellExperiment
  role: Container format for input count data and returned predictions/stateProbabilityMatrix
- name: doParallel
  role: Parallel backend required by TILPRED (nCores argument of predictTilState)
- name: doRNG
  role: Reproducible parallel RNG required by TILPRED
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1080/2162402x.2020.1737369
    title: Deciphering the transcriptomic landscape of tumor-infiltrating CD8 lymphocytes
      in B16 melanoma tumors with single-cell RNA-Seq
  source_content_hash: sha256:49909c106e023e1482ae1b8569d9e8518d66d31716d28809b7e3175fbde96766
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
  - R
  - SingleCellExperiment
  - TILPRED
  - doParallel
  - doRNG
  - remotes
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# classifier-accuracy-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a single-cell transcriptomic state classifier (TILPRED) by comparing its per-cell predicted labels to an unsupervised consensus-clustering ground truth on an independent dataset, reporting overall classification accuracy (~91% for GSE86042).

## When to use

Use when you have a trained cell-state classifier (e.g. TILPRED) and an independent scRNA-seq dataset with consensus labels from multiple clustering methods (SNN, K-means, hierarchical), and need a quantitative accuracy estimate of the classifier's agreement with that ground truth.

## When NOT to use

- Input is already a cluster/partition result and no classifier predictions are being evaluated — this skill benchmarks classifier agreement, not clustering quality
- Data are human scRNA-seq profiles where the mouse CD8 TIL reference states do not apply without appropriate human-mode settings (predictTilState supports a human flag) — verify species mapping before comparing to ground truth
- A maintained classifier is needed: the TILPRED README states it is no longer maintained and recommends ProjecTILs instead

## Inputs

- SingleCellExperiment object with a count matrix (e.g. plate-based smart-seq2 counts from GEO accession GSE86042)
- Consensus ground-truth cell labels from SNN, K-means and hierarchical clustering
- Installed TILPRED R package with AUCell, SingleCellExperiment, doParallel, doRNG dependencies

## Outputs

- Per-cell predictedState assignments from TILPRED
- State probability matrix (per-cell scores for Exhausted, MemoryLike, EffectorMemory, NaiveLike)
- Overall classification accuracy of TILPRED vs. the clustering consensus (~91% on GSE86042)

## How to apply

Install TILPRED from GitHub via remotes::install_github('carmonalab/TILPRED') and load its Bioconductor dependencies (AUCell, SingleCellExperiment, doParallel, doRNG) with BiocManager. Assemble the independent dataset (Singer et al. GSE86042, plate-based smart-seq2 counts from GEO) into a SingleCellExperiment object. Run predictTilState(data=<SCE>), letting the auto-filter (filterCD8T) remove non-CD8 T cells — Treg (Foxp3), CD4T, NKT, Non-Tcell — via curated gene-signature enrichment before scoring the four reference states (Exhausted/Tex, MemoryLike/Tpex, EffectorMemory, NaiveLike). Extract per-cell predictedState assignments and the stateProbabilityMatrix from the returned SCE. Compare predicted labels to the consensus ground truth built from SNN, K-means and hierarchical clustering and compute overall classification accuracy; the acceptance criterion is ~91% agreement as reported for TILPRED's construction and benchmarking.

## Related tools

- **TILPRED** (Logistic regression-based CD8 TIL state classifier whose predictions are benchmarked against clustering consensus) — https://github.com/carmonalab/TILPRED
- **remotes** (Installs TILPRED directly from its Git repository)
- **BiocManager** (Installs the Bioconductor dependencies (AUCell, SingleCellExperiment))
- **AUCell** (Gene-signature enrichment scoring used by TILPRED to auto-filter non-CD8 T cells and score states)
- **SingleCellExperiment** (Container format for input count data and returned predictions/stateProbabilityMatrix)
- **doParallel** (Parallel backend required by TILPRED (nCores argument of predictTilState))
- **doRNG** (Reproducible parallel RNG required by TILPRED)

## Examples

```
remotes::install_github("carmonalab/TILPRED"); data(B16CD8TIL_SCE); sce <- predictTilState(data = B16CD8TIL_SCE)
```

## Evaluation signals

- Overall classification accuracy of TILPRED predicted labels vs. SNN/K-means/hierarchical clustering consensus reaches approximately 91% on the independent GSE86042 dataset
- Returned object is a SingleCellExperiment containing per-cell predictedState and stateProbabilityMatrix columns
- Non-CD8 T cells (Treg/Foxp3, CD4T, NKT, Non-Tcell) are correctly reassigned by the signature-based auto-filter before state scoring
- Predicted labels fall within the four reference states: Exhausted, MemoryLike, EffectorMemory, NaiveLike
- Reproducibility check: run uses doRNG-backed parallel workers so repeated runs give identical state probability matrices

## Limitations

- TILPRED is no longer maintained; the authors recommend ProjecTILs (https://github.com/carmonalab/ProjecTILs) as the replacement
- The reference CD8 TIL map is limited to the B16 melanoma model and a single mouse strain; the article notes future studies of additional mouse strains and cancer types are needed to refine the landscape
- Accuracy is measured against an unsupervised clustering consensus, which is a proxy ground truth, not validated true cell states
- A test/example workflow is only available as an R Notebook at carmonalab/testTILPRED

## Evidence

- [readme] TILPRED is a logistic regression-based classifier that reads a SingleCellExperiment object containing CD8 T cell profiles and assigns to each cell a probability score of belonging to any of the following reference CD8 TIL transcriptomic states: [evidence span withheld — anchor status: not_found]
