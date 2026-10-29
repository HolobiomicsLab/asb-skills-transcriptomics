---
name: cross-study-clustering-meta-analysis
description: Use when you have expression data from multiple independent studies/cohorts
  (possibly across species) and need to test whether a shared reference annotation,
  rather than batch/study/species identity, drives transcriptional clustering.
when_to_use_negative:
- The reference atlas does not contain the cell types of interest — e.g. gamma-delta
  T cells 'were not represented at all' and CD4+ TILs were under-represented compared
  to CD8+ in the atlases
- The goal is a de novo unsupervised clustering of the query data without a stable
  annotated reference system of coordinates
- You need to alter or re-integrate the reference space with the query rather than
  embed queries without altering the reference
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: ProjecTILs
  role: R framework providing load.reference.map, read.sc.query, make.projection,
    cellstate.predict, find.discriminant.genes, and celltype.heatmap for reference-based
    projection and meta-analysis
  repo: https://github.com/carmonalab/ProjecTILs
- name: R
  role: implementation language; ProjecTILs is distributed as an R package
- name: Seurat
  role: ProjecTILs is integrated with Seurat; celltype.heatmap operates on Seurat
    objects to generate averaged gene expression heatmaps
- name: STACAS
  role: scRNA-seq integration dependency used for multi-study integration underlying
    the reference atlases
  repo: carmonalab/STACAS
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-23324-4
    title: Interpretation of T cell states from single-cell transcriptomics data using
      reference atlases
  source_content_hash: sha256:964750d63770428fd6539314408ce1b36bd0b4649332f1ad263cd7125ede56f2
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cross-study-clustering-meta-analysis

## Summary

A reference-based meta-analysis strategy in which subtype-study averaged expression profiles from multiple independent scRNA-seq cohorts are projected onto a common reference atlas and clustered, testing whether profiles co-cluster by reference cell subtype rather than by study, cohort, cancer type, or species.

## When to use

Use when you have expression data from multiple independent studies/cohorts (possibly across species) and need to test whether a shared reference annotation, rather than batch/study/species identity, drives transcriptional clustering.

## When NOT to use

- The reference atlas does not contain the cell types of interest — e.g. gamma-delta T cells 'were not represented at all' and CD4+ TILs were under-represented compared to CD8+ in the atlases
- The goal is a de novo unsupervised clustering of the query data without a stable annotated reference system of coordinates
- You need to alter or re-integrate the reference space with the query rather than embed queries without altering the reference

## Inputs

- 132 human tumor biopsy scRNA-seq expression matrices (10 studies, 7 cancer types)
- murine TIL reference atlas (16,803 single-cell transcriptomes from 25 B16/MC38 samples, 6 studies)
- human-to-mouse ortholog conversion table (Hs2Mm.convert.table)

## Outputs

- clustered averaged subtype-study gene expression heatmap (e.g. fig7_heatmap.png)
- per-subtype differential expression (DEG) tables (TSV)
- cell state label assignments and projected embeddings for each query biopsy
- co-clustering significance statistic (p < 3 × 10−6 for human-mouse subtype co-clustering)

## How to apply

Load the tumour-infiltrating lymphocyte reference map with load.reference.map and read each of the 132 human tumor biopsy expression matrices (10 studies, 7 cancer types) with read.sc.query. Convert human genes to mouse orthologs via Hs2Mm.convert.table and embed each query in the murine atlas PCA/UMAP space with make.projection, assigning cell states with cellstate.predict (nearest-neighbor, default k and min.confidence). Run find.discriminant.genes per reference subtype, comparing each projected human query against the murine reference as control, to obtain subtype-level differential expression tables. Build the averaged subtype-study gene expression matrix with celltype.heatmap and cluster the profiles: success is verified when subtype-study profiles co-cluster by TIL reference subtype across species (human-mouse co-clustering, p < 3 × 10−6), demonstrating conservation of T cell subtypes between human and mouse independent of study or cancer type.

## Related tools

- **ProjecTILs** (R framework providing load.reference.map, read.sc.query, make.projection, cellstate.predict, find.discriminant.genes, and celltype.heatmap for reference-based projection and meta-analysis) — https://github.com/carmonalab/ProjecTILs
- **R** (implementation language; ProjecTILs is distributed as an R package)
- **Seurat** (ProjecTILs is integrated with Seurat; celltype.heatmap operates on Seurat objects to generate averaged gene expression heatmaps)
- **STACAS** (scRNA-seq integration dependency used for multi-study integration underlying the reference atlases) — carmonalab/STACAS

## Examples

```
library(ProjecTILs); ref <- load.reference.map(); query <- read.sc.query("biopsy_matrix.tsv"); orthologs <- Hs2Mm.convert.table(rownames(query)); query.projected <- make.projection(ref, query, ortholog_table=orthologs); query.projected <- cellstate.predict(ref, query.projected)
```

## Evaluation signals

- Averaged subtype-study profiles cluster preferentially by TIL reference subtype rather than by study, cancer type, or species (co-clustering p < 3 × 10−6)
- Human and murine samples cluster together within each of the nine reference TIL subtypes
- Projected query cells occupy the reference PCA/UMAP space without altering the reference atlas structure
- Each projected query yields cell state labels with the nearest-neighbor classifier at default k and min.confidence, and per-subtype DEG tables from find.discriminant.genes
- Reference integrity is preserved: dimensionality reduction of the reference is unchanged by projection

## Limitations

- Reference atlases do not yet encompass the full diversity of T cell transcriptional states, especially CD4+ TILs (under-represented vs CD8+) and gamma-delta T cells (not represented at all)
- No high-quality human T cell reference atlas yet existed; human data must be interpreted through a murine atlas via ortholog mapping
- UMAP embeddings are for visual exploration only — cell-cell distances and densities in UMAP are highly distorted, so quantification is done in PCA space
- A robust definition of pre-exhausted/dysfunctional tumor-specific CD8+ TIL states has remained elusive, which can confound subtype interpretation

## Evidence

- [abstract] Meta-analysis of tumor-infiltrating T cells from several cohorts reveals strong conservation of T cell subtypes between human and mouse: "A meta-analysis of tumor-infiltrating T cells from several cohorts reveals a strong conservation of T cell subtypes between human and mouse"
- [discussion] 132-biopsy meta-analysis projection claim: "by meta-analysis of 132 cancer patient biopsies, we have shown that ProjecTILs can accurately project human T cell transcriptomes onto a reference mouse atlas"
- [discussion] ortholog mapping rationale: "ortholog mapping offers a viable solution to interpret human T cell responses in the context of a robust mouse atlas"
- [methods] celltype.heatmap function description: "`celltype.heatmap`   Generate a averaged gene expression heatmap from a Seurat object"
- [readme] default reference map is mouse TIL: [evidence span withheld — anchor status: external_doc]
- [readme] UMAP distortion caveat: [evidence span withheld — anchor status: external_doc]
