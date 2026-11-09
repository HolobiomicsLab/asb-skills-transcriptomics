---
name: cross-condition-gene-expression-comparison
description: Use when two or more scRNA-seq conditions (e.g. anti-CD4-treated vs isotype-control
  P14 cells in chronic LCMV) must be compared for cell subtype frequencies or gene
  expression within a defined subtype, using a curated reference atlas as the common
  coordinate system.
when_to_use_negative:
- You need to discover novel cell states absent from the reference — use deviation-aware
  analyses instead, since subtype assignment is constrained to reference subtypes
- The query is gamma-delta T cells or under-represented CD4+ TIL states, which the
  reference atlases do not represent
- You need accurate cell-cell distances or density estimates from UMAP — the README
  warns UMAP embeddings are highly distorted and should be used for visual exploration
  only
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: ProjecTILs
  role: Projects each condition onto the shared reference atlas (Run.ProjecTILs),
    predicts subtype labels, and produces composition barplots (plot.statepred.composition)
    and gene-expression radar plots (plot.states.radar) for cross-condition comparison
  repo: https://github.com/carmonalab/ProjecTILs
- name: Seurat
  role: Provides the Seurat object framework for query data and downstream integration
    of ProjecTILs outputs
- name: STACAS
  role: Semi-supervised integration method underlying the multi-study construction
    of the ProjecTILs reference atlases
  repo: https://github.com/carmonalab/STACAS
- name: R
  role: Implementation environment for the ProjecTILs package and comparison workflow
  repo: https://github.com/carmonalab/ProjecTILs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-23324-4
    title: Interpretation of T cell states from single-cell transcriptomics data using
      reference atlases
  source_content_hash: sha256:91f77c432565c2ea873b44c6fb3d76ed79a6132edb8401b3867494b50123a417
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/ProjecTILs
  tools:
  - ProjecTILs
  - R
  - STACAS
  - Seurat
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cross-condition-gene-expression-comparison

## Summary

Compares subtype composition and gene expression of query scRNA-seq cells between experimental conditions by projecting each condition onto the same reference atlas with ProjecTILs, so that differences (e.g. SLEC-to-Tpex shifts, Pdcd1 elevation) are measured in a stable, shared coordinate system rather than via de novo clustering.

## When to use

Use when two or more scRNA-seq conditions (e.g. anti-CD4-treated vs isotype-control P14 cells in chronic LCMV) must be compared for cell subtype frequencies or gene expression within a defined subtype, using a curated reference atlas as the common coordinate system.

## When NOT to use

- You need to discover novel cell states absent from the reference — use deviation-aware analyses instead, since subtype assignment is constrained to reference subtypes
- The query is gamma-delta T cells or under-represented CD4+ TIL states, which the reference atlases do not represent
- You need accurate cell-cell distances or density estimates from UMAP — the README warns UMAP embeddings are highly distorted and should be used for visual exploration only

## Inputs

- Query scRNA-seq expression matrices for each condition (raw counts; internally log-normalized by ProjecTILs, or pre-normalized data in the Seurat 'data' layer with skip.normalize=TRUE)
- A ProjecTILs reference map (e.g. mouse viral infection CD8 T cell atlas, doi.org/10.6084/m9.figshare.12489518)
- A gene set of interest for radar-plot comparison (e.g. Pdcd1, Havcr2, Tox, Gzmb, Tcf7)

## Outputs

- Projected Seurat objects per condition embedded in the reference PCA/UMAP space with predicted cell state labels
- Condition-wise cell subtype frequency barplots (plot.statepred.composition, e.g. kanev_statepred_composition.png)
- Radar plots of gene-set expression per subtype per condition (plot.states.radar)

## How to apply

Load the appropriate reference map (here the mouse acute/chronic viral infection CD8 T cell atlas) with load.reference.map, then load each condition's query expression matrix with read.sc.query. Project each condition independently with Run.ProjecTILs, which wraps make.projection and cellstate.predict (nearest-neighbor cell state prediction) to embed the query in the reference PCA/UMAP space and assign subtype labels without altering the reference. Quantify per-condition subtype frequencies with plot.statepred.composition (barplot comparison of SLEC, Tpex, Tex proportions) and compare marker expression within a subtype (e.g. Pdcd1 in Tex cells) across conditions with plot.states.radar on a specified gene set. Because both conditions are mapped into the same intact reference space, compositional and expression differences are directly comparable; ProjecTILs computes quantifications in the PCA latent space, with UMAP for visualization only.

## Related tools

- **ProjecTILs** (Projects each condition onto the shared reference atlas (Run.ProjecTILs), predicts subtype labels, and produces composition barplots (plot.statepred.composition) and gene-expression radar plots (plot.states.radar) for cross-condition comparison) — https://github.com/carmonalab/ProjecTILs
- **Seurat** (Provides the Seurat object framework for query data and downstream integration of ProjecTILs outputs)
- **STACAS** (Semi-supervised integration method underlying the multi-study construction of the ProjecTILs reference atlases) — https://github.com/carmonalab/STACAS
- **R** (Implementation environment for the ProjecTILs package and comparison workflow) — https://github.com/carmonalab/ProjecTILs

## Examples

```
library(ProjecTILs); ref <- load.reference.map('viral'); q <- Run.ProjecTILs(read.sc.query('antiCD4_matrix.txt'), ref=ref); plot.statepred.composition(q); plot.states.radar(q, genes=c('Pdcd1','Havcr2','Tcf7'))
```

## Evaluation signals

- The composition barplot reproduces the expected cross-condition shift: SLEC decrease with reciprocal Tpex increase in the anti-CD4 condition, with Tex proportion unchanged
- Pdcd1 expression in the Tex subtype is elevated in the anti-CD4 condition versus isotype control on the plot.states.radar output
- The reference atlas PCA/UMAP structure is unchanged after projection (query embedded without altering the reference space)
- Both conditions' subtype frequencies are computed in the same PCA latent space, making barplot fractions directly comparable
- Query counts are raw and log-normalized internally (or skip.normalize=TRUE is used with pre-normalized data in the 'data' layer)

## Limitations

- UMAP-based visual comparisons distort cell-cell distances and densities; quantifications should rely on the PCA latent space
- Reference atlases do not capture the full diversity of T cell transcriptional states, especially CD4+ TILs and gamma-delta T cells
- Subtype comparisons are limited to reference-defined subtypes; novel states require deviation analyses
- No high-quality human T cell reference atlas yet exists; human data relies on ortholog mapping onto the mouse atlas

## Evidence

- [methods] Run.ProjecTILs ... returns the query embedded in the PCA and UMAP space of the reference, and predict cell state labels for the query cells: "It returns the query embedded in the PCA and UMAP space of the reference, and predict cell state labels for the query cells"
- [methods] plot.statepred.composition makes a barplot of cell state frequencies: "Makes a barplot of the frequency of cell states in a query object"
- [methods] plot.states.radar makes a radar plot of gene expression: "Makes a radar plot of the expression level of a specified set of genes"
- [readme] ProjecTILs quantifications are done in PCA space; UMAP for visualization only: [evidence span withheld — anchor status: external_doc]
- [abstract] ProjecTILs embeds query data without altering the reference structure, enabling direct comparison: "ProjecTILs allows not only accurate embedding of new scRNA-seq data into a reference without altering its structure"
- [readme] Your query object is expected to have scRNA-seq raw counts, which will be internally log-normalized by ProjecTILs: [evidence span withheld — anchor status: external_doc]
