---
name: cell-type-specificity-score-implementation
description: Use when annotating clusters of a processed scRNA-seq dataset (e.g.,
  Seurat object with scaled matrix) against a tissue-restricted marker database using
  both positive and negative markers, or when verifying a from-scratch reimplementation
  of the ScType scoring algorithm.
when_to_use_negative:
- The input is raw counts without Seurat normalization, scaling, and clustering —
  ScType scoring requires a z-scored scale.data matrix and predefined clusters
- A supervised classifier with a labeled reference dataset is required — ScType is
  an unsupervised marker-specificity method and was benchmarked against supervised
  tools like CaSTLe and ACTINN rather than replacing them
- No marker database covers the tissue of interest — the ScTypeDB restriction to a
  tissue type is a required input
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: ScType
  role: Provides the sctype_score_ and gene_sets_prepare R functions implementing
    the specificity and enrichment scoring being reimplemented
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: Seurat
  role: Used (v4.0) for data processing, normalization (LogNormalize, scale.factor
    = 10000), scaling all genes, PCA, and FindNeighbors/FindClusters clustering whose
    clusters receive ScType labels
- name: openxlsx
  role: Reads the ScTypeDB_full.xlsx marker database file when preparing gene sets
- name: dplyr
  role: Data manipulation when aggregating per-cell scores into cluster-level ScType
    scores
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-28803-w
    title: Fully-automated and ultra-fast cell-type identification using specific
      marker combinations from single-cell transcriptomic data
  source_content_hash: sha256:93cc9224856fd6d1fafb257a713a513df996e23f48a60cbdf95c92bd21d6cc9b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IanevskiAleksandr/sc-type
  tools:
  - R
  - ScType
  - Seurat
  - dplyr
  - openxlsx
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# cell-type-specificity-score-implementation

## Summary

Implements the ScType cell-type annotation scoring: z-score transformed gene expression multiplied by a cell-type specificity score S_i^t, combined into positive/negative marker-enrichment scores aggregated to per-cluster ScType scores. Reimplementation of the reference sctype_score_ function from Ianevski et al. 2022 (Nature Communications).

## When to use

Use when annotating clusters of a processed scRNA-seq dataset (e.g., Seurat object with scaled matrix) against a tissue-restricted marker database using both positive and negative markers, or when verifying a from-scratch reimplementation of the ScType scoring algorithm.

## When NOT to use

- The input is raw counts without Seurat normalization, scaling, and clustering — ScType scoring requires a z-scored scale.data matrix and predefined clusters
- A supervised classifier with a labeled reference dataset is required — ScType is an unsupervised marker-specificity method and was benchmarked against supervised tools like CaSTLe and ACTINN rather than replacing them
- No marker database covers the tissue of interest — the ScTypeDB restriction to a tissue type is a required input

## Inputs

- Seurat object with normalized (LogNormalize, scale.factor = 10000) and scaled (ScaleData on all genes) expression matrix and clustering (FindNeighbors/FindClusters, dims = 1:10, resolution = 0.8)
- ScType marker database file (ScTypeDB_full.xlsx or ScTypeDB_short.xlsx) with positive and negative markers per cell type and tissue
- Tissue type restriction string, e.g., 'Immune system'

## Outputs

- Cell-type by cell enrichment score matrix (es.max) from sctype_score
- Per-cluster ScType scores with top-scoring cell type assignment per cluster
- Verification table (e.g., sctype_reimplementation_verification.csv) comparing reimplemented vs. reference cluster labels and scores

## How to apply

Prepare positive and negative marker gene sets with gene_sets_prepare on ScTypeDB_full.xlsx restricted to the target tissue (e.g., 'Immune system'). Compute the specificity score S_i^t = 1 - (|M_i|_t - min(|M|_t))/(max(|M|_t) - min(|M|_t)) for each marker gene within each candidate cell type of the tissue. Z-score each gene's expression across all cells on the Seurat scale.data matrix (scaled = TRUE), multiply by S_i^t, and compute per-cell enrichment as the sum over positive markers minus negative markers, normalized by the square root of the marker-set size. Sum per-cell scores within each Seurat cluster to get cluster-level ScType scores, assign the top-scoring cell type per cluster, and label clusters with scores below ncells/4 as 'Unknown'. Validate by running the reference sctype_score_ function (scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative) on the same matrix and comparing cluster-level assignments.

## Related tools

- **ScType** (Provides the sctype_score_ and gene_sets_prepare R functions implementing the specificity and enrichment scoring being reimplemented) — https://github.com/IanevskiAleksandr/sc-type
- **Seurat** (Used (v4.0) for data processing, normalization (LogNormalize, scale.factor = 10000), scaling all genes, PCA, and FindNeighbors/FindClusters clustering whose clusters receive ScType labels)
- **openxlsx** (Reads the ScTypeDB_full.xlsx marker database file when preparing gene sets)
- **dplyr** (Data manipulation when aggregating per-cell scores into cluster-level ScType scores)

## Examples

```
es.max <- sctype_score(scRNAseqData = scRNAseqData, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)
```

## Evaluation signals

- Reimplemented per-cluster top scores match the reference es.max-derived annotations: identical cluster-level cell-type assignments on PBMC 3k
- Reimplemented cluster score values are near-identical to the reference sctype_score_ output (scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)
- Clusters with aggregate scores below ncells/4 are labeled 'Unknown', matching reference behavior
- Per-cell enrichment sums positive-marker terms minus negative-marker terms, each normalized by the square root of the marker-set size
- Specificity scores satisfy S_i^t = 1 - (|M_i|_t - min(|M|_t))/(max(|M|_t) - min(|M|_t)) within each candidate cell type of the restricted tissue

## Limitations

- Low or negative cluster-level ScType scores are treated as 'Unknown', so some clusters remain unannotated
- The z-score transformation used for combining markers may be suboptimal for single-cell transcriptomic data
- Negative markers are still relatively few compared to positive markers, limiting annotation power
- Drop-out events in scRNA-seq data are not corrected in the current version

## Evidence

- [discussion] z-score standardization: "ScType first standardizes each gene expression profile into z-score across all cells."
- [discussion] specificity weighting: [evidence span withheld — anchor status: not_found]
- [intro] negative markers rationale: "the expression of negative marker genes, which provide evidence against a cell being of a particular type, should also be incorporated into the cell-type identification process"
- [readme] gene set preparation from marker DB: "gs_list <- gene_sets_prepare("https://raw.githubusercontent.com/IanevskiAleksandr/sc-type/master/ScTypeDB_short.xlsx", "Immune system")"
- [readme] reference scoring call: "es.max <- sctype_score(scRNAseqData = scRNAseqData, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)"
