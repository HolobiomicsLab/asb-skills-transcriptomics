---
name: single-cell-cellstate-label-transfer
description: Use when raw scRNA-seq counts of T cells (or other cell types with a
  pre-built reference map) must be annotated with reference-defined cell states, e.g.
  assigning tumor-specific CD8 TILs to Tex/Tpex/EM states on the murine TIL atlas.
when_to_use_negative:
- The query contains γδ T cells or CD4+ TIL states that are under-represented or not
  represented at all in the reference atlases
- The goal is de novo unsupervised discovery of entirely novel cell types for which
  no appropriate reference map exists rather than assignment to reference-defined
  states
- A different normalization strategy is preferred but raw counts cannot be placed
  in the Seurat 'data' layer with skip.normalize=TRUE
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ProjecTILs
  role: Core R package performing reference projection (Run.ProjecTILs / ProjecTILs.classifier)
    and nearest-neighbor cell state label transfer (cellstate.predict)
  repo: https://github.com/carmonalab/ProjecTILs
- name: Seurat
  role: Data structure (Seurat object) for the query and reference; ProjecTILs is
    integrated with Seurat so projected outputs feed downstream analyses
- name: STACAS
  role: Underlying scRNA-seq integration method used to build the multi-study reference
    atlases onto which queries are projected
  repo: https://github.com/carmonalab/STACAS
- name: ProjecTILs.demo
  role: Demos and tutorials (mouse TIL and human CD8 TIL) for running the projection
    workflow
  repo: https://github.com/carmonalab/ProjecTILs.demo
- name: TILPRED
  role: Superseded logistic regression classifier for CD8 TIL states (Tex vs Tpex/MemoryLike);
    no longer maintained in favor of ProjecTILs
  repo: https://github.com/carmonalab/TILPRED
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-23324-4
    title: Interpretation of T cell states from single-cell transcriptomics data using
      reference atlases
  source_content_hash: sha256:d74a0a184e17d02823b719b0671a7fba1869a48cc6d3d35269c606bb5244468e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/ProjecTILs
  tools:
  - ProjecTILs
  - ProjecTILs.demo
  - R
  - STACAS
  - Seurat
  - TILPRED
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# single-cell-cellstate-label-transfer

## Summary

Reference-based cell state label transfer for scRNA-seq: ProjecTILs projects query cells (e.g. tetramer+ CD8 TILs) into a stable annotated reference atlas PCA/UMAP space and transfers subtype labels (e.g. Tex, Tpex, EM) via nearest-neighbor prediction without altering the reference structure.

## When to use

Use when raw scRNA-seq counts of T cells (or other cell types with a pre-built reference map) must be annotated with reference-defined cell states, e.g. assigning tumor-specific CD8 TILs to Tex/Tpex/EM states on the murine TIL atlas.

## When NOT to use

- The query contains γδ T cells or CD4+ TIL states that are under-represented or not represented at all in the reference atlases
- The goal is de novo unsupervised discovery of entirely novel cell types for which no appropriate reference map exists rather than assignment to reference-defined states
- A different normalization strategy is preferred but raw counts cannot be placed in the Seurat 'data' layer with skip.normalize=TRUE

## Inputs

- Query single-cell RNA-seq raw counts expression matrix loaded into a Seurat object (e.g. Miller et al. tetramer+ CD8+ TIL data)
- ProjecTILs reference map downloaded/loaded via load.reference.map() (e.g. murine TIL atlas, 16,803 cells from B16 and MC38 tumors)
- Optional gene marker sets (e.g. cell.cycle.obj) for post-projection scoring

## Outputs

- Seurat object with query cells embedded in reference PCA/UMAP coordinates and predicted cell state labels (e.g. Tex, Tpex, EM)
- Barplot of query cell state frequencies from plot.statepred.composition (exemplar: ~90.2% Tex, 4.4% Tpex, 3.9% EM)
- Discriminant genes from find.discriminant.genes and radar plots of marker expression (e.g. Gzmb, Mki67) via plot.states.radar
- Cycling-score visualization for query cells using cell.cycle.obj marker sets

## How to apply

