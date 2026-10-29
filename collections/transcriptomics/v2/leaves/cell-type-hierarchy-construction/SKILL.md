---
name: cell-type-hierarchy-construction
description: Use when training a custom Census cell-type annotation model from a scRNA-seq
  reference (e.g. Tabula Sapiens plus CCLE) that requires inferring the cell-type
  hierarchy and fitting node classifiers.
when_to_use_negative:
- The goal is annotating an existing dataset rather than building a model — use the
  pretrained census_main function with the Tabula Sapiens model instead of training
  a hierarchy.
- The reference has too few cell types or unlabelled cells, so pseudo-bulk profiles
  and a meaningful hierarchy cannot be inferred.
- Seurat v4 is not available in the environment — Census requires Seurat version 4
  and will not run correctly with other versions.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Census
  role: R package that infers hierarchical cell-type relationships, trains hierarchically
    organized gradient-boosted decision tree (xgboost) node models, and serializes
    the model collection for successive root-to-terminal classification
  repo: https://github.com/sjdlabgroup/Census
- name: Seurat 4
  role: Processes the scRNA-seq reference (NormalizeData/ScaleData/FindVariableFeatures/RunPCA/RunUMAP)
    and must be loaded before the Census library; supplies the Seurat object input
    for model training
- name: xgboost
  role: Gradient-boosted decision tree learner used to fit the binary classifier at
    each internal node of the cell-type hierarchy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad714
    title: Hierarchical and automated cell-type annotation and inference of cancer
      cell of origin with Census
  source_content_hash: sha256:027d0944e7ea041802a6a87fbd3c520b9868bb36c5bae7e82f4c43c1f8b3e22a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sjdlabgroup/Census
  tools:
  - Census
  - Seurat 4
  - xgboost
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# cell-type-hierarchy-construction

## Summary

Builds a Census cell-type hierarchy by aggregating single-cell counts into pseudo-bulk profiles per cell type, computing pairwise distances, and applying Ward hierarchical clustering (merging CCLE cancer cell line profiles as extra reference nodes), then training a binary xgboost gradient-boosted decision tree classifier at each internal node. This is how custom Census models are created beyond the pretrained Tabula Sapiens model.

## When to use

Use when training a custom Census cell-type annotation model from a scRNA-seq reference (e.g. Tabula Sapiens plus CCLE) that requires inferring the cell-type hierarchy and fitting node classifiers.

## When NOT to use

- The goal is annotating an existing dataset rather than building a model — use the pretrained census_main function with the Tabula Sapiens model instead of training a hierarchy.
- The reference has too few cell types or unlabelled cells, so pseudo-bulk profiles and a meaningful hierarchy cannot be inferred.
- Seurat v4 is not available in the environment — Census requires Seurat version 4 and will not run correctly with other versions.

## Inputs

- scRNA-seq reference dataset with per-cell cell-type labels (e.g. Tabula Sapiens, 175 cell-types from 24 organs)
- Seurat object / single-cell counts matrix for the reference
- Optional additional reference profiles merged into the hierarchy (e.g. Cancer Cell Line Encyclopedia (CCLE) cancer cell line expression profiles)

## Outputs

- Cell-type hierarchy matrix (e.g. census_ts_model$hierarchy_mat) defining root-to-terminal node relationships
- Serialized trained node model collection (census_model.rds) of binary xgboost classifiers, one per internal node (351 nodes for the main Tabula Sapiens + CCLE model)
- Per-node training classification metrics (training AUC; median AUC 0.99 for the main model)

## How to apply

In R, first load Seurat v4 before the Census library (Census requires Seurat version 4), then install Census via remotes::install_github('sjdlabgroup/Census', ref = 'main'). Load the scRNA-seq reference (e.g. Tabula Sapiens) and aggregate single-cell counts into pseudo-bulk profiles per cell type; compute pairwise distances between pseudo-bulk profiles and apply Ward hierarchical clustering to infer the cell-type hierarchy, merging CCLE cancer cell line expression profiles in as additional reference nodes. At each internal node of the hierarchy, train a binary xgboost gradient-boosted decision tree classifier that exploits nodal cell-type relationships for successive root-to-terminal classification, mirroring how the main model classifies cells from the root node until terminal classifications are reached. Serialize the complete trained node model collection (e.g. census_model.rds) and record per-node training AUC. Rationale: Census infers hierarchical cell-type relationships and uses gradient-boosted decision trees because capitalizing on nodal cell-type relationships yields high prediction speed and accuracy, and the design lets users seamlessly train their own models for customized applications.

## Related tools

- **Census** (R package that infers hierarchical cell-type relationships, trains hierarchically organized gradient-boosted decision tree (xgboost) node models, and serializes the model collection for successive root-to-terminal classification) — https://github.com/sjdlabgroup/Census
- **Seurat 4** (Processes the scRNA-seq reference (NormalizeData/ScaleData/FindVariableFeatures/RunPCA/RunUMAP) and must be loaded before the Census library; supplies the Seurat object input for model training)
- **xgboost** (Gradient-boosted decision tree learner used to fit the binary classifier at each internal node of the cell-type hierarchy)

## Examples

```
remotes::install_github('sjdlabgroup/Census', ref = 'main'); library(Seurat); library(Census)
```

## Evaluation signals

- The trained model collection contains 351 nodes (Tabula Sapiens hierarchy plus CCLE cancer models).
- Per-node training AUC values are recorded and the collection achieves a median training AUC of 0.99.
- The hierarchy can be visualized with plot_cell_hierarchy(Census:::census_ts_model$hierarchy_mat, ...) and matches the expected root-to-terminal cell-type structure.
- Loading order check: Seurat v4 is loaded before the Census library in the R session, otherwise Census fails to run.
- Successive root-to-terminal classification on held-out cells reaches terminal classifications for all nodes, confirming each internal node classifier was fitted.

## Limitations

- Census requires Seurat version 4 to run; Seurat v4 must be loaded prior to loading the Census library in R.
- The pretrained main model is limited to 175 cell-types from 24 organs and a fixed set of organs; cancer models are currently only available for Kidney, Large_Intestine, Liver, Lung, Mammary, and Pancreas, and if more than one organ is specified the first one is used for the cancer model.
- The reported 351-node / median AUC 0.99 figures are training-set performance for the main Tabula Sapiens + CCLE model, not guaranteed for arbitrarily small or noisy custom references.

## Evidence

- [abstract] Census infers hierarchical cell-type relationships and uses gradient-boosted decision trees: [evidence span withheld — anchor status: not_found]
- [intro] Hierarchically organized GBDT models classify cells successively: "Census implements a collection of hierarchically organized gradient-boosted decision tree models that successively classify individual cells according to a predefined cell-type hierarchy"
- [abstract] Users can train custom models for customized applications: "Census is pretrained on the Tabula Sapiens to classify 175 cell-types from 24 organs; however, users can seamlessly train their own models for customized applications."
- [readme] Seurat v4 must be loaded before Census: "Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R."
- [readme] Census installation command: "remotes::install_github('sjdlabgroup/Census', ref = 'main')"
