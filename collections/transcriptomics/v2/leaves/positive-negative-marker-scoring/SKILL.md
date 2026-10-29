---
name: positive-negative-marker-scoring
description: Use when clusters of closely related cell subtypes (e.g. naive vs effector
  T cells, immature B vs plasma cells) collapse to near-equal scores under positive-only
  markers, and negative marker evidence is needed to resolve them.
when_to_use_negative:
- When cell types of interest differ by well-separated positive markers alone and
  no negative markers are available — ScType's annotation depends on marker specificity
  across both clusters and cell types, and negative markers are still relatively few
  compared to positive markers
- When input is not raw scRNA-seq data but an already annotated or reference-labeled
  dataset — supervised methods like CaSTLe or ACTINN that train on labeled references
  are the appropriate choice
- When the marker database lacks entries for the tissue of interest (tissues with
  fewer than 5 cell types were excluded from the ScType database)
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: ScType
  role: Provides gene_sets_prepare and sctype_score functions that compute combined
    positive/negative marker scores per cell and per cluster
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: Seurat
  role: Normalizes (LogNormalize, scale.factor = 10000), scales, clusters (FindNeighbors/FindClusters,
    dims 1:10, resolution 0.8), and supplies the scale.data matrix scored by ScType
- name: dplyr
  role: Aggregates per-cluster scores via row sums and selects top-scoring cell types
    with top_n(n = 1, wt = scores)
- name: R
  role: Open-source implementation language for the ScType package and scoring workflow
  repo: https://github.com/IanevskiAleksandr/sc-type
provenance:
  source_task_ids:
  - task_002
  - task_006
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-28803-w
    title: Fully-automated and ultra-fast cell-type identification using specific
      marker combinations from single-cell transcriptomic data
  source_content_hash: sha256:c76fac82c4c6eb333df1a0676ca53de3fa51516712452439f64c0c98ca1441df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IanevskiAleksandr/sc-type
  tools:
  - R
  - ScType
  - Seurat
  - dplyr
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# positive-negative-marker-scoring

## Summary

Compute ScType cell-type scores by combining positive and negative marker gene sets, where negative markers (genes providing evidence against a cell type) suppress incorrect assignments. This is the core scoring mechanism that lets ScType distinguish closely related subtypes, e.g. naive vs effector T cells separated by CCR7/SELL as negative markers.

## When to use

Use when clusters of closely related cell subtypes (e.g. naive vs effector T cells, immature B vs plasma cells) collapse to near-equal scores under positive-only markers, and negative marker evidence is needed to resolve them.

## When NOT to use

- When cell types of interest differ by well-separated positive markers alone and no negative markers are available — ScType's annotation depends on marker specificity across both clusters and cell types, and negative markers are still relatively few compared to positive markers
- When input is not raw scRNA-seq data but an already annotated or reference-labeled dataset — supervised methods like CaSTLe or ACTINN that train on labeled references are the appropriate choice
- When the marker database lacks entries for the tissue of interest (tissues with fewer than 5 cell types were excluded from the ScType database)

## Inputs

- Scaled/normalized scRNA-seq expression matrix (Seurat scale.data slot) from a Seurat v4/v5 object
- ScTypeDB_full.xlsx marker database filtered by tissue type (gene_sets_prepare output: gs_positive and gs_negative lists)
- Negative marker genes for confusable cell types (e.g. CCR7 and SELL as negative markers for effector T cells)
- Seurat cluster assignments (from FindClusters at resolution 0.8 on the first 10 PCs)

## Outputs

- Cell-type-by-cell score matrix (es.max) from sctype_score
- Per-cluster cell-type annotation table with aggregated scores (cluster, cell type, score, ncells), e.g. sctype_liver_annotations.csv
- Per-cluster top-scoring T-cell type labels and scores for positive-only vs positive+negative runs
- sctype_classification column in Seurat metadata for UMAP overlay via DimPlot

## How to apply

Prepare tissue-matched positive (gs) and negative (gs2) gene sets with gene_sets_prepare() from ScTypeDB_full.xlsx, restricted to the relevant tissue type (e.g. 'Immune system', 'Liver'). Extract the scaled expression matrix from the Seurat object's scale.data slot and call sctype_score(scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative); ScType standardizes each gene expression profile into a z-score across all cells and multiplies expression by a cell-type specificity score before combining markers. Aggregate es.max scores per Seurat cluster by row sums, take the top-scoring cell type per cluster with dplyr::top_n(n = 1, wt = scores), and set clusters whose top score is below ncells/4 to 'Unknown'. To quantify the negative markers' effect, rerun sctype_score with gs2 = NULL (positive markers only) and compare per-cluster scores: in the PBMC example, positive-only scoring gave near-equal scores of 4 vs 7 for naive and effector T cells, whereas including CCR7/SELL as negative markers correctly suppressed the effector assignment. Correct application is confirmed when negative-marker-expressing clusters flip to the biologically expected label (e.g. CCR7+/SELL+ clusters called naive/memory rather than effector T cells).

## Related tools

- **ScType** (Provides gene_sets_prepare and sctype_score functions that compute combined positive/negative marker scores per cell and per cluster) — https://github.com/IanevskiAleksandr/sc-type
- **Seurat** (Normalizes (LogNormalize, scale.factor = 10000), scales, clusters (FindNeighbors/FindClusters, dims 1:10, resolution 0.8), and supplies the scale.data matrix scored by ScType)
- **dplyr** (Aggregates per-cluster scores via row sums and selects top-scoring cell types with top_n(n = 1, wt = scores))
- **R** (Open-source implementation language for the ScType package and scoring workflow) — https://github.com/IanevskiAleksandr/sc-type

## Examples

```
es.max <- sctype_score(scRNAseqData = scRNAseqData, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)
```

## Evaluation signals

- Clusters with top scores below ncells/4 are labelled 'Unknown', matching the paper's threshold rule
- Including negative markers changes the assignment for marker-expressing clusters: CCR7/SELL+ clusters are no longer scored as effector T cells
- Positive-only rerun reproduces the paper's reported near-equal scores for the two T-cell types (4 vs 7)
- Liver re-analysis yields 17 clusters across 11 cell types with B-cell clusters split into immature B cells (CD19+/CD20+) vs plasma cells (CD138+, CD19/CD20-negative), matching manual annotations
- Retina re-analysis separates rod bipolar (PRKCA+, CAR8+) from cone bipolar (SCGN+) clusters that the original study had merged

## Limitations

- The number of negative markers is still relatively low compared to positive markers, limiting current annotation power
- z-score transformation is used for computationally-efficient combination of multiple markers but may be suboptimal for single-cell transcriptomic data
- Drop-out events in scRNA-seq data are not corrected in the current version, which can mask true negative-marker expression
- The CellMarker database component did not list source publications, so marker overlap with benchmark datasets could not be confirmed (reproducibility gap)

## Evidence

- [results] negative markers suppress effector assignment: [evidence span withheld — anchor status: not_found]
- [intro] negative marker rationale: "the expression of negative marker genes, which provide evidence against a cell being of a particular type, should also be incorporated"
- [methods] scoring pipeline: [evidence span withheld — anchor status: not_found]
- [readme] marker combination: "es.max <- sctype_score(scRNAseqData = scRNAseqData, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)"
- [abstract] specificity guarantee: [evidence span withheld — anchor status: not_found]
