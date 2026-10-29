---
name: cd8-til-state-annotation-from-reference-map
description: Use when murine CD8 T cell scRNA-seq data (e.g. Smart-seq2 or 10X counts)
  must be assigned to reference CD8 TIL states (exhausted, memory-like, EM-like, naïve)
  prior to clonotype or state-composition analyses.
when_to_use_negative:
- Input is bulk RNA-seq or a pre-computed cell-type feature table rather than single-cell
  expression data — the classifier needs per-cell profiles in a SingleCellExperiment
  object
- Data are human CD8 T cells without setting the human parameter, or non-T cells dominate
  the sample — non-CD8 cells are classified into Treg/CD4T/NKT and must be filtered
  before state prediction
- A stable, maintained reference-mapping workflow with detection of states deviating
  from the reference is required — TILPRED is no longer maintained and ProjecTILs
  is recommended instead
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: TILPRED
  role: Logistic-regression classifier that assigns each CD8 T cell in a SingleCellExperiment
    object a probability score for the reference TIL states (predictTilState with
    nCores, human, scoreThreshold, cellCycleThreshold, filterCD8T)
  repo: https://github.com/carmonalab/TILPRED
- name: SingleCellExperiment
  role: Bioconductor container format holding the CD8 T cell expression matrix passed
    to TILPRED
- name: AUCell
  role: Bioconductor package used with SingleCellExperiment for gene-signature enrichment
    scoring underlying the state classification
- name: ProjecTILs
  role: Maintained successor framework that projects new scRNA-seq data into reference
    CD8 TIL atlases for state annotation when TILPRED is unsuitable
  repo: https://github.com/carmonalab/ProjecTILs
- name: tracer
  role: Reconstructs paired TCR alpha/beta sequences and clonotypes from the same
    single-cell data, so annotated states can be linked to clonotypes for Morisita-Horn
    overlap analysis
  repo: https://github.com/Teichlab/tracer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1080/2162402x.2020.1737369
    title: Deciphering the transcriptomic landscape of tumor-infiltrating CD8 lymphocytes
      in B16 melanoma tumors with single-cell RNA-Seq
  source_content_hash: sha256:069aeceeb5ccaed829dc3f6d25dbfb2b09a87bdf258e14eaee5a176bcea5e94b
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
  - ProjecTILs
  - R
  - SingleCellExperiment
  - TILPRED
  - tracer
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# cd8-til-state-annotation-from-reference-map

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates individual CD8 TILs from single-cell RNA-seq with one of the four B16-melanoma reference transcriptomic states — exhausted (Tex), memory-like (Tpex), effector memory-like (EM-like) and naïve — using the TILPRED logistic-regression classifier on a SingleCellExperiment object, enabling downstream clonal-overlap analyses such as Morisita-Horn TCR repertoire comparisons.

## When to use

Use when murine CD8 T cell scRNA-seq data (e.g. Smart-seq2 or 10X counts) must be assigned to reference CD8 TIL states (exhausted, memory-like, EM-like, naïve) prior to clonotype or state-composition analyses.

## When NOT to use

- Input is bulk RNA-seq or a pre-computed cell-type feature table rather than single-cell expression data — the classifier needs per-cell profiles in a SingleCellExperiment object
- Data are human CD8 T cells without setting the human parameter, or non-T cells dominate the sample — non-CD8 cells are classified into Treg/CD4T/NKT and must be filtered before state prediction
- A stable, maintained reference-mapping workflow with detection of states deviating from the reference is required — TILPRED is no longer maintained and ProjecTILs is recommended instead

## Inputs

- SingleCellExperiment object with murine CD8 T cell scRNA-seq counts (e.g. B16CD8TIL_SCE example object, plate-based Smart-seq2 or droplet 10X data)
- Optional paired full-length TCR sequences per cell (TraCeR-reconstructed from scRNA-seq) for downstream clonal overlap analysis

## Outputs

- Per-cell CD8 TIL state annotations (exhausted, memory-like, naive, EM-like) with probability scores
- Non-CD8 T cell classifications (Treg, CD4T, NKT) after filterCD8T
- State-annotated cell table ready for clonal expansion and Morisita-Horn overlap computation

