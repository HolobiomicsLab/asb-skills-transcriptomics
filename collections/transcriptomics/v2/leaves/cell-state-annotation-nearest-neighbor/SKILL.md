---
name: cell-state-annotation-nearest-neighbor
description: Use when query scRNA-seq count matrices (e.g. 10x/hdf5 CD45+ TIL or P14
  T cell data) must be assigned reference cell-state labels and subtype frequencies
  by projecting them onto a curated TIL or viral-infection reference atlas.
when_to_use_negative:
- The query needs an independent clustering/annotation because no relevant curated
  reference atlas exists — e.g. gamma-delta T cells, which are not represented at
  all in the available reference atlases
- Only label transfer is desired and the query's own dimensionality reduction must
  not be modified — use the classifier mode rather than full embedding projection
- Data are CD4+ TILs or other states under-represented in the reference, since reference
  atlases do not yet encompass the full diversity of T cell transcriptional states
tools:
- name: ProjecTILs
  role: R package providing load.reference.map, read.sc.query, make.projection, cellstate.predict
    and Run.ProjecTILs for nearest-neighbor cell-state annotation onto reference atlases
  repo: https://github.com/carmonalab/ProjecTILs
- name: Seurat
  role: Underlying Seurat object/infrastructure for query and reference objects; ProjecTILs
    is integrated with Seurat for upstream and downstream analyses
- name: STACAS
  role: Semi-supervised scRNA-seq integration used to build custom reference atlases
    by aligning heterogeneous studies (e.g. three LCMV P14 studies)
  repo: https://github.com/carmonalab/STACAS
provenance:
  source_task_ids:
  - task_004
  - task_007
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-23324-4
    title: Interpretation of T cell states from single-cell transcriptomics data using
      reference atlases
  source_content_hash: sha256:f58d8e6ba35f169afd0a86d7cf80aae9adaed296bcd987083b0f0b0b196f1401
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/ProjecTILs
  tools:
  - Azimuth
  - ProjecTILs
  - R
  - STACAS
  - Seurat
  - ggplot2
  - scmap
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cell-state-annotation-nearest-neighbor

## Summary

Annotates cell states of new scRNA-seq query data by nearest-neighbor label transfer within the stable coordinate system of a reference T cell atlas (ProjecTILs), preserving the reference embedding rather than rebuilding it. Used to quantify subtype composition shifts and gene-program changes across perturbation conditions.

## When to use

Use when query scRNA-seq count matrices (e.g. 10x/hdf5 CD45+ TIL or P14 T cell data) must be assigned reference cell-state labels and subtype frequencies by projecting them onto a curated TIL or viral-infection reference atlas.

## When NOT to use

- The query needs an independent clustering/annotation because no relevant curated reference atlas exists — e.g. gamma-delta T cells, which are not represented at all in the available reference atlases
- Only label transfer is desired and the query's own dimensionality reduction must not be modified — use the classifier mode rather than full embedding projection
- Data are CD4+ TILs or other states under-represented in the reference, since reference atlases do not yet encompass the full diversity of T cell transcriptional states

## Inputs

- Query single-cell expression count matrix in 10x/hdf5 format (raw or log counts), e.g. CD45+ B16 TIL or virus-specific P14 T cell matrices
- Pre-defined reference map (mouse TIL atlas, human CD8/CD4 TIL atlas, DC atlas, stromal atlas, or LCMV viral CD8/CD4 T cell atlas) loaded via load.reference.map
- Optionally a custom reference atlas built with make.reference and STACAS integration

## Outputs

- Projected Seurat object with query cells embedded in reference PCA/UMAP space and assigned cell state labels
- Cell-state subtype frequency tables/barplots per condition (plot.statepred.composition)
- Radar plots of gene-set expression per state and condition (plot.states.radar)
- Discriminant genes separating projected query from reference or control (find.discriminant.genes)

## How to apply

