---
name: single-cell-reference-atlas-projection
description: Use when raw single-cell RNA-seq count matrices (10x format or Seurat
  objects) must be interpreted in the stable coordinate system of a curated reference
  atlas, or when cell subtype composition of a query condition must be quantified
  and compared.
when_to_use_negative:
- The query population is not represented in the reference atlas — e.g., gamma-delta
  T cells are not represented at all, and CD4+ TILs are under-represented in the reference
  atlases
- The goal is de novo clustering of a heterogeneous dataset rather than interpreting
  query cells in a fixed reference coordinate system
- Only a label per cell is needed and re-embedding in the reference space is unnecessary
  — use the ProjecTILs.classifier mode instead of full projection, or a label-transfer-only
  tool
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: ProjecTILs
  role: Core framework for projecting query scRNA-seq data onto the reference atlas
    (Run.ProjecTILs / make.projection) and predicting cell state labels (cellstate.predict)
  repo: https://github.com/carmonalab/ProjecTILs
- name: Seurat
  role: Data structure and PCA/UMAP embedding backbone; ProjecTILs is integrated with
    Seurat for up- and downstream analyses
- name: STACAS
  role: Semi-supervised integration method used to build the multi-study reference
    atlases that queries are projected onto
  repo: https://github.com/carmonalab/STACAS
- name: ProjecTILs.demo
  role: Tutorial vignettes and demo workflows (HumanCD8TILs.demo, mouse TILs tutorial,
    custom reference atlas construction)
  repo: https://github.com/carmonalab/ProjecTILs.demo
- name: TILPRED
  role: Legacy logistic regression CD8 TIL state classifier, superseded by ProjecTILs
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_005
  - task_007
  - task_008
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-23324-4
    title: Interpretation of T cell states from single-cell transcriptomics data using
      reference atlases
  source_content_hash: sha256:39962be430bf5412ed2cc699ab9e72958bb102101c713a28497562b117b0f339
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/ProjecTILs
  tools:
  - Azimuth
  - ProjecTILs
  - ProjecTILs.demo
  - R
  - STACAS
  - Seurat
  - TILPRED
  - ggplot2
  - scmap
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# single-cell-reference-atlas-projection

## Summary

Projects new scRNA-seq query data into the PCA/UMAP space of a curated reference single-cell atlas (e.g., the murine TIL or viral CD8 T cell atlases) with ProjecTILs, preserving the reference coordinate system, and transfers cell state labels via nearest-neighbor prediction.

## When to use

Use when raw single-cell RNA-seq count matrices (10x format or Seurat objects) must be interpreted in the stable coordinate system of a curated reference atlas, or when cell subtype composition of a query condition must be quantified and compared.

## When NOT to use

- The query population is not represented in the reference atlas — e.g., gamma-delta T cells are not represented at all, and CD4+ TILs are under-represented in the reference atlases
- The goal is de novo clustering of a heterogeneous dataset rather than interpreting query cells in a fixed reference coordinate system
- Only a label per cell is needed and re-embedding in the reference space is unnecessary — use the ProjecTILs.classifier mode instead of full projection, or a label-transfer-only tool

## Inputs

- Query scRNA-seq raw count matrix (10x format) or Seurat object, e.g., tumor-infiltrating lymphocyte (TIL) or virus-specific CD8 T cell profiles
- Reference atlas map loaded via load.reference.map (mouse TIL atlas: 16,803 cells from 25 B16/MC38 samples; mouse viral CD8 T cell atlas: 7,000 cells from 12 LCMV samples)
- Human-to-mouse ortholog conversion table (Hs2Mm.convert.table) for cross-species projection
- Gene sets and cell.cycle.obj marker sets for radar/cycling scoring

## Outputs

- Projected Seurat object with query cells embedded in reference PCA/UMAP space and predicted functional.subset labels (e.g., Tex, Tpex, Tn, EM)
- Cell state frequency composition barplot per condition (plot.statepred.composition output)
- Per-subtype differential expression / discriminant gene tables (find.discriminant.genes)
- Ranked discriminant PCA/ICA dimension table with KS statistics (find.discriminant.dimensions)
- Radar plots of gene expression across predicted states and averaged expression heatmaps (celltype.heatmap)

## How to apply