## How to apply

Load the expression matrix as a SingleCellExperiment object, install TILPRED from GitHub via remotes, and run the classifier (predictTilState) with parameters for human/mouse, score threshold, cell-cycle threshold and CD8 T filtering (filterCD8T removes non-CD8 cells, which are otherwise classified as Treg, CD4T or NKT from curated gene signatures). TILPRED computes signature enrichment (AUCell-based) and assigns each cell a probability score of belonging to Exhausted, MemoryLike (Tpex: Tox+/Pdcd1+/Tcf7+), EffectorMemory (Gzmk/Gzmb with Tcf7/Lef1/Il7r/Ly6c2, low-intermediate Pdcd1) or other reference states; Ly6C marks EM-like cells. Per-mouse annotation of exhausted, memory-like and EM-like states then feeds clonal expansion quantification (fraction of cells in clones seen in >1 cell) and pairwise Morisita-Horn overlap computed within each mouse. Note TILPRED is unmaintained; ProjecTILs is the recommended successor for reference-map projection of new scRNA-seq data.

## Related tools

- **TILPRED** (Logistic-regression classifier that assigns each CD8 T cell in a SingleCellExperiment object a probability score for the reference TIL states (predictTilState with nCores, human, scoreThreshold, cellCycleThreshold, filterCD8T)) — https://github.com/carmonalab/TILPRED
- **SingleCellExperiment** (Bioconductor container format holding the CD8 T cell expression matrix passed to TILPRED)
- **AUCell** (Bioconductor package used with SingleCellExperiment for gene-signature enrichment scoring underlying the state classification)
- **ProjecTILs** (Maintained successor framework that projects new scRNA-seq data into reference CD8 TIL atlases for state annotation when TILPRED is unsuitable) — https://github.com/carmonalab/ProjecTILs
- **tracer** (Reconstructs paired TCR alpha/beta sequences and clonotypes from the same single-cell data, so annotated states can be linked to clonotypes for Morisita-Horn overlap analysis) — https://github.com/Teichlab/tracer

## Examples

```
remotes::install_github("carmonalab/TILPRED"); data(B16CD8TIL_SCE); preds <- predictTilState(B16CD8TIL_SCE, nCores=8, human=FALSE, scoreThreshold=0, cellCycleThreshold=0.9, filterCD8T=TRUE)
```

## Evaluation signals

- Cells pass filterCD8T and non-CD8 contaminants are labelled Treg/CD4T/NKT rather than being force-assigned a CD8 TIL state
- Annotated states reproduce the four reference CD8 TIL states (exhausted, memory-like, naive, EM-like) with plausible marker expression: Tcf7 in memory-like, Tox/Pdcd1/Havcr2/Gzmb in exhausted, Ly6C/Ly6c2 in EM-like
- Downstream validation per task_003: pairwise Morisita-Horn indices are in (0,1) with highest overlap between exhausted and memory-like and near-zero overlap between EM-like and memory-like
- State assignments remain consistent when recomputed with a fixed random seed via doRNG/doParallel (deterministic multi-core runs)
- Score-threshold filtering (scoreThreshold) removes low-confidence cells; only cells above the threshold should carry state labels

## Limitations

- TILPRED is no longer maintained; the authors recommend ProjecTILs instead for reference-based analysis
- Reference map is limited to the B16 melanoma model and a single mouse strain (C57BL/6 plus PMEL transgenic); additional mouse strains and cancer types are needed to refine the CD8 TIL landscape
- Reference states rely on signature-enrichment thresholds (scoreThreshold, cellCycleThreshold), so poorly covered Smart-seq2 cells or high cell-cycle scores may receive low-confidence or filtered assignments

## Evidence

- [abstract] Unsupervised clustering and gene-signature analysis revealed four distinct CD8 TIL states - exhausted, memory-like, naïve and effector memory-like (EM-like): "Unsupervised clustering and gene-signature analysis revealed four distinct CD8 TIL states - exhausted, memory-like, naïve and effector memory-like (EM-like)"
