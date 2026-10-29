---
name: hierarchical-node-classifier-training
description: Use when you need to train a custom Census cell-type classification model
  from a reference scRNA-seq dataset (e.g. Tabula Sapiens) rather than using the pretrained
  model, inferring the cell-type hierarchy from the reference and fitting one binary
  classifier per node.
when_to_use_negative:
- The goal is annotating a new dataset with normal/cancer cell types and cell of origin
  using the pretrained Tabula Sapiens/CCLE model — use the census_main prediction
  workflow instead of retraining
- The input is a single Seurat object needing annotation rather than a labeled reference
  dataset from which a hierarchy and node models can be trained
- The reference lacks multiple cell types sufficient to define nodal cell-type relationships
  for successive root-to-terminal classification
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Census
  role: R framework for inferring the cell-type hierarchy and training the hierarchically
    organized gradient-boosted decision tree node models
  repo: https://github.com/sjdlabgroup/Census
- name: Seurat 4
  role: Required dependency for processing the reference scRNA-seq data; must be loaded
    before the Census library
- name: xgboost
  role: Binary gradient-boosted decision tree classifier trained at each internal
    node of the hierarchy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad714
    title: Hierarchical and automated cell-type annotation and inference of cancer
      cell of origin with Census
  source_content_hash: sha256:b947bb1bbdf2e639386cfb573b0874f260d617b56069cf9c71a52a6e7faa7989
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sjdlabgroup/Census
  tools:
  - Census
  - Seurat 4
  - xgboost
  edam_operation: http://edamontology.org/operation_2995
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# hierarchical-node-classifier-training

## Summary

Train a Census-style collection of hierarchically organized binary xgboost gradient-boosted decision tree classifiers, one per internal node of a cell-type hierarchy inferred from pseudo-bulk reference profiles (Tabula Sapiens plus CCLE), yielding a serialized node model collection used for successive root-to-terminal cell-type classification.

## When to use

Use when you need to train a custom Census cell-type classification model from a reference scRNA-seq dataset (e.g. Tabula Sapiens) rather than using the pretrained model, inferring the cell-type hierarchy from the reference and fitting one binary classifier per node.

## When NOT to use

- The goal is annotating a new dataset with normal/cancer cell types and cell of origin using the pretrained Tabula Sapiens/CCLE model — use the census_main prediction workflow instead of retraining
- The input is a single Seurat object needing annotation rather than a labeled reference dataset from which a hierarchy and node models can be trained
- The reference lacks multiple cell types sufficient to define nodal cell-type relationships for successive root-to-terminal classification

## Inputs

- Reference scRNA-seq single-cell count matrix (e.g. Tabula Sapiens, 175 cell types from 24 organs) as a Seurat object
- Cancer cell line expression profiles (Cancer Cell Line Encyclopedia) merged as additional hierarchy nodes
- Cell-type labels for the reference profiles

## Outputs

- Serialized collection of trained per-node xgboost classifiers (census_model.rds), 351 nodes for the main model
- Inferred cell-type hierarchy matrix (e.g. census_ts_model$hierarchy_mat)
- Per-node training AUC values (median AUC 0.99 for the main Census model)

## How to apply

In R, load Seurat v4 before the Census library, then aggregate the reference scRNA-seq counts into pseudo-bulk profiles per cell type. Compute pairwise distances between pseudo-bulk profiles and apply Ward hierarchical clustering to infer the cell-type hierarchy, merging in CCLE cancer cell line expression profiles as additional reference nodes for cancer-of-origin capability. Train a binary xgboost gradient-boosted decision tree classifier at each internal node, exploiting nodal cell-type relationships so classification proceeds successively from the root node until terminal classifications are reached. Serialize the complete trained node collection (the main model is 351 nodes) to census_model.rds and record per-node training AUC; the reference implementation reports a median training AUC of 0.99. Users can also train custom models from other references using the same procedure.

## Related tools

- **Census** (R framework for inferring the cell-type hierarchy and training the hierarchically organized gradient-boosted decision tree node models) — https://github.com/sjdlabgroup/Census
- **Seurat 4** (Required dependency for processing the reference scRNA-seq data; must be loaded before the Census library)
- **xgboost** (Binary gradient-boosted decision tree classifier trained at each internal node of the hierarchy)

## Examples

```
remotes::install_github('sjdlabgroup/Census', ref = 'main')
```

## Evaluation signals

- The trained collection contains 351 node models when built on the Tabula Sapiens hierarchy plus CCLE cancer models
- Median per-node training AUC equals 0.99 (high training classification accuracy across nodes)
- The hierarchy matrix supports successive classification from the root node until terminal classifications are reached
- The full node model collection serializes successfully to census_model.rds
- Inference of the hierarchy reproduces via Ward clustering on pairwise distances between pseudo-bulk cell-type profiles

## Limitations

- Census requires Seurat version 4 to run; Seurat v4 must be loaded prior to loading the Census library in R
- The main pretrained model covers 175 cell types from 24 organs trained on Tabula Sapiens and CCLE; other tissues require training custom models
- In the cancer workflow, if more than one organ is specified in the organ parameter, only the first one is used for the cancer model

## Evidence

- [intro] hierarchical node model collection: "Census implements a collection of hierarchically organized gradient-boosted decision tree models that successively classify individual cells according to a predefined cell-type hierarchy"
- [abstract] gradient-boosted trees capitalize on nodal relationships: [evidence span withheld — anchor status: not_found]
- [intro] users can train custom models: "Census is pretrained on the Tabula Sapiens to classify 175 cell-types from 24 organs; however, users can seamlessly train their own models for customized applications."
- [readme] Seurat v4 loading requirement: "Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R."