Load the appropriate reference map with load.reference.map (default is the mouse TIL atlas; a viral LCMV CD8/CD4 atlas or human CD8/CD4 TIL atlases are also available), then load the query counts with read.sc.query (10x format, standard min.cells/min.features filtering; raw counts are log-normalized internally, or use skip.normalize=TRUE for pre-normalized data). Run Run.ProjecTILs to embed the query in the reference PCA/UMAP space and predict cell state labels via nearest-neighbor voting (cellstate.predict with default k, nn.decay, min.confidence); for label transfer only, use the faster ProjecTILs.classifier mode which leaves query embeddings intact. For human queries onto the murine atlas, convert genes with Hs2Mm.convert.table and pass the ortholog_table parameter to make.projection. Quantify subtype frequencies per condition with plot.statepred.composition and compare gene programs between conditions with find.discriminant.genes, find.discriminant.dimensions (PCA or ICA reduction), and plot.states.radar. The rationale is that, unlike Seurat-based integration or scmap, ProjecTILs preserves the integrity of the reference atlas space upon mapping, enabling direct cross-study, cross-condition, and even cross-species comparison.

## Related tools

- **ProjecTILs** (Core framework for projecting query scRNA-seq data onto the reference atlas (Run.ProjecTILs / make.projection) and predicting cell state labels (cellstate.predict)) — https://github.com/carmonalab/ProjecTILs
- **Seurat** (Data structure and PCA/UMAP embedding backbone; ProjecTILs is integrated with Seurat for up- and downstream analyses)
- **STACAS** (Semi-supervised integration method used to build the multi-study reference atlases that queries are projected onto) — https://github.com/carmonalab/STACAS
- **ProjecTILs.demo** (Tutorial vignettes and demo workflows (HumanCD8TILs.demo, mouse TILs tutorial, custom reference atlas construction)) — https://github.com/carmonalab/ProjecTILs.demo
- **TILPRED** (Legacy logistic regression CD8 TIL state classifier, superseded by ProjecTILs)

## Examples

```
library(ProjecTILs); ref <- load.reference.map(); query.projected <- Run.ProjecTILs(query_example_seurat, ref=ref)
```

## Evaluation signals

- In leave-half-samples-out cross-validation on the TIL reference atlas, projection accuracy exceeds 90% within one UMAP unit (93.4% within five PCA units) and classification accuracy reaches ~91.6%, outperforming Azimuth/Seurat 4 and scmap
- A known benchmark query (Miller et al. tetramer+ CD8 TIL) reproduces the expected composition: ~90.2% Tex, 4.4% Tpex, 3.9% EM assignments
- Projected query cells land within the reference embedding without altering its structure; compute_silhouette or recalculate.embeddings can verify embedding quality
- Compositional shifts match expected biology in perturbation case studies (e.g., miR-155 KO: naive-like shift with ~16% vs 8% KO vs WT T-cell fraction; Regnase-1 KO: ~3-fold Tpex and ~1.8-fold Tex enrichment; anti-CD4 LCMV: SLEC-to-Tpex shift with Tex unchanged)
- Human biopsies projected via ortholog mapping co-cluster by TIL reference subtype across species rather than by study or cancer type (human-mouse co-clustering p < 3e-6)

## Limitations

- Reference atlases do not yet encompass the full diversity of T cell transcriptional states, particularly CD4+ TILs (under-represented vs CD8+) and gamma-delta T cells (not represented at all)
- No high-quality human T cell reference atlas yet exists; human queries must be mapped through mouse orthologs onto the murine atlas
- Cell state quantifications are performed in the PCA latent space; UMAP embeddings are for visual exploration only since cell-cell distances and densities in UMAP are highly distorted (Chari and Pachter 2023)
- Poorly matched queries may represent previously unknown cell states that 'deviate' from the reference; projection alone will mislabel such cells rather than flag them unless deviation-detection tools (find.discriminant.dimensions, find.discriminant.genes) are used

## Evidence

- [abstract] ProjecTILs allows not only accurate embedding of new scRNA-seq data into a reference without altering its structure: "ProjecTILs allows not only accurate embedding of new scRNA-seq data into a reference without altering its structure, but also characterizing previously unknown cell states that "deviate" from"
- [methods] Run.ProjecTILs returns query in reference space: "It returns the query embedded in the PCA and UMAP space of the reference, and predict cell state labels for the query cells"
- [readme] README projection example: [evidence span withheld — anchor status: external_doc]
- [discussion] Cross-species projection via ortholog mapping: "by meta-analysis of 132 cancer patient biopsies, we have shown that ProjecTILs can accurately project human T cell transcriptomes onto a reference mouse atlas"
- [readme] UMAP interpretation caveat: [evidence span withheld — anchor status: external_doc]
