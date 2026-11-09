---
name: tumor-infiltrating-lymphocyte-subtype-classification
description: Use when scRNA-seq data from tumor-infiltrating T cells (human or mouse)
  must be annotated into stable, reference-defined functional states, or when comparing
  TIL composition across cohorts, tissues, or perturbation conditions.
when_to_use_negative:
- When the query contains cell types absent from the reference (e.g. gamma-delta T
  cells, which were not represented at all in the reference atlases) — deviation analysis,
  not straightforward label transfer, is needed
- When the goal is discovery of novel, previously uncharacterized transcriptional
  states rather than assignment to known reference subtypes
- When a normalized expression matrix is provided instead of raw counts without setting
  skip.normalize=TRUE, risking double normalization
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3934
- http://edamontology.org/topic_3308
tools:
- name: ProjecTILs
  role: Projects query scRNA-seq data onto the reference TIL atlas and transfers cell
    state labels (Run.ProjecTILs / ProjecTILs.classifier / cellstate.predict)
  repo: https://github.com/carmonalab/ProjecTILs
- name: STACAS
  role: Semi-supervised multi-study integration of heterogeneous query datasets prior
    to projection, used to build custom reference atlases
  repo: https://github.com/carmonalab/STACAS
- name: Seurat
  role: Object model for query and reference data; ProjecTILs is integrated with Seurat
    for upstream and downstream analyses
- name: TILPRED
  role: Logistic regression classifier for CD8+ TIL states (Tex, Tpex/MemoryLike);
    superseded by ProjecTILs
  repo: https://github.com/carmonalab/TILPRED
- name: ProjecTILs.demo
  role: Tutorial repositories including a vignette for building a custom reference
    atlas for ProjecTILs
  repo: https://github.com/carmonalab/ProjecTILs.demo
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-23324-4
    title: Interpretation of T cell states from single-cell transcriptomics data using
      reference atlases
  source_content_hash: sha256:7c6ed8014bc058f111fb3a2453287a4fe4ae77bae7e5e1c38ae522d1abff3cb9
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3934
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# tumor-infiltrating-lymphocyte-subtype-classification

## Summary

Classifies tumor-infiltrating lymphocytes from scRNA-seq data into defined functional T cell subtypes (naive-like, EM, early-activation, Tex, Tpex, Th1-like, Tfh, Treg) by projecting query cells into a curated reference TIL atlas using ProjecTILs, rather than relying on subjective unsupervised annotation.

## When to use

Use when scRNA-seq data from tumor-infiltrating T cells (human or mouse) must be annotated into stable, reference-defined functional states, or when comparing TIL composition across cohorts, tissues, or perturbation conditions.

## When NOT to use

- When the query contains cell types absent from the reference (e.g. gamma-delta T cells, which were not represented at all in the reference atlases) — deviation analysis, not straightforward label transfer, is needed
- When the goal is discovery of novel, previously uncharacterized transcriptional states rather than assignment to known reference subtypes
- When a normalized expression matrix is provided instead of raw counts without setting skip.normalize=TRUE, risking double normalization

## Inputs

- Query Seurat object (or expression matrix via read.sc.query) containing scRNA-seq raw counts of T cells
- Reference TIL atlas .rds (e.g. TIL.atlas.rds) loadable by load.reference.map()
- Human-to-mouse ortholog conversion table (Hs2Mm.convert.table) when projecting human data onto a mouse reference

## Outputs

- Query object embedded in the reference PCA and UMAP space with predicted cell state labels per cell
- Cell state label probabilities (via ProjecTILs.classifier / cellstate.predict)
- Cell state composition barplots (plot.statepred.composition), gene-expression heatmaps (celltype.heatmap), and radar plots (plot.states.radar)

## How to apply

