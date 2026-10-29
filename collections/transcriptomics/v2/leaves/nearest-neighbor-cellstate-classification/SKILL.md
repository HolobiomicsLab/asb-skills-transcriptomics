---
name: nearest-neighbor-cellstate-classification
description: Use when a scRNA-seq query dataset (e.g., human tumor biopsies converted
  to mouse orthologs) has been projected onto a reference atlas via make.projection
  and you need per-cell state labels from the reference subtype vocabulary (e.g.,
  CD8 Tex, Tpex, Treg) with a nearest-neighbor vote.
when_to_use_negative:
- The query contains cell types absent from the chosen reference — gamma-delta T cells
  are not represented at all in the ProjecTILs reference atlases, and CD4+ TIL states
  are under-represented
- The input is not scRNA-seq count data (e.g., bulk RNA-seq or a pre-computed feature
  table) — the query object is expected to have raw counts, which ProjecTILs internally
  log-normalizes
- You need to re-dimensionally reduce or re-cluster the reference itself — ProjecTILs
  projects queries without altering the reference atlas structure
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ProjecTILs
  role: Provides cellstate.predict (nearest-neighbor cell state prediction) together
    with make.projection, load.reference.map and Run.ProjecTILs used to embed queries
    and transfer reference cell state labels
  repo: https://github.com/carmonalab/ProjecTILs
- name: Seurat
  role: Underlying Seurat object structures and PCA embeddings on which the nearest-neighbor
    label transfer operates
- name: R
  role: Implementation language; ProjecTILs is distributed as an R package and functions
    are documented via R help (e.g., ?Run.ProjecTILs)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-23324-4
    title: Interpretation of T cell states from single-cell transcriptomics data using
      reference atlases
  source_content_hash: sha256:add01ca7afb79f19fefadd3e71862251d18f76a2dd82cb75f98c1e08e32b5061
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/ProjecTILs
  tools:
  - ProjecTILs
  - R
  - Seurat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# nearest-neighbor-cellstate-classification

## Summary

Predicts cell state labels for a query scRNA-seq dataset by finding nearest neighbors of each query cell in the PCA space of a reference atlas (e.g., the murine TIL reference map), using ProjecTILs' cellstate.predict function. It transfers the reference's curated subtype annotations onto query cells while leaving the reference embedding intact.

## When to use

Use when a scRNA-seq query dataset (e.g., human tumor biopsies converted to mouse orthologs) has been projected onto a reference atlas via make.projection and you need per-cell state labels from the reference subtype vocabulary (e.g., CD8 Tex, Tpex, Treg) with a nearest-neighbor vote.

## When NOT to use

- The query contains cell types absent from the chosen reference — gamma-delta T cells are not represented at all in the ProjecTILs reference atlases, and CD4+ TIL states are under-represented
- The input is not scRNA-seq count data (e.g., bulk RNA-seq or a pre-computed feature table) — the query object is expected to have raw counts, which ProjecTILs internally log-normalizes
- You need to re-dimensionally reduce or re-cluster the reference itself — ProjecTILs projects queries without altering the reference atlas structure

## Inputs

- A projected Seurat query object (query cells embedded in the reference PCA space by make.projection)
- A reference map object loaded via load.reference.map (e.g., the mouse TIL atlas, 16,803 cells from 25 B16/MC38 samples)
- For human-to-mouse queries: an ortholog table generated with Hs2Mm.convert.table
- Optional parameters k (number of neighbors) and min.confidence (assignment confidence threshold)

## Outputs

- Per-cell cell state labels for query cells, drawn from the reference subtype vocabulary
- Confidence scores per predicted cell state (min.confidence threshold applied)
- Input for downstream summaries such as plot.statepred.composition barplots of cell state frequencies

## How to apply

First load the reference map (load.reference.map downloads the murine TIL atlas by default) and project the query into the reference PCA/UMAP space with make.projection (for cross-species queries, pre-convert human genes with Hs2Mm.convert.table and pass the ortholog_table parameter). Then run cellstate.predict, which uses a nearest-neighbor algorithm in the PCA latent space to vote on each query cell's state from the reference labels; use default k and min.confidence unless downstream analyses justify tuning, and rely on the confidence value to flag ambiguous assignments. Rationale: cell state quantification is done in PCA space because UMAP distances are highly distorted; the query embeddings are left intact and only labels are returned in ProjecTILs.classifier mode, whereas Run.ProjecTILs embeds the query into the reference space and returns predicted cell state labels. Validate assignments by checking the composition of predicted states (plot.statepred.composition) against expected biology — e.g., in the 132-biopsy human TIL meta-analysis, subtype-study averages clustered by reference subtype with human-mouse co-clustering at p < 3e-6.

## Related tools

- **ProjecTILs** (Provides cellstate.predict (nearest-neighbor cell state prediction) together with make.projection, load.reference.map and Run.ProjecTILs used to embed queries and transfer reference cell state labels) — https://github.com/carmonalab/ProjecTILs
- **Seurat** (Underlying Seurat object structures and PCA embeddings on which the nearest-neighbor label transfer operates)
- **R** (Implementation language; ProjecTILs is distributed as an R package and functions are documented via R help (e.g., ?Run.ProjecTILs))

## Examples

```
library(ProjecTILs); ref <- load.reference.map(); query.projected <- Run.ProjecTILs(query, ref=ref); query.projected$cellstate <- cellstate.predict(query.projected, ref=ref)
```

## Evaluation signals

- Predicted labels fall only within the reference atlas's documented subtype vocabulary (e.g., the nine TIL reference subtypes), never unlisted states
- Cells below the min.confidence threshold are flagged/uncertain rather than silently assigned; ambiguity rate is reported
- Cell state composition (via plot.statepred.composition) is biologically plausible for the input tissue/study, e.g., exhausted Tex predominating in tumor biopsies
- For cross-species queries, subtype-study averaged profiles co-cluster by reference subtype with human-mouse co-clustering at p < 3e-6
- Predictions are computed in the reference PCA latent space, and UMAP is used only for visual exploration, not for the neighbor computation

## Limitations

- Reference atlases do not yet encompass the full diversity of T cell transcriptional states, especially CD4+ TILs (under-represented compared to CD8+) and gamma-delta T cells (not represented at all)
- UMAP cell-cell distances and densities are highly distorted; ProjecTILs performs cell state quantifications in the PCA latent space and UMAP is for visual exploration only
- No high-quality human T cell reference atlas exists for all contexts; human queries are commonly interpreted against the murine atlas via ortholog mapping

## Evidence

- [methods] Use a nearest-neighbor algorithm to predict a feature (e.g. the cell state) of the query cells: "Use a nearest-neighbor algorithm to predict a feature (e.g. the cell state) of the query cells"
- [methods] Projection returns query embedded and labeled: "It returns the query embedded in the PCA and UMAP space of the reference, and predict cell state labels for the query cells"
- [discussion] Ortholog mapping for human queries: "ortholog mapping offers a viable solution to interpret human T cell responses in the context of a robust mouse atlas"
- [readme] Default reference map is mouse TIL atlas: [evidence span withheld — anchor status: external_doc]
- [readme] Query input expectations: [evidence span withheld — anchor status: external_doc]
