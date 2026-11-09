---
name: gene-set-enrichment-auc-scoring
description: Use when single-cell RNA-seq data has been clustered into CD8 TIL states
  and you need quantitative per-cell scores for phenotype axes defined by marker gene
  sets (e.g. Pdcd1/Havcr2 exhaustion, Gzmb cytotoxicity, Tcf7 stemness).
when_to_use_negative:
- The goal is discrete state classification with probability scores rather than continuous
  signature-axis enrichment — use TILPRED's predictTilState classifier instead
- Gene identifiers are not consistent mouse symbols (e.g. Ensembl IDs or human symbols
  without conversion), since AUCell ranking depends on exact signature-gene matching
- You need reference-based projection of new data onto an annotated atlas rather than
  within-dataset signature scoring — the TILPRED README directs users to ProjecTILs
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: TILPRED
  role: Provides scAUCscore, which wraps AUCell to compute signature AUCs on the SingleCellExperiment;
    ships the B16CD8TIL_SCE example object
  repo: github.com/carmonalab/TILPRED
- name: AUCell
  role: Bioconductor algorithm that ranks genes per cell and computes AUC over the
    top-ranked window (top 1500 genes)
  repo: https://bioconductor.org/packages/release/bioc/html/AUCell.html
- name: SingleCellExperiment
  role: Bioconductor container format holding the expression data and per-cell metadata
    passed to scAUCscore
- name: ggradar
  role: Renders the radar chart of the three signature axes per CD8 TIL cluster after
    0–1 scaling
  repo: github.com/carmonalab/ggradar
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1080/2162402x.2020.1737369
    title: Deciphering the transcriptomic landscape of tumor-infiltrating CD8 lymphocytes
      in B16 melanoma tumors with single-cell RNA-Seq
  source_content_hash: sha256:bd20b07f299f3ac84c66e161b149ce55c5af98397dde5100e2c3151060e9d77e
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
  - R
  - SingleCellExperiment
  - TILPRED
  - ggradar
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# gene-set-enrichment-auc-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies per-cell enrichment of curated gene signatures (stemness, cytotoxicity, inhibition/exhaustion) in CD8 TILs by computing AUCell AUC scores over each cell's ranked expression, then summarizing cluster-level profiles on a radar chart.

## When to use

Use when single-cell RNA-seq data has been clustered into CD8 TIL states and you need quantitative per-cell scores for phenotype axes defined by marker gene sets (e.g. Pdcd1/Havcr2 exhaustion, Gzmb cytotoxicity, Tcf7 stemness).

## When NOT to use

- The goal is discrete state classification with probability scores rather than continuous signature-axis enrichment — use TILPRED's predictTilState classifier instead
- Gene identifiers are not consistent mouse symbols (e.g. Ensembl IDs or human symbols without conversion), since AUCell ranking depends on exact signature-gene matching
- You need reference-based projection of new data onto an annotated atlas rather than within-dataset signature scoring — the TILPRED README directs users to ProjecTILs

## Inputs

- SingleCellExperiment object of CD8 TIL scRNA-seq profiles with mouse gene symbols (e.g. B16CD8TIL_SCE)
- Curated signature gene sets: naive/stemness (Tcf7, Lef1, Sell, Il7r), cytotoxic effector (Prf1, Fasl, Gzmb), exhausted (Pdcd1, Havcr2, Tigit, Lag3, Ctla4)
- Cluster labels per cell and cycling-cell annotations

## Outputs

- aucell_scores.csv: one row per cell, three AUC columns (stemness, cytotoxicity, inhibition/exhaustion) within the valid AUCell range
- radar_profile.png: ggradar radar chart of cluster-level mean signature scores scaled 0–1

## How to apply

Load the SingleCellExperiment object (e.g. the B16CD8TIL_SCE example shipped with TILPRED) and confirm mouse gene symbols are used. Define curated signature sets: naive/stemness (Tcf7, Lef1, Sell, Il7r), cytotoxic effector (Prf1, Fasl, Gzmb), and exhausted/dysfunctional (Pdcd1, Havcr2, Tigit, Lag3, Ctla4). Run scAUCscore from TILPRED, which wraps AUCell: genes are ranked per cell by expression and the AUC is computed over the top-ranked window with aucMaxRank set to the top 1500 expressed genes, so each AUC equals the fraction of signature genes within that window. Assemble the per-cell score matrix (one row per cell, one column per signature axis) and save it as aucell_scores.csv. Aggregate mean scores per cluster — removing cycling cells first — scale 0–1 and plot a radar profile of the three axes per cluster with ggradar (radar_profile.png). This ranks signatures relative to each cell's own expression distribution, making scores comparable across cells with different total counts (relevant for smart-seq2 plate-based data).

## Related tools

- **TILPRED** (Provides scAUCscore, which wraps AUCell to compute signature AUCs on the SingleCellExperiment; ships the B16CD8TIL_SCE example object) — github.com/carmonalab/TILPRED
- **AUCell** (Bioconductor algorithm that ranks genes per cell and computes AUC over the top-ranked window (top 1500 genes)) — https://bioconductor.org/packages/release/bioc/html/AUCell.html
- **SingleCellExperiment** (Bioconductor container format holding the expression data and per-cell metadata passed to scAUCscore)
- **ggradar** (Renders the radar chart of the three signature axes per CD8 TIL cluster after 0–1 scaling) — github.com/carmonalab/ggradar

## Examples

```
R: data(B16CD8TIL_SCE); aucs <- scAUCscore(B16CD8TIL_SCE, geneSets=list(stemness=c('Tcf7','Lef1','Sell','Il7r'), cytotox=c('Prf1','Fasl','Gzmb'), exhaust=c('Pdcd1','Havcr2','Tigit','Lag3','Ctla4')))
```

## Evaluation signals

- aucell_scores.csv has exactly one row per cell and three AUC columns with values within the valid AUCell range
- Naive cells are Tcf7/Lef1/Il7r-high on the stemness axis
- Exhausted cells are Pdcd1/Havcr2-high on the inhibition/exhaustion axis
- EM-like cells score high for cytotoxicity (Gzmb/Prf1/Fasl) with no PD-1 expression
- Cluster-level radar profile reproduces the expected four-state separation (exhausted, memory-like, naive, EM-like) after removing cycling cells

## Limitations

- TILPRED is no longer maintained; the README recommends ProjecTILs instead
- Scores depend on the aucMaxRank window (top 1500 genes); the reference map was built on B16 melanoma in a single mouse strain, and future studies of additional strains and cancer types are needed
- Cycling cells must be removed before cluster-level averaging or radar profiles will be confounded by cell-cycle signatures

## Evidence

- [methods] AUCell AUC definition: [evidence span withheld — anchor status: not_found]
- [methods] cluster-level radar: [evidence span withheld — anchor status: not_found]