Load the default murine TIL reference atlas with load.reference.map() (or pick another map via list.reference.maps()/get.reference.maps(), e.g. human CD8 TIL or LCMV viral CD8 atlases). Read the query expression matrix with read.sc.query into a Seurat object containing raw counts (ProjecTILs log-normalizes internally; use skip.normalize=TRUE if data are already normalized in the 'data' layer). Run Run.ProjecTILs(query, ref), which embeds query cells in the reference PCA/UMAP space and assigns cell state labels via nearest-neighbor prediction (cellstate.predict); for label transfer only, use the faster ProjecTILs.classifier mode that leaves query embeddings intact. Quantify subtype frequencies with plot.statepred.composition and, to verify the projection, run find.discriminant.genes against the reference and visualize markers (e.g. Gzmb, Mki67) with plot.states.radar; score cycling cells with the cell.cycle.obj marker sets. In the exemplar case, the Miller et al. B16-OVA tetramer+ CD8 TIL query was assigned ~90.2% Tex, 4.4% Tpex, and 3.9% EM.

## Related tools

- **ProjecTILs** (Core R package performing reference projection (Run.ProjecTILs / ProjecTILs.classifier) and nearest-neighbor cell state label transfer (cellstate.predict)) — https://github.com/carmonalab/ProjecTILs
- **Seurat** (Data structure (Seurat object) for the query and reference; ProjecTILs is integrated with Seurat so projected outputs feed downstream analyses)
- **STACAS** (Underlying scRNA-seq integration method used to build the multi-study reference atlases onto which queries are projected) — https://github.com/carmonalab/STACAS
- **ProjecTILs.demo** (Demos and tutorials (mouse TIL and human CD8 TIL) for running the projection workflow) — https://github.com/carmonalab/ProjecTILs.demo
- **TILPRED** (Superseded logistic regression classifier for CD8 TIL states (Tex vs Tpex/MemoryLike); no longer maintained in favor of ProjecTILs) — https://github.com/carmonalab/TILPRED

## Examples

```
ref <- load.reference.map(); query.projected <- Run.ProjecTILs(query_example_seurat, ref=ref)
```

## Evaluation signals

- Predicted subtype composition matches the reported distribution within a few percent (exemplar B16-OVA tetramer+ CD8 TILs: ~90.2% Tex, 4.4% Tpex, 3.9% EM)
- Run.ProjecTILs returns query cells embedded in the reference PCA/UMAP space with non-empty cell state labels for all (or nearly all) query cells
- plot.statepred.composition produces a non-empty barplot of cell state frequencies
- find.discriminant.genes yields a non-empty gene list and plot.states.radar renders marker expression (e.g. Gzmb, Mki67) across predicted states
- Silhouette coefficients of query cells (compute_silhouette) indicate query cells sit near reference subtypes rather than as isolated outliers

## Limitations

- UMAP embeddings are for visual exploration only — cell-cell distances and densities in UMAP are highly distorted; ProjecTILs performs cell state quantifications in the PCA latent space
- Reference atlases do not encompass the full diversity of T cell transcriptional states: CD4+ TILs were under-represented compared to CD8+, and γδ T cells were not represented at all
- A high-quality human T cell reference atlas does not yet exist (human data can be projected onto the mouse atlas via ortholog mapping, e.g. Hs2Mm.convert.table)
- Query objects are expected to contain raw counts; already-normalized data must be stored in the 'data' layer and projected with skip.normalize=TRUE

## Evidence

- [abstract] ProjecTILs allows not only accurate embedding of new scRNA-seq data into a reference without altering its structure: "ProjecTILs allows not only accurate embedding of new scRNA-seq data into a reference without altering its structure"
- [methods] It returns the query embedded in the PCA and UMAP space of the reference, and predict cell state labels: "It returns the query embedded in the PCA and UMAP space of the reference, and predict cell state labels for the query cells"
- [methods] Use a nearest-neighbor algorithm to predict a feature (e.g. the cell state) of the query cells: "Use a nearest-neighbor algorithm to predict a feature (e.g. the cell state) of the query cells"
- [discussion] Because ProjecTILs is integrated with Seurat, it can be easily combined with other tools: "Because ProjecTILs is integrated with Seurat, it can be easily combined with other tools for up- and down-stream analyses."
- [readme] ProjecTILs performs cell state quantifications in the PCA latent space: [evidence span withheld — anchor status: external_doc]
