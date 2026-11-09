---
name: marker-enrichment-score-aggregation-per-cluster
description: Use when annotating unsupervised Seurat clusters of an scRNA-seq dataset
  with a marker database (e.g. ScTypeDB restricted to a tissue), requiring combined
  positive and negative marker scoring per cluster.
when_to_use_negative:
- Input is not a Seurat-scaled expression matrix (e.g. raw counts or an already-annotated
  feature table) — z-scoring and S_i^t weighting assume the scale.data slot
- Marker database lacks negative markers for the tissue, since the score explicitly
  subtracts negative-marker enrichment
- The goal is supervised annotation from a labeled reference dataset (e.g. CaSTLe/ACTINN
  style), which ScType explicitly does not require or use
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: ScType
  role: Provides the specificity score S_i^t, z-score weighting, and marker-enrichment/cluster
    scoring algorithm being reimplemented
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: Seurat
  role: Data loading (Read10X, CreateSeuratObject min.cells = 3, min.features = 200),
    LogNormalize, ScaleData over all genes, PCA and FindNeighbors/FindClusters clustering
    whose clusters are the aggregation units
- name: gene_sets_prepare
  role: Builds gs_positive and gs_negative marker gene sets from ScTypeDB_full.xlsx
    for the selected tissue type
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: sctype_score_
  role: Reference implementation (scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)
    used to validate the reimplementation's per-cluster scores
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: dplyr
  role: R data manipulation for aggregating per-cell scores into cluster-level sums
- name: openxlsx
  role: Reads the ScTypeDB_full.xlsx marker database
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-28803-w
    title: Fully-automated and ultra-fast cell-type identification using specific
      marker combinations from single-cell transcriptomic data
  source_content_hash: sha256:b4b2579273d6a755b56261922d53e32b5c2f3d6e1bc2c8f98820ea269de991fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IanevskiAleksandr/sc-type
  tools:
  - R
  - ScType
  - Seurat
  - dplyr
  - gene_sets_prepare
  - openxlsx
  - sctype_score_
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# marker-enrichment-score-aggregation-per-cluster

## Summary

Implements the ScType scoring scheme: z-score gene expression, weight by the cell-type specificity score S_i^t, compute per-cell marker-enrichment scores from positive minus negative markers, and aggregate them into per-cluster ScType cell-type scores.

## When to use

Use when annotating unsupervised Seurat clusters of an scRNA-seq dataset with a marker database (e.g. ScTypeDB restricted to a tissue), requiring combined positive and negative marker scoring per cluster.

## When NOT to use

- Input is not a Seurat-scaled expression matrix (e.g. raw counts or an already-annotated feature table) — z-scoring and S_i^t weighting assume the scale.data slot
- Marker database lacks negative markers for the tissue, since the score explicitly subtracts negative-marker enrichment
- The goal is supervised annotation from a labeled reference dataset (e.g. CaSTLe/ACTINN style), which ScType explicitly does not require or use

## Inputs

- Seurat object with normalized (LogNormalize, scale.factor = 10000) and scaled (ScaleData over all genes) PBMC 3k counts from 10x filtered gene-barcode matrices
- Marker database file (ScTypeDB_full.xlsx) with positive and negative markers per cell type and tissue
- Cluster assignments from FindNeighbors/FindClusters (dims = 1:10, resolution = 0.8)

## Outputs

- Per-cluster ScType scores per candidate cell type (es.max-style cell-type by cluster matrix)
- Cluster-level cell-type label assignments, with low/negative-scoring clusters set to 'Unknown'
- Verification table comparing reimplemented vs reference annotations (e.g. sctype_reimplementation_verification.csv)

## How to apply

Prepare positive and negative marker gene sets with gene_sets_prepare from ScTypeDB_full.xlsx restricted to the target tissue (e.g. 'Immune system'). Z-score each gene's expression across all cells on the Seurat scale.data matrix, then multiply each gene's expression by its cell-type specificity score S_i^t = 1 - (|M_i|_t - min(|M|_t))/(max(|M|_t) - min(|M|_t)). Compute per-cell enrichment as the sum over positive markers minus negative markers, normalized by the square root of the marker-set size. Aggregate per-cell scores by summing within each Seurat cluster (FindNeighbors/FindClusters, dims = 1:10, resolution = 0.8), assign the top-scoring cell type per cluster, and label clusters whose score falls below ncells/4 as 'Unknown'. Validate by diffing cluster-level top scores and labels against the reference sctype_score_ output (scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative).

## Related tools

- **ScType** (Provides the specificity score S_i^t, z-score weighting, and marker-enrichment/cluster scoring algorithm being reimplemented) — https://github.com/IanevskiAleksandr/sc-type
- **Seurat** (Data loading (Read10X, CreateSeuratObject min.cells = 3, min.features = 200), LogNormalize, ScaleData over all genes, PCA and FindNeighbors/FindClusters clustering whose clusters are the aggregation units)
- **gene_sets_prepare** (Builds gs_positive and gs_negative marker gene sets from ScTypeDB_full.xlsx for the selected tissue type) — https://github.com/IanevskiAleksandr/sc-type
- **sctype_score_** (Reference implementation (scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative) used to validate the reimplementation's per-cluster scores) — https://github.com/IanevskiAleksandr/sc-type
- **dplyr** (R data manipulation for aggregating per-cell scores into cluster-level sums)
- **openxlsx** (Reads the ScTypeDB_full.xlsx marker database)

## Examples

```
gs_list <- gene_sets_prepare('ScTypeDB_full.xlsx','Immune system'); es.max <- sctype_score(scRNAseqData = pbmc[['RNA']]@scale.data, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)
```

## Evaluation signals

- Reimplemented per-cluster top cell-type assignments are identical to the reference es.max-derived annotations on PBMC 3k
- Reimplemented cluster-level score values are near-identical (within numerical tolerance) to the reference sctype_score_ output
- Clusters whose aggregated score is below ncells/4 are labeled 'Unknown' as in the reference workflow
- Enrichment scores are normalized by the square root of the marker-set size and combine positive minus negative marker sums
- Verification written to sctype_reimplementation_verification.csv shows matching labels per Seurat cluster

## Limitations

- Z-score transformation may be suboptimal for single-cell transcriptomic data, as acknowledged by the authors
- Negative markers are still relatively few compared to positive markers, limiting annotation power
- Drop-out events in scRNA-seq data are not corrected in the current version
- Clusters with insufficient scores are assigned 'Unknown' rather than a forced label

## Evidence

- [discussion] Standardize gene expression to z-score: "ScType first standardizes each gene expression profile into z-score across all cells."
- [discussion] Multiply gene expression by cell-type specificity score: [evidence span withheld — anchor status: not_found]
- [intro] Negative markers provide evidence against a cell type: "the expression of negative marker genes, which provide evidence against a cell being of a particular type, should also be incorporated"
- [readme] Reference scoring call signature: "es.max <- sctype_score(scRNAseqData = scRNAseqData, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)"
- [discussion] ScType uses marker gene specificity across clusters and cell types: "ScType is currently the only unsupervised method that makes use of the marker gene specificity across both cell clusters and cell types"
