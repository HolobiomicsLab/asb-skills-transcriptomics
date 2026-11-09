---
name: reference-atlas-construction
description: Use when building a stable, annotated reference map (e.g. mouse TIL atlas,
  LCMV viral CD8 atlas) from multiple heterogeneous scRNA-seq studies so new query
  datasets can later be projected and cell-state-labeled with ProjecTILs.
when_to_use_negative:
- A pre-defined ProjecTILs reference map already covers your biological context (mouse
  TIL, human CD8/CD4 TIL, viral CD8/CD4 T cell, DC, stromal atlases) — download it
  via load.reference.map instead of rebuilding.
- Your goal is annotating a new query dataset against an existing reference — use
  Run.ProjecTILs projection, not atlas construction.
- The cell type of interest is not represented in the source data (e.g. gamma-delta
  T cells, which were not represented at all in these reference atlases).
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: ProjecTILs
  role: Provides make.reference to convert the annotated integrated Seurat object
    into a custom reference atlas, and load.reference.map to load the saved map for
    later projections
  repo: https://github.com/carmonalab/ProjecTILs
- name: STACAS
  role: Semi-supervised scRNA-seq integration of heterogeneous multi-study datasets
    (anchor coverage, correction scale, k.anchor, k.weight parameters) prior to atlas
    curation
  repo: https://github.com/carmonalab/STACAS
- name: Seurat
  role: Data structure (Seurat objects) in which samples are loaded, integrated, annotated
    and stored as the reference atlas
- name: TILPRED
  role: Supervised classifier used with MAST marker-gene enrichment testing to annotate
    integrated cells into T cell functional states
  repo: https://github.com/carmonalab/TILPRED
- name: R
  role: Implementation language; ProjecTILs is an R package invoked via R help and
    functions like ?Run.ProjecTILs
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-23324-4
    title: Interpretation of T cell states from single-cell transcriptomics data using
      reference atlases
  source_content_hash: sha256:4fb62e0f2474d6b41da7987032aef0c096f2ea8cce415cf8955f7ab0224b54f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/ProjecTILs
  tools:
  - ProjecTILs
  - R
  - STACAS
  - Seurat
  - TILPRED
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# reference-atlas-construction

## Summary

Construct a cell-type-specific single-cell reference atlas by integrating heterogeneous public scRNA-seq studies with STACAS and annotating cells into functional subtypes via the TILPRED supervised classifier plus MAST marker-gene testing, producing a curated reference map loadable by ProjecTILs for query projection.

## When to use

Use when building a stable, annotated reference map (e.g. mouse TIL atlas, LCMV viral CD8 atlas) from multiple heterogeneous scRNA-seq studies so new query datasets can later be projected and cell-state-labeled with ProjecTILs.

## When NOT to use

- A pre-defined ProjecTILs reference map already covers your biological context (mouse TIL, human CD8/CD4 TIL, viral CD8/CD4 T cell, DC, stromal atlases) — download it via load.reference.map instead of rebuilding.
- Your goal is annotating a new query dataset against an existing reference — use Run.ProjecTILs projection, not atlas construction.
- The cell type of interest is not represented in the source data (e.g. gamma-delta T cells, which were not represented at all in these reference atlases).

## Inputs

- Public scRNA-seq datasets from multiple studies (e.g. 21 human melanoma/colon adenocarcinoma tumor samples and murine MC38_dLN; or P14 LCMV studies), loaded as Seurat objects
- Cell type/state labels or marker gene sets for supervised annotation (TILPRED classifier, MAST marker testing)

## Outputs

- A curated reference atlas saved as an .rds object (e.g. TIL.atlas.rds) with integrated PCA/UMAP space and functional subtype labels
- Reference map resolving the expected functional subtypes (e.g. nine TIL subtypes: naive-like CD4/CD8, EM, early-activation, Tex, Tpex, Th1-like, Tfh, Treg; or seven CD8 clusters for LCMV)

## How to apply

