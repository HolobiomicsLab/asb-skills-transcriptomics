---
name: gradient-boosted-tree-model-training
description: 'Use when building a custom Census cell-type classifier from a scRNA-seq
  reference: reference data must be aggregated to pseudo-bulk per cell type, a cell-type
  hierarchy inferred, and one binary xgboost model trained per internal node.'
when_to_use_negative:
- When the goal is annotation of a new dataset with the already-pretrained model —
  use census_main rather than retraining a custom model
- When the scRNA-seq reference is a Seurat object being loaded — remember Census requires
  Seurat version 4, and mismatched versions will break the workflow
- When the reference contains cell types with too few cells to form meaningful pseudo-bulk
  profiles for node splitting
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Census
  role: Main package providing the hierarchical model training functionality, hierarchy
    inference, and model serialization in R
  repo: https://github.com/sjdlabgroup/Census
- name: Seurat 4
  role: Required dependency for handling the scRNA-seq reference object; must be loaded
    before the Census library
- name: xgboost
  role: Gradient-boosted decision tree engine used to train one binary classifier
    per internal hierarchy node
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad714
    title: Hierarchical and automated cell-type annotation and inference of cancer
      cell of origin with Census
  source_content_hash: sha256:d5c72d3bde34f62e87cbc182c908bbfb040bfb8d9aaed5d608f6b6db159bc020
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# gradient-boosted-tree-model-training

## Summary

Training hierarchically organized gradient-boosted decision tree (xgboost) classifiers at each node of an inferred cell-type hierarchy, as implemented by the Census method for scRNA-seq annotation on Tabula Sapiens plus CCLE references.

## When to use

Use when building a custom Census cell-type classifier from a scRNA-seq reference: reference data must be aggregated to pseudo-bulk per cell type, a cell-type hierarchy inferred, and one binary xgboost model trained per internal node.

## When NOT to use

- When the goal is annotation of a new dataset with the already-pretrained model — use census_main rather than retraining a custom model
- When the scRNA-seq reference is a Seurat object being loaded — remember Census requires Seurat version 4, and mismatched versions will break the workflow
- When the reference contains cell types with too few cells to form meaningful pseudo-bulk profiles for node splitting

## Inputs

- scRNA-seq reference dataset (e.g., Tabula Sapiens) as a Seurat object with cell-type labels
- Optionally, cancer cell line expression profiles (Cancer Cell Line Encyclopedia) as additional reference nodes
- Pseudo-bulk profiles aggregated per cell type from single-cell counts

## Outputs

- A serialized collection of per-node xgboost models encoding the cell-type hierarchy (e.g., census_model.rds)
- Per-node training performance metrics (training AUC; median AUC 0.99 expected)
- A cell-type hierarchy matrix (as used by plot_cell_hierarchy with census_ts_model$hierarchy_mat)

## How to apply

In R, load Seurat v4 before loading the Census library (Census requires Seurat version 4), then load the scRNA-seq reference (e.g., Tabula Sapiens) and aggregate single-cell counts into pseudo-bulk profiles per cell type. Compute pairwise distances between pseudo-bulk profiles and apply Ward hierarchical clustering to infer the cell-type hierarchy, optionally merging CCLE cancer cell line expression profiles as additional reference nodes. Train a binary xgboost gradient-boosted decision tree classifier at each internal node of the hierarchy, capitalizing on nodal cell-type relationships, so prediction proceeds from the root node successively to terminal classifications. Serialize the complete node model collection (e.g., census_model.rds) and record per-node training AUC. The reference implementation trained 351 nodes with a median training AUC of 0.99, which serves as the quality benchmark for a correct training run.

## Related tools

- **Census** (Main package providing the hierarchical model training functionality, hierarchy inference, and model serialization in R) — https://github.com/sjdlabgroup/Census
- **Seurat 4** (Required dependency for handling the scRNA-seq reference object; must be loaded before the Census library)
- **xgboost** (Gradient-boosted decision tree engine used to train one binary classifier per internal hierarchy node)

## Examples

```
remotes::install_github('sjdlabgroup/Census', ref = 'main')
```

## Evaluation signals

- The trained model collection contains 351 nodes for the Tabula Sapiens hierarchy plus CCLE cancer models
- Per-node training AUC recorded, with median AUC = 0.99 as reported for the main Census model
- The serialized RDS loads and exposes a hierarchy_mat usable by plot_cell_hierarchy (e.g., census_ts_model$hierarchy_mat)
- Classification proceeds from the root node through successive node identities to terminal classifications without orphan nodes

## Limitations

- Census requires Seurat version 4 to run; Seurat v4 must be loaded prior to loading the Census library in R
- The pretrained main model identifies 175 cell-types from 24 organs; cell types outside this reference require custom training
- Training is specific to user-supplied references; benchmarked performance (44 atlas-scale normal and cancer tissues) applies to the published models, not arbitrary custom references

## Evidence

- [abstract] hierarchy and uses gradient-boosted decision trees: "Census infers hierarchical cell-type relationships and uses gradient-boosted \decision trees that capitalize on nodal cell-type relationships to achieve high prediction speed and accuracy."
- [intro] hierarchically organized models per node: "Census implements a collection of hierarchically organized gradient-boosted decision tree models that successively classify individual cells according to a predefined cell-type hierarchy"
- [readme] Seurat v4 load order requirement: "Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R."
- [readme] root-to-terminal classification procedure: "The Census model starts at the root node of the cell-type hierarchy and successively classifies cells according to their node identities until terminal classifications are reached."
- [abstract] Tabula Sapiens pretrained scope: "Census is pretrained on the Tabula Sapiens to classify 175 cell-types from 24 organs; however, users can seamlessly train their own models for customized applications."
