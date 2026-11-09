---
name: cluster-signature-radar-visualization
description: Use when single-cell clusters (e.g., CD8 TIL states) need to be summarized
  as quantitative phenotypic axes from curated marker signatures, with a compact radar-chart
  comparison across clusters.
when_to_use_negative:
- Input is not single-cell expression data (e.g., already-aggregated bulk signature
  scores or a feature/importance table) — no per-cell ranking can be computed
- Genes are not in mouse symbol space or signatures contain genes absent from the
  matrix, since AUCell AUC depends on ranking the actual measured genes
- The goal is cell-state classification rather than signature-axis quantification
  — use TILPRED's predictTilState / classifier instead
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: TILPRED
  role: Provides scAUCscore, the function that wraps AUCell to compute per-cell signature
    AUC scores with aucMaxRank set to the top 1500 genes
  repo: https://github.com/carmonalab/TILPRED
- name: AUCell
  role: Underlying Bioconductor algorithm that ranks genes per cell by expression
    and computes AUC over the top-ranked window
- name: SingleCellExperiment
  role: Data container for the cell-by-gene expression matrix passed to scAUCscore
- name: ggradar
  role: Renders the scaled cluster-level mean signature scores (three axes per cluster)
    as a radar/spider chart
  repo: https://github.com/ricardo-bion/ggradar
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1080/2162402x.2020.1737369
    title: Deciphering the transcriptomic landscape of tumor-infiltrating CD8 lymphocytes
      in B16 melanoma tumors with single-cell RNA-Seq
  source_content_hash: sha256:7b46b3a9b5180ee55140efb7bd691677e03057b93b7e138427b9198a30af2275
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
  - R
  - SingleCellExperiment
  - TILPRED
  - ggradar
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# cluster-signature-radar-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify curated gene-signature axes (e.g., stemness, cytotoxicity, inhibition/exhaustion) per single cell using AUCell-based AUC scores, then aggregate per-cluster averages and render a radar profile with ggradar to summarize phenotypic differences among cell clusters.

## When to use

Use when single-cell clusters (e.g., CD8 TIL states) need to be summarized as quantitative phenotypic axes from curated marker signatures, with a compact radar-chart comparison across clusters.

## When NOT to use

- Input is not single-cell expression data (e.g., already-aggregated bulk signature scores or a feature/importance table) — no per-cell ranking can be computed
- Genes are not in mouse symbol space or signatures contain genes absent from the matrix, since AUCell AUC depends on ranking the actual measured genes
- The goal is cell-state classification rather than signature-axis quantification — use TILPRED's predictTilState / classifier instead

## Inputs

- SingleCellExperiment object with a cell-by-gene expression matrix using mouse gene symbols (e.g., B16CD8TIL_SCE)
- Curated gene signature sets: stemness (Tcf7, Lef1, Sell, Il7r), cytotoxicity (Prf1, Fasl, Gzmb), inhibition/exhaustion (Pdcd1, Havcr2, Tigit, Lag3, Ctla4)
- Per-cell cluster/state labels for aggregation

## Outputs

- aucell_scores.csv — per-cell AUC score matrix (one row per cell, one column per signature axis)
- radar_profile.png — ggradar radar chart of the three scaled signature axes per cluster

## How to apply

Load the cell-by-gene data as a SingleCellExperiment (e.g., the B16CD8TIL_SCE example object) and confirm mouse gene symbols. Define curated signature gene sets: naive/stemness (Tcf7, Lef1, Sell, Il7r), cytotoxicity (Prf1, Fasl, Gzmb), and inhibition/exhaustion (Pdcd1, Havcr2, Tigit, Lag3, Ctla4). Run scAUCscore from TILPRED, which wraps AUCell: genes are ranked per cell by expression and the AUC is computed over the top-ranked window with aucMaxRank set to the top 1500 expressed genes, so each AUC represents the fraction of signature genes within the top 1500 genes. Assemble a per-cell score matrix (one row per cell, one column per signature axis) and save it as aucell_scores.csv. Compute mean scores per cluster (removing cycling cells), scale cluster-level values 0–1, and plot with ggradar as radar_profile.png.

## Related tools

- **TILPRED** (Provides scAUCscore, the function that wraps AUCell to compute per-cell signature AUC scores with aucMaxRank set to the top 1500 genes) — https://github.com/carmonalab/TILPRED
- **AUCell** (Underlying Bioconductor algorithm that ranks genes per cell by expression and computes AUC over the top-ranked window)
- **SingleCellExperiment** (Data container for the cell-by-gene expression matrix passed to scAUCscore)
- **ggradar** (Renders the scaled cluster-level mean signature scores (three axes per cluster) as a radar/spider chart) — https://github.com/ricardo-bion/ggradar

## Examples

```
library(TILPRED); data(B16CD8TIL_SCE); scores <- scAUCscore(B16CD8TIL_SCE, geneSets=list(stemness=c("Tcf7","Lef1","Sell","Il7r"), cytotoxicity=c("Prf1","Fasl","Gzmb"), exhaustion=c("Pdcd1","Havcr2","Tigit","Lag3","Ctla4")), aucMaxRank=1500)
```

## Evaluation signals

- aucell_scores.csv has exactly one row per cell and three AUC columns, with all values in the valid AUCell range (AUC bounded by the fraction-of-signature-genes semantics over the top-1500 window)
- AUCs rank genes per cell with aucMaxRank set to the top 1500 expressed genes — check the parameter is actually passed to scAUCscore/AUCell
- Axes discriminate expected populations: naive cells are Tcf7/Lef1/Il7r-high, exhausted cells are Pdcd1/Havcr2-high, and EM-like cells are cytotoxicity-high with no PD-1 expression
- Radar chart axes are cluster-level averages computed after removing cycling cells and scaled to the 0–1 range before plotting with ggradar

## Limitations

- TILPRED is no longer maintained; the authors recommend using ProjecTILs instead, which may affect long-term reproducibility of the scAUCscore path
- Signatures are curated for mouse CD8 TILs in the B16 melanoma model; axes and marker choices may not transfer to other models or human data without adjustment
- Cluster-level radar profiles depend on prior state annotation; mislabeled clusters or unremoved cycling cells will distort the averaged profile

## Evidence

- [methods] stemness/cytotoxicity/exhaustion axes quantified by AUCell: [evidence span withheld — anchor status: not_found]
- [methods] AUC window definition: [evidence span withheld — anchor status: not_found]
- [readme] ggradar purpose: [evidence span withheld — anchor status: external_doc]