Download the source scRNA-seq samples (e.g. 21 human melanoma/colon adenocarcinoma tumor samples plus murine MC38_dLN, or P14 LCMV studies) and load them as Seurat objects. Integrate the heterogeneous multi-study data with STACAS — the integration tool designed for heterogeneous datasets — tuning its parameters exposed via the ProjecTILs API (STACAS.anchor.coverage, STACAS.correction.scale, STACAS.k.anchor, STACAS.k.weight). Annotate integrated cells into T cell functional states using the TILPRED supervised classifier combined with MAST marker-gene enrichment testing. Build the reference with make.reference, setting parameters such as ndim, nfeatures, recalculate.umap, umap.method, min_dist, n_neighbors, store.markers, n.markers and seed, then save the atlas (TIL.atlas.rds) for later use with load.reference.map. Verify the integrated atlas resolves the expected subtypes (nine functional subtypes for the tumor TIL atlas; seven functional CD8 clusters for the viral LCMV atlas) in its UMAP/label space before using it as a projection target.

## Related tools

- **ProjecTILs** (Provides make.reference to convert the annotated integrated Seurat object into a custom reference atlas, and load.reference.map to load the saved map for later projections) — https://github.com/carmonalab/ProjecTILs
- **STACAS** (Semi-supervised scRNA-seq integration of heterogeneous multi-study datasets (anchor coverage, correction scale, k.anchor, k.weight parameters) prior to atlas curation) — https://github.com/carmonalab/STACAS
- **Seurat** (Data structure (Seurat objects) in which samples are loaded, integrated, annotated and stored as the reference atlas)
- **TILPRED** (Supervised classifier used with MAST marker-gene enrichment testing to annotate integrated cells into T cell functional states) — https://github.com/carmonalab/TILPRED
- **R** (Implementation language; ProjecTILs is an R package invoked via R help and functions like ?Run.ProjecTILs)

## Examples

```
remotes::install_github("carmonalab/STACAS"); remotes::install_github("carmonalab/ProjecTILs"); ref <- make.reference(integrated.seurat, ndim=30, recalculate.umap=TRUE, store.markers=TRUE); saveRDS(ref, "TIL.atlas.rds")
```

## Evaluation signals

- The integrated atlas UMAP/label space resolves the expected number of functional subtypes (nine for the murine TIL atlas: naive-like CD4/CD8, EM, early-activation, Tex, Tpex, Th1-like, Tfh, Treg; seven for the LCMV CD8 atlas).
- The annotated Seurat object is successfully converted by make.reference and saved (TIL.atlas.rds) and can be reloaded with load.reference.map.
- Subtype labels show distinct phenotypes, functions, metabolic lifestyles, and preferential tissue distributions consistent with the curated annotations.
- STACAS integration parameters (anchor coverage, correction scale, k.anchor, k.weight) yield a coherent integrated space without a dominant batch/study structure.
- A test query projected onto the new atlas via Run.ProjecTILs embeds in the reference PCA/UMAP space and receives nearest-neighbor cell state labels without altering the reference structure.

## Limitations

- Reference atlases do not yet encompass the full diversity of T cell transcriptional states, particularly for CD4+ TILs which were under-represented compared to CD8+ among available data.
- Gamma-delta T cells are not represented at all in the reference atlases.
- No high-quality human T cell reference atlas yet exists; its construction is only anticipated as feasible in the future.
- A major challenge is integrating datasets from heterogeneous samples from multiple individuals, tissues, batches, protocols and technologies.

## Evidence

- [abstract] we generate reference T cell atlases for cancer and viral infection by multi-study integration: "we generate reference T cell atlases for cancer and viral infection by multi-study integration"
- [intro] STACAS designed for heterogeneous data integration: "We previously developed STACAS14, a bioinformatic tool for scRNA-seq data integration specifically designed for the challenges of integrating heterogeneous datasets"
- [methods] make.reference function documentation: "`make.projection`   Project a single-cell RNA-seq dataset onto a reference map of cellular states"
- [discussion] ProjecTILs available as R package with Docker image: "We have implemented ProjecTILs as an R package (https://github.com/carmonalab/ProjecTILs) and we provide a Docker image ready to use."
- [readme] load.reference.map README description: [evidence span withheld — anchor status: external_doc]
