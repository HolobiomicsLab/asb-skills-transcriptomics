---
name: silhouette-based-cluster-number-selection
description: Use when performing unsupervised clustering of single-cell expression
  data and you need a quantitative, unbiased criterion (silhouette score) to pick
  K instead of fixing cluster resolution ad hoc.
when_to_use_negative:
- Input contains mixed lymphocyte populations (Treg, CD4T, NKT cells) — filter to
  CD8 T cells first (e.g. TILPRED's filterCD8T) or silhouette values will be dominated
  by coarse cell-type separation
- Input is bulk RNA-seq or flow cytometry data, where per-cell silhouette scoring
  on a single-cell neighborhood graph is not applicable
- A curated reference atlas projection (e.g. ProjecTILs) is preferred over de novo
  clustering, e.g. when mapping query cells onto a stable annotated reference
edam_operation: http://edamontology.org/operation_3611
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: TILPRED
  role: R logistic-regression classifier assigning each filtered CD8 T cell to one
    of the reference CD8 TIL states (exhausted, memory-like, effector memory, etc.)
    after clustering
  repo: https://github.com/carmonalab/TILPRED
- name: SingleCellExperiment
  role: Bioconductor container for the normalized CD8 TIL expression matrix passed
    to clustering and state prediction
- name: AUCell
  role: Bioconductor package used to score curated gene-signature enrichment (e.g.
    Tcf7/Lef1/Il7r for naïve, Pdcd1 for exhausted) for cluster annotation
- name: smart-seq2
  role: Plate-based scRNA-seq technology that generated the CD8 TIL transcriptomes
    being clustered
- name: ProjecTILs
  role: Recommended successor to TILPRED for projecting query scRNA-seq data into
    the CD8 TIL reference atlas
  repo: https://github.com/carmonalab/ProjecTILs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1080/2162402x.2020.1737369
    title: Deciphering the transcriptomic landscape of tumor-infiltrating CD8 lymphocytes
      in B16 melanoma tumors with single-cell RNA-Seq
  source_content_hash: sha256:51d621959e8a87924000393b5720daa12d5f5a900c8971d00781091624022b1d
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
  - smart-seq2
  edam_operation: http://edamontology.org/operation_3611
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# silhouette-based-cluster-number-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects the optimal number of transcriptomic cell states K by computing silhouette coefficients across candidate clustering resolutions and choosing the K before the sharp drop — here K=4 for CD8 TILs from B16 melanoma scRNA-seq (GSE116390, >3500 smart-seq2 cells from seven tumor-bearing mice).

## When to use

Use when performing unsupervised clustering of single-cell expression data and you need a quantitative, unbiased criterion (silhouette score) to pick K instead of fixing cluster resolution ad hoc.

## When NOT to use

- Input contains mixed lymphocyte populations (Treg, CD4T, NKT cells) — filter to CD8 T cells first (e.g. TILPRED's filterCD8T) or silhouette values will be dominated by coarse cell-type separation
- Input is bulk RNA-seq or flow cytometry data, where per-cell silhouette scoring on a single-cell neighborhood graph is not applicable
- A curated reference atlas projection (e.g. ProjecTILs) is preferred over de novo clustering, e.g. when mapping query cells onto a stable annotated reference

## Inputs

- Quality-controlled CD8 TIL single-cell RNA-seq count matrix (GSE116390, smart-seq2, >3500 cells from seven B16 tumor-bearing mice)
- SingleCellExperiment object (e.g. the B16CD8TIL_SCE example dataset)
- Top 10 principal components of the normalized expression matrix
- Curated marker gene signatures for cluster annotation (Tcf7/Lef1/Il7r, Pdcd1/Tox, Gzmk/Gzmb/Ly6c2)

## Outputs

- Silhouette coefficient vs. K curve justifying the selected K (K=4)
- Per-cell cluster/state assignment table (cd8_til_state_assignments.csv) with labels in {exhausted, memory-like, naive, EM-like}
- Annotated four-state CD8 TIL partition embedded in the Seurat SNN graph

## How to apply

After cell/gene QC filtering of the CD8 TIL count matrix, normalize the expression matrix, run PCA and retain the top 10 PCs. Build a shared-nearest-neighbor graph in Seurat and cluster across a sweep of candidate K values, computing the mean silhouette coefficient for each K. Choose the K at which silhouette is high before a large drop — in the B16 CD8 TIL data the silhouette dropped sharply after K=4, so K=4 was selected. Annotate the resulting four clusters using curated marker signatures: naïve (Tcf7/Lef1/Il7r high, no cytotoxicity/activation markers), exhausted (PD-1/Pdcd1 high, Tox, cytotoxic molecules, lacking Tcf7), memory-like (Tox+ PD-1+ Tcf7+ progenitor-exhausted), and EM-like (Gzmk/Gzmb with Tcf7/Lef1/Il7r/Ly6c2, low-to-intermediate PD-1). The classifier TILPRED (logistic regression over these reference states, superseded by ProjecTILs) can assign each cell a state probability; AUCell computes signature enrichment scores on the SingleCellExperiment object.

## Related tools

- **TILPRED** (R logistic-regression classifier assigning each filtered CD8 T cell to one of the reference CD8 TIL states (exhausted, memory-like, effector memory, etc.) after clustering) — https://github.com/carmonalab/TILPRED
- **SingleCellExperiment** (Bioconductor container for the normalized CD8 TIL expression matrix passed to clustering and state prediction)
- **AUCell** (Bioconductor package used to score curated gene-signature enrichment (e.g. Tcf7/Lef1/Il7r for naïve, Pdcd1 for exhausted) for cluster annotation)
- **smart-seq2** (Plate-based scRNA-seq technology that generated the CD8 TIL transcriptomes being clustered)
- **ProjecTILs** (Recommended successor to TILPRED for projecting query scRNA-seq data into the CD8 TIL reference atlas) — https://github.com/carmonalab/ProjecTILs

## Evaluation signals

- The silhouette-vs-K curve shows a sharp drop after K=4, and K=4 is the selected optimum
- Exactly four CD8 TIL states are recovered and each cluster is annotated to one of {exhausted, memory-like, naive, EM-like}
- cd8_til_state_assignments.csv contains a state label for every QC-filtered cell (no unassigned cells)
- Marker checks: naïve cluster is Tcf7/Lef1/Il7r-high with no cytotoxicity/activation markers; exhausted cluster expresses PD-1 (Pdcd1), Tox and cytotoxic molecules but lacks Tcf7; EM-like co-expresses Gzmk/Gzmb with memory genes and Ly6c2

## Limitations

- The K=4 optimum is derived from this specific B16 C57BL/6 dataset (plate-based smart-seq2, seven mice); the silhouette drop point is dataset-dependent and should be recomputed on new data
- TILPRED is no longer maintained; the authors recommend ProjecTILs instead
- Non-CD8 T cells must be removed beforehand — TILPRED classifies non-CD8 cells (Treg/Foxp3+, CD4T, NKT) by signature enrichment, and their inclusion distorts within-CD8 silhouette structure

## Evidence

- [readme] TILPRED input container and classifier behavior: [evidence span withheld — anchor status: not_found]