Load (or build) a reference TIL atlas with load.reference.map() — by default the mouse TIL atlas of 16,803 transcriptomes from 25 B16 melanoma and MC38 colon adenocarcinoma samples — ensuring raw counts in the query Seurat object's counts layer, which ProjecTILs internally log-normalizes. Project the query with Run.ProjecTILs to embed cells in the reference PCA/UMAP space and transfer cell state labels, or use the faster ProjecTILs.classifier mode for label transfer only, leaving query embeddings intact. For heterogeneous query data spanning multiple studies, first integrate with STACAS, tuning parameters such as STACAS.k.anchor, STACAS.k.weight, STACAS.anchor.coverage and STACAS.correction.scale; for human data projected onto the mouse atlas, rely on human-mouse ortholog mapping (Hs2Mm.convert.table). Supplement nearest-neighbor label prediction (cellstate.predict) with marker-gene enrichment testing (MAST), and summarize composition with plot.statepred.composition or celltype.heatmap. Interpret UMAP placement only visually, since cell-state quantification is performed in the PCA latent space.

## Related tools

- **ProjecTILs** (Projects query scRNA-seq data onto the reference TIL atlas and transfers cell state labels (Run.ProjecTILs / ProjecTILs.classifier / cellstate.predict)) — https://github.com/carmonalab/ProjecTILs
- **STACAS** (Semi-supervised multi-study integration of heterogeneous query datasets prior to projection, used to build custom reference atlases) — https://github.com/carmonalab/STACAS
- **Seurat** (Object model for query and reference data; ProjecTILs is integrated with Seurat for upstream and downstream analyses)
- **TILPRED** (Logistic regression classifier for CD8+ TIL states (Tex, Tpex/MemoryLike); superseded by ProjecTILs) — https://github.com/carmonalab/TILPRED
- **ProjecTILs.demo** (Tutorial repositories including a vignette for building a custom reference atlas for ProjecTILs) — https://github.com/carmonalab/ProjecTILs.demo

## Examples

```
library(ProjecTILs); ref <- load.reference.map(); query.projected <- Run.ProjecTILs(query_example_seurat, ref=ref)
```

## Evaluation signals

- The integrated atlas UMAP resolves the nine expected functional subtypes (naive-like CD4/CD8, EM, early-activation, Tex, Tpex, Th1-like, Tfh, Treg) in its label space
- Projection preserves the reference space structure: the reference embedding is unaltered and only query cells are added
- Query cells receive state labels whose marker-gene programs (e.g. Pdcd1, Havcr2, Tox, Gzmb low/Tcf7 for Tpex vs Tex) are consistent with the assigned subtype
- Silhouette coefficients (compute_silhouette) of query cells against the reference indicate reasonable agreement with assigned states
- Strong human-mouse conservation of assigned TIL subtypes, consistent with the reported meta-analysis across cohorts

## Limitations

- Reference atlases do not encompass the full diversity of T cell transcriptional states, especially CD4+ TILs which were under-represented compared to CD8+ in available data
- Gamma-delta T cells are not represented at all in the reference atlases
- No high-quality human T cell reference atlas existed at publication; human data must be ortholog-mapped onto the mouse atlas
- UMAP embeddings are for visual exploration only — cell-cell distances and densities in UMAP are highly distorted, and state quantification is done in PCA space

## Evidence

- [abstract] ProjecTILs allows accurate embedding of query data without altering reference structure: "ProjecTILs allows not only accurate embedding of new scRNA-seq data into a reference without altering its structure"
- [methods] load.reference.map loads or downloads the reference map used for projection: "`load.reference.map`   Load or download the reference map for dataset projection"
- [abstract] Strong conservation of T cell subtypes between human and mouse supports cross-species projection: "A meta-analysis of tumor-infiltrating T cells from several cohorts reveals a strong conservation of T cell subtypes between human and mouse"
- [readme] Default reference is the mouse tumor-infiltrating T cell atlas: [evidence span withheld — anchor status: external_doc]
- [readme] State quantification performed in PCA space; UMAP for visual exploration only: [evidence span withheld — anchor status: external_doc]
- [methods] Run.ProjecTILs returns query embedded in reference PCA/UMAP space with predicted labels: "It returns the query embedded in the PCA and UMAP space of the reference, and predict cell state labels for the query cells"
