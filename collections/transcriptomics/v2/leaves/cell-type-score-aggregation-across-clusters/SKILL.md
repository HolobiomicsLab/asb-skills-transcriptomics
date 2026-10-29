---
name: cell-type-score-aggregation-across-clusters
description: Use when you have a cell-type-by-cell score matrix (es.max) from sctype_score()
  and Seurat cluster identities and must assign a single label per cluster, with an
  Unknown fallback for low-confidence assignments.
when_to_use_negative:
- The tissue of interest is absent from the ScType database (tissues comprising fewer
  than 5 cell types were excluded during database construction)
- Only per-cell scores are needed downstream (e.g. cell-level visualization) and cluster-level
  summarization would lose resolution
- A labeled reference dataset is available and a supervised classifier such as CaSTLe
  or ACTINN is preferred
tools:
- name: ScType
  role: Provides sctype_score() to compute per-cell cell-type scores from positive/negative
    marker sets, and gene_sets_prepare() to build those sets from ScTypeDB_full.xlsx
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: Seurat
  role: Produces the scaled expression matrix (scale.data slot) and cluster identities
    that per-cell scores are aggregated into
- name: R
  role: Open-source implementation language in which sctype_score_ and gene_sets_prepare
    are sourced and the row-sum aggregation and labeling are executed
  repo: https://github.com/IanevskiAleksandr/sc-type
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-28803-w
    title: Fully-automated and ultra-fast cell-type identification using specific
      marker combinations from single-cell transcriptomic data
  source_content_hash: sha256:db984c3a906cbfd97aeddc4830ec49955dcf4f235fd2ab70c0c55c196f72d4ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IanevskiAleksandr/sc-type
  tools:
  - R
  - ScType
  - Seurat
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cell-type-score-aggregation-across-clusters

## Summary

Converts per-cell ScType scores from sctype_score() into one cell-type label per Seurat cluster by aggregating scores (row-sums) per cluster, taking the top-scoring type, and labeling low-confidence clusters 'Unknown' when scores fall below ncells/4 — the decisive step that yields fully-automated cluster annotations.

## When to use

Use when you have a cell-type-by-cell score matrix (es.max) from sctype_score() and Seurat cluster identities and must assign a single label per cluster, with an Unknown fallback for low-confidence assignments.

## When NOT to use

- The tissue of interest is absent from the ScType database (tissues comprising fewer than 5 cell types were excluded during database construction)
- Only per-cell scores are needed downstream (e.g. cell-level visualization) and cluster-level summarization would lose resolution
- A labeled reference dataset is available and a supervised classifier such as CaSTLe or ACTINN is preferred

## Inputs

- Scaled expression matrix (Seurat scale.data slot) from a processed Seurat object
- gs_list$gs_positive and gs_list$gs_negative marker sets from gene_sets_prepare() on ScTypeDB_full.xlsx with tissue type 'Liver'
- Seurat cluster identities from FindNeighbors/FindClusters (dims = 1:10, resolution = 0.8)

## Outputs

- Cell-type-by-cluster aggregated score matrix (row-summed per-cell scores)
- Cluster-to-cell-type label table with scores (17 clusters -> 11 liver cell types for GSE124395)
- sctype_liver_annotations.csv cluster annotation file

## How to apply

After unsupervised clustering in Seurat (NormalizeData LogNormalize scale.factor=10000, 2000 vst variable features, PCA with 10 PCs, FindNeighbors/FindClusters at resolution 0.8), extract the scaled expression matrix from the scale.data slot. Prepare positive and negative marker sets with gene_sets_prepare() against ScTypeDB_full.xlsx for the matching tissue (e.g. 'Liver') and run sctype_score(scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative) to obtain a cell-type-by-cell score matrix. Aggregate these per-cell scores to clusters by row-sums, select the top-scoring cell type per cluster as its label, and set any label whose score is below ncells/4 to 'Unknown' to avoid forcing annotations on unconfident clusters. On the liver atlas benchmark (GSE124395) this procedure should reproduce 17 clusters mapped to 11 liver cell types, with B-cell clusters separated into immature B cells (CD19+/CD20+) and plasma cells (CD138+, CD19/CD20-negative).

## Related tools

- **ScType** (Provides sctype_score() to compute per-cell cell-type scores from positive/negative marker sets, and gene_sets_prepare() to build those sets from ScTypeDB_full.xlsx) — https://github.com/IanevskiAleksandr/sc-type
- **Seurat** (Produces the scaled expression matrix (scale.data slot) and cluster identities that per-cell scores are aggregated into)
- **R** (Open-source implementation language in which sctype_score_ and gene_sets_prepare are sourced and the row-sum aggregation and labeling are executed) — https://github.com/IanevskiAleksandr/sc-type

## Examples

```
es.max <- sctype_score(scRNAseqData = scRNAseqData, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative); ct.by.cluster <- rowsum(es.max, group = Idents(pbmc))
```

## Evaluation signals

- Each Seurat cluster receives exactly one cell-type label, and labels scoring below ncells/4 are 'Unknown' rather than forced assignments
- On GSE124395 the automated assignments reproduce the original manual annotations: 17 clusters covering 11 liver cell types
- The two jointly-annotated B-cell clusters separate into immature B cells (CD19+/CD20+) and plasma cells (CD138+, CD19/CD20-negative)
- The score matrix passed to sctype_score has dimensions cell-type x cell and was computed with scaled = TRUE on the scale.data slot
- The written sctype_liver_annotations.csv contains the cluster-to-type table with aggregated scores

## Limitations

- ScType utilizes z-score transformation for computationally-efficient combination of multiple markers, which may be suboptimal when dealing with single-cell transcriptomic data
- Drop-out events in scRNA-seq data are not corrected in the current version, affecting true negative-marker detection
- Negative markers remain relatively few compared to positive markers, limiting annotation power; low-scoring clusters default to 'Unknown'

## Evidence

- [other] Aggregate scores per Seurat cluster by row-sums, take the top-scoring cell type per cluster, and set labels with scores < ncells/4 to 'Unknown': [evidence span withheld — anchor status: not_found]
- [readme] sctype_score call on scaled data with positive and negative gene sets: "es.max <- sctype_score(scRNAseqData = scRNAseqData, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)"
- [discussion] z-score standardization underpinning the per-cell scores: "ScType first standardizes each gene expression profile into z-score across all cells."
- [methods] marker gene set preparation from ScType DB: "# load gene set preparation function
source("https://raw.githubusercontent.com/IanevskiAleksandr/sc-type/master/R/gene_sets_prepare.R")"
