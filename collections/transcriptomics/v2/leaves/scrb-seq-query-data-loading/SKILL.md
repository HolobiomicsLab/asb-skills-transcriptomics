---
name: scrb-seq-query-data-loading
description: Use when a raw scRNA-seq expression matrix (query dataset) must be ingested
  and formatted as a Seurat object prior to Run.ProjecTILs reference-map projection.
when_to_use_negative:
- The data are already a projected/annotated Seurat object with cell state labels
  — loading again is unnecessary; proceed to Run.ProjecTILs or composition plotting
- The goal is label transfer only without reference embedding — use ProjecTILs.classifier
  mode instead of preparing for full Run.ProjecTILs embedding
- The input is bulk RNA-seq or a feature table without single-cell resolution — reference-map
  projection of query cells does not apply
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ProjecTILs
  role: Provides read.sc.query to load the query expression matrix and load.reference.map
    to fetch the reference atlas the query will be projected onto
  repo: https://github.com/carmonalab/ProjecTILs
- name: Seurat
  role: Data structure of the loaded query object; ProjecTILs is integrated with Seurat
    so loaded queries combine with upstream/downstream Seurat analyses
- name: R
  role: Language in which ProjecTILs is implemented and query loading is executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-23324-4
    title: Interpretation of T cell states from single-cell transcriptomics data using
      reference atlases
  source_content_hash: sha256:c8df0c5a5ea11bd222ce42625dead8a7a1c93c7d3ee31b288ff3e69647add42d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/ProjecTILs
  tools:
  - ProjecTILs
  - R
  - Seurat
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# scrb-seq-query-data-loading

## Summary

Load a query single-cell RNA-seq expression matrix into R as a Seurat object with read.sc.query, so it can be projected onto a ProjecTILs reference atlas. This step prepares heterogeneous query datasets (e.g. Kanev et al. P14 anti-CD4 vs isotype-control matrices) in the format the projection pipeline expects.

## When to use

Use when a raw scRNA-seq expression matrix (query dataset) must be ingested and formatted as a Seurat object prior to Run.ProjecTILs reference-map projection.

## When NOT to use

- The data are already a projected/annotated Seurat object with cell state labels — loading again is unnecessary; proceed to Run.ProjecTILs or composition plotting
- The goal is label transfer only without reference embedding — use ProjecTILs.classifier mode instead of preparing for full Run.ProjecTILs embedding
- The input is bulk RNA-seq or a feature table without single-cell resolution — reference-map projection of query cells does not apply

## Inputs

- Query scRNA-seq expression matrix (raw counts), e.g. Kanev et al. P14 anti-CD4-treated and isotype-control condition matrices
- Reference map object from load.reference.map() (e.g. mouse viral infection CD8 T cell reference atlas)
- Optionally a pre-existing Seurat query object (query_example_seurat-style)

## Outputs

- Seurat object containing the query expression data, ready for Run.ProjecTILs projection

## How to apply

Call load.reference.map() first to obtain the target reference (e.g. the mouse acute/chronic viral infection CD8 T cell atlas for LCMV P14 data), then load the query expression matrix with read.sc.query, which ingests the matrix and returns a Seurat object ready for projection. The query object is expected to contain raw scRNA-seq counts, which ProjecTILs will internally log-normalize; if a custom normalization is preferred, store normalized values in the 'data' layer and run downstream functions with skip.normalize=TRUE. For human query data projected onto a murine reference, apply human-mouse ortholog mapping (Hs2Mm.convert.table) at this stage. Verify the loaded object holds counts for the expected genes/cells before calling Run.ProjecTILs, which wraps make.projection and cellstate.predict and embeds the query in the reference PCA and UMAP space.

## Related tools

- **ProjecTILs** (Provides read.sc.query to load the query expression matrix and load.reference.map to fetch the reference atlas the query will be projected onto) — https://github.com/carmonalab/ProjecTILs
- **Seurat** (Data structure of the loaded query object; ProjecTILs is integrated with Seurat so loaded queries combine with upstream/downstream Seurat analyses)
- **R** (Language in which ProjecTILs is implemented and query loading is executed)

## Examples

```
library(ProjecTILs); ref <- load.reference.map("viral-CD8-T"); query <- read.sc.query("kanev_p14_antiCD4_counts.txt"); query.projected <- Run.ProjecTILs(query, ref=ref)
```

## Evaluation signals

- The returned object is a valid Seurat object with raw counts in the counts layer (or pre-normalized data in the 'data' layer if skip.normalize=TRUE is intended)
- Gene identifiers match the reference atlas space (ortholog mapping applied for human-onto-mouse projections)
- Cell and gene counts in the loaded object match the source expression matrices for each condition (anti-CD4 vs isotype control)
- Subsequent Run.ProjecTILs succeeds without normalization or dimension errors, embedding the query in the reference PCA/UMAP space

## Limitations

- UMAP embeddings produced downstream are for visual exploration only — cell-cell distances and densities in UMAP are highly distorted, so quantification is done in PCA space
- Reference atlases do not encompass the full diversity of T cell transcriptional states, especially CD4+ TILs (under-represented) and γδ T cells (not represented), limiting interpretability for such queries

## Evidence

- [methods] read.sc.query: "`read.sc.query`   Load a query expression matrix to be projected onto the reference atlas"
- [methods] load.reference.map: "`load.reference.map`   Load or download the reference map for dataset projection"
- [readme] counts normalization: [evidence span withheld — anchor status: external_doc]
- [readme] skip.normalize: [evidence span withheld — anchor status: external_doc]
- [discussion] ortholog mapping: "ortholog mapping offers a viable solution to interpret human T cell responses in the context of a robust mouse atlas"
