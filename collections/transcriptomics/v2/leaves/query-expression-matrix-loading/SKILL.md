---
name: query-expression-matrix-loading
description: Use when a new scRNA-seq dataset (e.g. the Lafleur Ptpn2-KO/WT P14 or
  Yao Tox-KO query matrices) must be loaded as a query object before Run.ProjecTILs
  projection onto a reference map.
when_to_use_negative:
- Input is already a projected Seurat object with a dimensionality reduction — re-loading
  via read.sc.query could discard it; use ProjecTILs.classifier or Run.ProjecTILs
  directly
- The goal is to integrate datasets into an atlas itself (multi-study reference construction)
  — that is a STACAS/make.reference task, not query loading
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ProjecTILs
  role: Provides read.sc.query to load the query expression matrix before projection
    onto the reference atlas
  repo: https://github.com/carmonalab/ProjecTILs
- name: Seurat
  role: Defines the Seurat object structure (counts/data layers) that read.sc.query
    produces for downstream Run.ProjecTILs use
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-23324-4
    title: Interpretation of T cell states from single-cell transcriptomics data using
      reference atlases
  source_content_hash: sha256:4a3dc5ef366f47b7c4a68b96b36c90ec6e1e3d156218122b443dcaee2b9564d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/ProjecTILs
  tools:
  - ProjecTILs
  - R
  - STACAS
  - Seurat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Query expression matrix loading

## Summary

Loads a scRNA-seq query expression matrix (e.g. 10x/hdf5 raw or log counts, or a Seurat object) into ProjecTILs via read.sc.query so it can be projected onto a reference atlas. Correct handling of the counts layer and normalization here is a prerequisite for accurate cell state label transfer.

## When to use

Use when a new scRNA-seq dataset (e.g. the Lafleur Ptpn2-KO/WT P14 or Yao Tox-KO query matrices) must be loaded as a query object before Run.ProjecTILs projection onto a reference map.

## When NOT to use

- Input is already a projected Seurat object with a dimensionality reduction — re-loading via read.sc.query could discard it; use ProjecTILs.classifier or Run.ProjecTILs directly
- The goal is to integrate datasets into an atlas itself (multi-study reference construction) — that is a STACAS/make.reference task, not query loading

## Inputs

- Query expression matrix in 10x/hdf5 format (e.g. Lafleur Ptpn2-KO/WT co-transfer matrix)
- Raw or log-normalized scRNA-seq counts for virus-specific CD8 T cells (e.g. Yao Tox-KO dataset)
- Seurat object query (query_example_seurat)

## Outputs

- Seurat query object with counts in the appropriate layer, ready for make.projection / Run.ProjecTILs

## How to apply

Load the query expression matrix with read.sc.query, which supports 10x/hdf5 formats and raw or log-normalized counts, returning a Seurat object ready for projection. ProjecTILs expects raw counts, which it will internally log-normalize; if you already use a custom normalization, place it in the 'data' layer and set skip.normalize=TRUE. The loaded query object is then passed to Run.ProjecTILs (reference-embedding mode) or ProjecTILs.classifier (label-only mode), so the choice of normalization directly affects nearest-neighbor cell state prediction and downstream subtype composition estimates.

## Related tools

- **ProjecTILs** (Provides read.sc.query to load the query expression matrix before projection onto the reference atlas) — https://github.com/carmonalab/ProjecTILs
- **Seurat** (Defines the Seurat object structure (counts/data layers) that read.sc.query produces for downstream Run.ProjecTILs use)

## Examples

```
library(ProjecTILs); query <- read.sc.query("Lafleur_Ptpn2-KO_expr.h5"); query.projected <- Run.ProjecTILs(query, ref = ref)
```

## Evaluation signals

- read.sc.query returns a valid Seurat object with a populated counts (or pre-normalized data) layer and no error on genes/cells assignment
- Skipping normalization invariant: raw counts input triggers internal log-normalization; skip.normalize=TRUE is set only when the 'data' layer already holds normalized values
- Gene identifiers in the query are compatible with the murine viral CD8 T cell reference (LCMV Arm/cl13 atlas) before projection
- Downstream Run.ProjecTILs on the loaded query reproduces expected subtype labels (e.g. Tex fraction >50% in Ptpn2-KO vs ~20% WT in the Lafleur co-transfer data)

## Limitations

- UMAP densities of the projected query are visually distorted and should not be used for quantitative judgments; cell state quantification is done in PCA space
- ProjecTILs expects raw counts by default; feeding log counts without setting skip.normalize=TRUE risks double normalization
- The viral reference atlases do not cover all T cell states (e.g. gamma-delta T cells are absent, CD4+ TILs under-represented), so query cells of such types will map poorly

## Evidence

- [methods] read.sc.query loads query expression matrix: "`read.sc.query`   Load a query expression matrix to be projected onto the reference atlas"
- [readme] raw counts expectation and internal normalization: [evidence span withheld — anchor status: external_doc]
- [readme] skip.normalize flag for pre-normalized data: [evidence span withheld — anchor status: external_doc]
- [methods] Run.ProjecTILs consumes the loaded query and embeds it: "It returns the query embedded in the PCA and UMAP space of the reference, and predict cell state labels for the query cells"
