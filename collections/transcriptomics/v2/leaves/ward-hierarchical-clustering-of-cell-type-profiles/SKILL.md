---
name: ward-hierarchical-clustering-of-cell-type-profiles
description: Use when building a custom Census cell-type annotation model from a scRNA-seq
  reference (e.g., Tabula Sapiens) and you need a biologically intuitive cell-type
  hierarchy to organize the per-node xgboost classifiers.
when_to_use_negative:
- When you only need to annotate a new dataset with the pretrained main Census model
  — census_main already ships with the pretrained Tabula Sapiens + CCLE hierarchy
  and no re-clustering is needed.
- When the reference cell types lack trustworthy labels, since pseudo-bulk profiles
  are aggregated per cell-type annotation and mislabeled cells will distort the inferred
  hierarchy.
- When only Seurat v5 or later is installed — Census requires Seurat version 4 and
  will not run otherwise.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: Used (v4) to load and normalize the scRNA-seq reference and aggregate single-cell
    counts into pseudo-bulk profiles per cell type prior to clustering
- name: Census
  role: Consumes the inferred hierarchy as the predefined cell-type hierarchy organizing
    its per-node gradient-boosted decision tree classifiers
  repo: https://github.com/sjdlabgroup/Census
- name: xgboost
  role: Trains the binary classifier at each internal node of the Ward-derived hierarchy
    for successive root-to-terminal classification
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad714
    title: Hierarchical and automated cell-type annotation and inference of cancer
      cell of origin with Census
  source_content_hash: sha256:b8a25960a582847096b92be7a6e3eba74afe1357e2d94ac7be730db14370a3c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sjdlabgroup/Census
  tools:
  - Census
  - Seurat
  - xgboost
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# ward-hierarchical-clustering-of-cell-type-profiles

## Summary

Infer the cell-type hierarchy underlying a Census model by aggregating single-cell reference data (Tabula Sapiens scRNA-seq, plus CCLE cancer cell line profiles) into per-cell-type pseudo-bulk expression profiles, computing pairwise distances between them, and applying Ward hierarchical clustering to define the nodal relationships used by Census's gradient-boosted tree classifiers.

## When to use

Use when building a custom Census cell-type annotation model from a scRNA-seq reference (e.g., Tabula Sapiens) and you need a biologically intuitive cell-type hierarchy to organize the per-node xgboost classifiers.

## When NOT to use

- When you only need to annotate a new dataset with the pretrained main Census model — census_main already ships with the pretrained Tabula Sapiens + CCLE hierarchy and no re-clustering is needed.
- When the reference cell types lack trustworthy labels, since pseudo-bulk profiles are aggregated per cell-type annotation and mislabeled cells will distort the inferred hierarchy.
- When only Seurat v5 or later is installed — Census requires Seurat version 4 and will not run otherwise.

## Inputs

- Seurat object containing a scRNA-seq reference with cell-type annotations (e.g., Tabula Sapiens, 175 cell-types from 24 organs)
- Cancer cell line expression profiles (e.g., Cancer Cell Line Encyclopedia / CCLE) as additional reference nodes
- Cell-type label metadata for aggregating counts into pseudo-bulk profiles

## Outputs

- Cell-type hierarchy matrix (hierarchy_mat) encoding parent-child relationships among reference cell types
- Dendrogram of pseudo-bulk profile similarity used as the predefined cell-type hierarchy
- Set of internal nodes, each to be trained as a binary xgboost classifier (yielding the 351-node census_model.rds collection when combined with CCLE nodes)

## How to apply

First load Seurat v4 before the Census library in R (Census requires Seurat version 4). Aggregate the scRNA-seq reference counts into pseudo-bulk profiles per cell type (e.g., using Seurat's aggregation on the cell-type annotations from the Tabula Sapiens reference, which spans 175 cell-types from 24 organs). Compute pairwise distances between the pseudo-bulk profiles and apply Ward hierarchical clustering (hclust with Ward linkage) to infer the dendrogram of cell-type relationships; merge in CCLE cancer cell line expression profiles as additional reference nodes if cancer cell-of-origin inference is needed. The resulting hierarchy_mat defines the parent-child nodal relationships that Census's collection of hierarchically organized gradient-boosted decision tree models (one binary xgboost classifier per internal node) then traverses root-to-terminal. Rationale: Census capitalizes on nodal cell-type relationships so successive binary classifications from the root node until terminal classifications are reached yield high prediction speed and accuracy (median training AUC 0.99 across the 351-node model collection).

## Related tools

- **Seurat** (Used (v4) to load and normalize the scRNA-seq reference and aggregate single-cell counts into pseudo-bulk profiles per cell type prior to clustering)
- **Census** (Consumes the inferred hierarchy as the predefined cell-type hierarchy organizing its per-node gradient-boosted decision tree classifiers) — https://github.com/sjdlabgroup/Census
- **xgboost** (Trains the binary classifier at each internal node of the Ward-derived hierarchy for successive root-to-terminal classification)

## Examples

```
pb <- obj %>% AggregateExpression(by = 'celltype'); hc <- hclust(dist(pb), method = 'ward.D2'); hierarchy_mat <- cutree(hc, k = length(unique(obj$celltype)))
```

## Evaluation signals

- The inferred hierarchy plausibly groups biologically related cell types (e.g., organ- and lineage-related Tabula Sapiens types cluster together) when visualized via plot_cell_hierarchy on the hierarchy_mat.
- Training the node collection on the hierarchy yields 351 node models (Tabula Sapiens hierarchy plus CCLE cancer nodes) for the main Census model.
- Per-node training AUC values are recorded and the collection achieves a median training AUC of 0.99.
- Every reference cell type (175 Tabula Sapiens types from 24 organs, plus CCLE nodes) appears exactly once as a terminal classification in the hierarchy.
- The hierarchy_mat loads correctly into Census's Census:::census_ts_model-style model structure for root-to-terminal traversal.

## Limitations

- Census requires Seurat version 4 specifically; loading a different Seurat version before the Census library will break the workflow.
- The hierarchy is inferred from the reference's predefined cell-type labels, so hierarchy quality is bounded by the granularity and correctness of the reference annotations (175 cell-types from 24 organs in the main model).
- Only 6 organs have cancer models available (Kidney, Large_Intestine, Liver, Lung, Mammary, Pancreas); cancer cell-of-origin nodes outside these organs cannot be merged from CCLE profiles via this route.

## Evidence

- [abstract] Census infers hierarchical cell-type relationships and uses gradient-boosted decision trees that capitalize on nodal cell-type relationships to achieve high prediction speed and accuracy.: "Census infers hierarchical cell-type relationships and uses gradient-boosted \decision trees that capitalize on nodal cell-type relationships to achieve high prediction speed and accuracy."
- [intro] Census implements a collection of hierarchically organized gradient-boosted decision tree models that successively classify individual cells according to a predefined cell-type hierarchy: "Census implements a collection of hierarchically organized gradient-boosted decision tree models that successively classify individual cells according to a predefined cell-type hierarchy"
- [readme] The Census model starts at the root node of the cell-type hierarchy and successively classifies cells according to their node identities until terminal classifications are reached: "The Census model starts at the root node of the cell-type hierarchy and successively classifies cells according to their node identities until terminal classifications are reached"
- [readme] Census requires Seurat version 4 to run; load Seurat v4 before the Census library in R.: "Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R."
