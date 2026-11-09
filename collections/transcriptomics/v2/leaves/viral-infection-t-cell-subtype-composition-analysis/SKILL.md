---
name: viral-infection-t-cell-subtype-composition-analysis
description: Use when scRNA-seq data of virus-specific T cells (e.g. P14 cells from
  LCMV chronic infection under perturbations like CD4 T cell depletion) must be assigned
  to reference subtypes (SLEC, Tpex, Tex) and their composition compared between conditions.
when_to_use_negative:
- The query contains cell types absent from the reference (e.g. γδ T cells or CD4+
  TILs, which are not represented in the viral CD8 T cell atlas)
- The goal is de novo clustering/annotation of query data without a suitable reference
  atlas — unsupervised annotation would be required instead
- UMAP-based quantitative claims (cell-cell distances/densities) are intended, since
  ProjecTILs performs cell state quantifications in PCA space and UMAP distances are
  highly distorted
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: ProjecTILs
  role: Reference projection and cell state label transfer (Run.ProjecTILs, make.projection,
    cellstate.predict) plus composition and radar plotting
  repo: https://github.com/carmonalab/ProjecTILs
- name: Seurat
  role: Underlying single-cell object framework; ProjecTILs is integrated with Seurat
    for upstream/downstream analyses
  repo: https://github.com/carmonalab/ProjecTILs
- name: R
  role: Language in which the projection workflow is executed as an R package
  repo: https://github.com/carmonalab/ProjecTILs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-23324-4
    title: Interpretation of T cell states from single-cell transcriptomics data using
      reference atlases
  source_content_hash: sha256:0403188327d9c7ac52a2fe38402e01c808691c01238c8340c7f58c8a171e58f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/ProjecTILs
  tools:
  - ProjecTILs
  - R
  - Seurat
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# viral-infection-t-cell-subtype-composition-analysis

## Summary

Quantify the subtype composition of virus-specific CD8+ T cells (e.g. LCMV P14 query data) by projecting scRNA-seq query expression matrices onto the mouse acute/chronic viral infection CD8 T cell reference atlas with ProjecTILs, then comparing subtype frequencies and gene expression (e.g. Pdcd1) across conditions.

## When to use

Use when scRNA-seq data of virus-specific T cells (e.g. P14 cells from LCMV chronic infection under perturbations like CD4 T cell depletion) must be assigned to reference subtypes (SLEC, Tpex, Tex) and their composition compared between conditions.

## When NOT to use

- The query contains cell types absent from the reference (e.g. γδ T cells or CD4+ TILs, which are not represented in the viral CD8 T cell atlas)
- The goal is de novo clustering/annotation of query data without a suitable reference atlas — unsupervised annotation would be required instead
- UMAP-based quantitative claims (cell-cell distances/densities) are intended, since ProjecTILs performs cell state quantifications in PCA space and UMAP distances are highly distorted

## Inputs

- Query scRNA-seq raw count expression matrix or Seurat object of virus-specific T cells (e.g. P14 cells from anti-CD4-treated and isotype-control LCMV cl13 samples)
- ProjecTILs mouse acute/chronic viral infection CD8 T cell reference atlas (load.reference.map, figshare DOI 10.6084/m9.12489518)

## Outputs

- Projected Seurat objects with query cells embedded in reference PCA/UMAP space and predicted cell state labels per condition
- Subtype composition barplot per condition (e.g. kanev_statepred_composition.png) showing SLEC/Tpex/Tex frequencies
- Radar plots of gene expression (e.g. Pdcd1) per subtype and condition from plot.states.radar

## How to apply

Load the viral infection CD8 T cell reference map with load.reference.map (the LCMV Arm/cl13 atlas of ~7,000 virus-specific CD8 T cells from 12 spleen samples at day 4.5, day 7/8 and day 30). Load the query expression matrices (e.g. Kanev et al. anti-CD4-treated and isotype-control P14 conditions) with read.sc.query; ProjecTILs expects raw counts and log-normalizes internally (use skip.normalize=TRUE if data are pre-normalized). Project each condition with Run.ProjecTILs, which wraps make.projection and cellstate.predict (nearest-neighbor label transfer) and embeds the query in the reference PCA and UMAP space. Quantify cell subtype frequencies per condition with plot.statepred.composition to generate composition barplots, and compare gene expression (e.g. Pdcd1) in specific subtypes such as Tex with plot.states.radar. Interpret condition-level shifts in subtype proportions as perturbation effects on T cell differentiation, since projection preserves the reference space and only transfers cell state labels.

## Related tools

- **ProjecTILs** (Reference projection and cell state label transfer (Run.ProjecTILs, make.projection, cellstate.predict) plus composition and radar plotting) — https://github.com/carmonalab/ProjecTILs
- **Seurat** (Underlying single-cell object framework; ProjecTILs is integrated with Seurat for upstream/downstream analyses) — https://github.com/carmonalab/ProjecTILs
- **R** (Language in which the projection workflow is executed as an R package) — https://github.com/carmonalab/ProjecTILs

## Examples

```
ref <- load.reference.map("viral-CD8-T"); query.projected <- Run.ProjecTILs(read.sc.query("p14_counts.txt"), ref=ref); plot.statepred.composition(query.projected)
```

## Evaluation signals

- Composition barplot reproduces the expected condition contrast: SLEC decrease with reciprocal Tpex increase in anti-CD4 vs control, with Tex proportion unchanged
- Query cells are embedded in the reference PCA/UMAP space without altering the reference atlas structure
- Pdcd1 expression in the Tex subtype is elevated in the anti-CD4 condition in the plot.states.radar output
- Every query cell receives a predicted cell state label from the nearest-neighbor classifier (cellstate.predict)
- Projection runs on raw counts; if normalized data were supplied, the skip.normalize=TRUE flag was used

## Limitations

- UMAP embeddings are for visual exploration only; cell-cell distances and densities in UMAP are highly distorted, so quantification is done in PCA latent space
- Reference atlases do not yet encompass the full diversity of T cell transcriptional states, and query states may deviate from reference subtypes
- Gamma-delta T cells are not represented at all and CD4+ T cells are under-represented in the available reference atlases

## Evidence

- [methods] load.reference.map: "`load.reference.map`   Load or download the reference map for dataset projection"
- [methods] Run.ProjecTILs embedding: "It returns the query embedded in the PCA and UMAP space of the reference, and predict cell state labels for the query cells"
- [methods] cellstate.predict: "Use a nearest-neighbor algorithm to predict a feature (e.g. the cell state) of the query cells"
- [methods] composition barplot function: "`plot.statepred.composition`   Makes a barplot of the frequency of cell states in a query object"
- [readme] viral CD8 reference atlas content: [evidence span withheld — anchor status: external_doc]
- [readme] raw counts expectation: [evidence span withheld — anchor status: external_doc]