Load a curated reference atlas with load.reference.map (default: mouse TIL atlas; alternatives include the human CD8/CD4 TIL atlases and the LCMV viral CD8/CD4 T cell atlases), then read the query count matrix with read.sc.query, which accepts 10x/hdf5 and raw or log counts and expects raw counts to be internally log-normalized (or set skip.normalize=TRUE if data are already normalized in the 'data' layer). Run Run.ProjecTILs, which wraps make.projection (embedding the query in the reference PCA and UMAP space) and cellstate.predict (nearest-neighbor voting for cell-state labels, with parameters such as k, nn.decay and min.confidence); use ProjecTILs.classifier instead if only labels are needed and the query embedding should remain intact. Quantify subtype frequencies per condition with plot.statepred.composition barplots and compare marker genes (e.g. Pdcd1, Tnfrsf9, Ifng, Tcf7, Ccr7, Klrg1) between states/conditions with plot.states.radar and find.discriminant.genes. The rationale is that the reference space is left unaltered, so conditions are compared in a stable, annotated coordinate system.

## Related tools

- **ProjecTILs** (R package providing load.reference.map, read.sc.query, make.projection, cellstate.predict and Run.ProjecTILs for nearest-neighbor cell-state annotation onto reference atlases) — https://github.com/carmonalab/ProjecTILs
- **Seurat** (Underlying Seurat object/infrastructure for query and reference objects; ProjecTILs is integrated with Seurat for upstream and downstream analyses)
- **STACAS** (Semi-supervised scRNA-seq integration used to build custom reference atlases by aligning heterogeneous studies (e.g. three LCMV P14 studies)) — https://github.com/carmonalab/STACAS

## Examples

```
query.projected <- Run.ProjecTILs(query_example_seurat, ref=ref); ProjecTILs.classifier(query = query_object, ref = reference_map)
```

## Evaluation signals

- Cross-validated classification accuracy on the TIL reference should reach ~91.6%, and projection accuracy should exceed 90% within one UMAP unit (93.4% within five PCA units)
- For miR-155 KO vs WT B16 TILs: KO T cells shift toward the naive-like state (elevated Tcf7/Ccr7, reduced Tnfrsf9/Ifng) and T-cell fraction approximates 16% (KO) vs 8% (WT) of CD45+ cells
- For chronic LCMV anti-CD4 P14 data: composition barplot shows SLEC decrease with reciprocal Tpex increase, Tex proportion unchanged, and Pdcd1 elevated in Tex cells of the anti-CD4 condition
- For Ptpn2-KO co-transfer data: Tex fraction >50% in KO vs ~20% WT; for Tox-KO: SLEC expansion at the expense of Tpex with high Klrg1
- Query objects expect raw counts in the counts layer (internally log-normalized); verify normalization handling and that projected objects and all plots are generated successfully

## Limitations

- UMAP embeddings are for visual exploration only — cell-cell distances and densities in UMAP are highly distorted; state quantifications are performed in the PCA latent space
- Reference atlases under-represent CD4+ TILs and do not represent gamma-delta T cells at all; no high-quality human T cell reference atlas yet exists
- Cell cycle can confound projections (a cell.cycle.obj helper exists); human query data require Hs2Mm ortholog mapping onto the murine reference

## Evidence

- [methods] cellstate.predict: "`cellstate.predict`   Use a nearest-neighbor algorithm to predict a feature (e.g. the cell state) of the query cells"
- [methods] Run.ProjecTILs embedding: "It returns the query embedded in the PCA and UMAP space of the reference, and predict cell state labels for the query cells"
- [results] accuracy benchmark: [evidence span withheld — anchor status: not_found]
- [readme] README test invocation: "ref <- load.reference.map()
data(query_example_seurat)

query.projected <- Run.ProjecTILs(query_example_seurat, ref=ref)"
- [readme] UMAP caution: [evidence span withheld — anchor status: external_doc]
- [discussion] reference scope limitation: "they do not yet encompass the full diversity of transcriptional states that can be acquired by T cells, especially for CD4+ TILs"
