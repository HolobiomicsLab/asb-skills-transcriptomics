---
name: ligand-receptor-pair-enrichment
description: Use when you have an scRNA-seq count matrix plus cell-type annotations
  and need to infer, rank, or visualize ligand-receptor-mediated cell-cell communication
  between annotated cell populations.
when_to_use_negative:
- Input is not single-cell transcriptomic data with cell-type labels (e.g. bulk RNA-seq
  or a precomputed communication matrix)
- Species lacks CellTalkDB coverage or no suitable lrpairs data.frame is available
  for the organism
- The goal is pathway-level transcriptome enrichment (e.g. GSEA of DEGs) rather than
  pairwise ligand-receptor communication inference
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scCrossTalk
  role: Core R package implementing Z-test-based enrichment of highly expressed LR
    pairs and CCI visualization
  repo: https://github.com/multitalk/scCrossTalk
- name: R
  role: Runtime environment; scCrossTalk is installed via devtools (install_github
    or install_local)
- name: Seurat
  role: Compatible single-cell data source for the count matrix and cell-type annotations
    (scCrossTalk is compatible with Seurat 5)
- name: ggplot2
  role: Imported dependency backing scCrossTalk's LR-pair and CCI plotting functions
- name: circlize
  role: Imported dependency for chord-diagram CCI/LR-pair plots
- name: heatmaply
  role: Imported dependency for interactive CCI and LR-pair heatmaps
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.eng.2023.12.004
    title: A Single-Cell Landscape of Human Liver Transplantation Reveals a Pathogenic
      Immune Niche Associated with Early Allograft Dysfunction
  source_content_hash: sha256:f1f64cb15b10b808f8ffc64c4ab3c4b6aaea877aaf2ab003dc78a64c3bb1fedc
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/multitalk/scCrossTalk
  repo_url: https://github.com/multitalk/scCrossTalk
  tools:
  - R
  - Seurat
  - circlize
  - ggplot2
  - heatmaply
  - scCrossTalk
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# ligand-receptor-pair-enrichment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Enrich highly expressed ligand-receptor pairs from single-cell RNA-seq data using scCrossTalk, which tests CellTalkDB-derived LR pairs with the Z-test statistical method to infer and rank cell-cell communications (e.g., hepatocyte SAA1-FPR1 signaling to S100A12+ neutrophils in liver transplantation).

## When to use

Use when you have an scRNA-seq count matrix plus cell-type annotations and need to infer, rank, or visualize ligand-receptor-mediated cell-cell communication between annotated cell populations.

## When NOT to use

- Input is not single-cell transcriptomic data with cell-type labels (e.g. bulk RNA-seq or a precomputed communication matrix)
- Species lacks CellTalkDB coverage or no suitable lrpairs data.frame is available for the organism
- The goal is pathway-level transcriptome enrichment (e.g. GSEA of DEGs) rather than pairwise ligand-receptor communication inference

## Inputs

- scRNA-seq count matrix (e.g. the demo_sc_data dataset)
- Cell-type annotations for each cell (sc_celltype character vector)
- CellTalkDB-derived ligand-receptor pair data.frame (lrpairs, e.g. demo_lrpairs)
- Gene info table for reverse-gene annotation (geneinfo, e.g. demo_geneinfo)

## Outputs

- scCrossTalk object with enriched LR interactions (679 found in the demo run)
- Ranked top-N ligand-receptor pair table (e.g. top 10, exportable as CSV)
- Cell-cell communication plots: circle, heatmap, chord, sankey, bubble

## How to apply

Construct an scCrossTalk object with create_scCrossTalk(sc_data, sc_celltype, species='human') from the count matrix and cell-type vector. Enrich highly expressed ligand-receptor pairs against the CellTalkDB-based system lrpairs data.frame using find_lrpairs(object, lrpairs=lrpairs), which applies a Z-test together with expressed-cell percentage (cell_min_pct), minimum cell number (min_cell_num), and p-value (p_value) filters. Extract the leading interactions with get_top_lrpairs(object, top_n=10) and optionally annotate downstream signaling with rev_gene(data, data_type, species, geneinfo) using the geneinfo data. Visualize the communications with plot_cci_circle/plot_cci_heatmap (number and score of LR pairs) and sender/receiver-specific plots such as plot_cci_chord or plot_lrpairs_heatmap with celltype_sender/celltype_receiver arguments. The documented demo run recovers 679 LR interactions after find_lrpairs, and the hepatocyte SAA1–FPR1 axis (SAA1 in hepatocytes, FPR1 in S100A12+ neutrophils) must be recoverable in ranked hepatocyte-to-neutrophil communications.

## Related tools

- **scCrossTalk** (Core R package implementing Z-test-based enrichment of highly expressed LR pairs and CCI visualization) — https://github.com/multitalk/scCrossTalk
- **R** (Runtime environment; scCrossTalk is installed via devtools (install_github or install_local))
- **Seurat** (Compatible single-cell data source for the count matrix and cell-type annotations (scCrossTalk is compatible with Seurat 5))
- **ggplot2** (Imported dependency backing scCrossTalk's LR-pair and CCI plotting functions)
- **circlize** (Imported dependency for chord-diagram CCI/LR-pair plots)
- **heatmaply** (Imported dependency for interactive CCI and LR-pair heatmaps)

## Examples

```
obj <- create_scCrossTalk(sc_data, sc_celltype, species='human'); obj <- find_lrpairs(obj, lrpairs=lrpairs); top10 <- get_top_lrpairs(obj, top_n=10)
```

## Evaluation signals

- The scCrossTalk object summary after find_lrpairs reports a non-zero interaction count; the documented demo run reports 679 ligand-receptor interactions found
- The SAA1-FPR1 hepatocyte-to-S100A12+ neutrophil pair is recoverable in the ranked top-N LR results
- find_lrpairs completes its progress bar ('Finished:100%') without errors and returns an object of class scCrossTalk
- Top-10 LR table from get_top_lrpairs(object, top_n=10) exports cleanly to CSV with ligand, receptor, sender and receiver cell types
- Visualization functions (plot_cci_circle, plot_cci_heatmap, plot_cci_chord) render non-empty plots reflecting the number and score of LR pairs

## Limitations

- Inference depends on the completeness of CellTalkDB v1.0; interactions absent from the database cannot be recovered
- Expression filters (cell_min_pct, min_cell_num, p_value) determine which LR pairs are retained, so weakly expressed but biologically relevant pairs may be filtered out
- Findings are organism-specific (human transplanted liver); the SAA1-FPR1 and HGF-MET signals are context-dependent on the EAD versus non-EAD comparison and may not generalize

## Evidence

- [intro] Z-test enrichment basis: [evidence span withheld — anchor status: not_found]
