---
name: per-cluster-score-aggregation-and-thresholding
description: Use when a cell-by-cell-type score matrix (e.g. ScType es.max output
  on a clustered Seurat object) must be collapsed into one label per cluster, with
  a minimum-score threshold to flag unannotated clusters.
when_to_use_negative:
- The input is raw counts or unclustered cells — aggregation requires a prior FindClusters
  clustering and an es.max score matrix
- You need cell-level (not cluster-level) annotations or per-cell confidence values
  rather than a single label per cluster
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0089
tools:
- name: ScType
  role: Provides sctype_score for computing es.max scores using positive (gs) and
    negative (gs2) marker sets that feed the aggregation
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: Seurat
  role: Supplies the clustered Seurat object (FindNeighbors/FindClusters, dims = 1:10,
    resolution = 0.8) whose cluster identities group cells for score aggregation
- name: dplyr
  role: Used for group-by row-sum aggregation and top_n(n = 1, wt = scores) selection
    of the top-scoring cell type per cluster
- name: R
  role: Implementation environment for the aggregation and thresholding workflow
  repo: https://github.com/IanevskiAleksandr/sc-type
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-28803-w
    title: Fully-automated and ultra-fast cell-type identification using specific
      marker combinations from single-cell transcriptomic data
  source_content_hash: sha256:d8a0c0372fa761fa3e23fc558ee4e194462e0a8d8917aa808bbd3a56ff0ff55e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IanevskiAleksandr/sc-type
  tools:
  - R
  - ScType
  - Seurat
  - dplyr
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# per-cluster-score-aggregation-and-thresholding

## Summary

Aggregates ScType cell-type scores (es.max) per Seurat cluster by row sums, selects the top-scoring cell type per cluster with dplyr::top_n(n = 1, wt = scores), and labels low-confidence clusters as 'Unknown' when the aggregated score falls below ncells/4. This converts the cell-by-cell-type score matrix into discrete cluster-level annotations with an explicit confidence cutoff.

## When to use

Use when a cell-by-cell-type score matrix (e.g. ScType es.max output on a clustered Seurat object) must be collapsed into one label per cluster, with a minimum-score threshold to flag unannotated clusters.

## When NOT to use

- The input is raw counts or unclustered cells — aggregation requires a prior FindClusters clustering and an es.max score matrix
- You need cell-level (not cluster-level) annotations or per-cell confidence values rather than a single label per cluster

## Inputs

- es.max cell-type-by-cell score matrix from sctype_score (scaled scRNA-seq matrix, e.g. PBMC3k)
- Seurat object with cluster identities from FindNeighbors/FindClusters (dims = 1:10, resolution = 0.8)
- Positive (gs) and negative (gs2) marker gene sets from gene_sets_prepare on ScTypeDB_full.xlsx restricted to 'Immune system'

## Outputs

- Per-cluster top-scoring cell type labels with aggregated scores
- 'Unknown' designation for clusters scoring below ncells/4
- CSV table of per-cluster scores and labels for both positive+negative and positive-only runs

## How to apply

After computing the es.max cell-type-by-cell score matrix with sctype_score (using gs and optionally gs2 negative marker sets), aggregate per cluster by summing es.max rows within each Seurat cluster (row sums grouped by the cluster identity from Idents or pbmc@meta.data). For each cluster, apply dplyr::top_n(n = 1, wt = scores) to pick the highest-scoring cell type. Enforce a confidence cutoff: clusters whose aggregated score is below ncells/4 (one quarter of the number of cells in the cluster) are set to 'Unknown'. To compare annotation strategies (e.g. positive+negative markers vs. gs2 = NULL positive-only), rerun sctype_score, recompute per-cluster top scores, and verify that negative markers suppress incorrect assignments — in the PBMC3k example, positive-only scoring yielded near-equal naïve/effector T-cell scores (4 and 7) while negative markers (CCR7, SELL) corrected the naïve assignment. Write per-cluster scores and labels to a CSV table for comparison.

## Related tools

- **ScType** (Provides sctype_score for computing es.max scores using positive (gs) and negative (gs2) marker sets that feed the aggregation) — https://github.com/IanevskiAleksandr/sc-type
- **Seurat** (Supplies the clustered Seurat object (FindNeighbors/FindClusters, dims = 1:10, resolution = 0.8) whose cluster identities group cells for score aggregation)
- **dplyr** (Used for group-by row-sum aggregation and top_n(n = 1, wt = scores) selection of the top-scoring cell type per cluster)
- **R** (Implementation environment for the aggregation and thresholding workflow) — https://github.com/IanevskiAleksandr/sc-type

## Examples

```
es.max <- sctype_score(scRNAseqData = scRNAseqData, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative); scores <- es.max %>% aggregate(by = list(cluster = pbmc@meta.data[[cluster_col]]), FUN = sum); top <- scores %>% group_by(cluster) %>% top_n(n = 1, wt = scores)
```

## Evaluation signals

- Each cluster receives exactly one top-scoring cell type after top_n(n = 1, wt = scores); no ties or empty selections
- Clusters with aggregated score < ncells/4 are labeled 'Unknown', not force-annotated
- In the PBMC3k naïve/effector T-cell test, adding CCR7 and SELL as negative markers for effector T cells changes the label of CCR7/SELL-expressing clusters from effector to naïve/memory T cells
- Positive-only rerun (gs2 = NULL) reproduces the near-equal scores for the two T-cell types (4 vs 7), confirming the comparison baseline
- The per-cluster score/label CSV is written and matches the Seurat cluster identities 1:1

## Limitations

- ScType utilizes z-score transformation for combining multiple markers, which may be suboptimal when dealing with single-cell transcriptomic data
- Drop-out events in scRNA-seq data are not corrected in the current version, which can affect score reliability
- The ncells/4 threshold is a heuristic; the number of negative markers is still relatively low compared to positive markers, limiting annotation confidence

## Evidence

- [readme] top_n selection and Unknown thresholding: "# assign cell types
es.max <- sctype_score(scRNAseqData = scRNAseqData, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)"
- [intro] negative markers rationale: "the expression of negative marker genes, which provide evidence against a cell being of a particular type, should also be incorporated into the cell-type identification process"
- [abstract] ScType specificity via positive and negative markers: "we show how ScType provides unbiased and accurate cell type annotations by guaranteeing the specificity of positive and negative marker genes across cell clusters and cell types."
- [discussion] z-score standardization: "ScType first standardizes each gene expression profile into z-score across all cells."
- [readme] clustering parameters grounding the per-cluster grouping: "pbmc <- FindNeighbors(pbmc, dims = 1:10)
pbmc <- FindClusters(pbmc, resolution = 0.8)"
