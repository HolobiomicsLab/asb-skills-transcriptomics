---
name: pseudo-bulk-profile-aggregation-from-scrna-seq
description: Use when you have a scRNA-seq reference with per-cell cell-type annotations
  (e.g., Tabula Sapiens) and need pseudo-bulk per-type profiles as training input
  for a hierarchical annotation model.
when_to_use_negative:
- Input is already bulk or pseudo-bulk expression data — no per-cell aggregation step
  is needed; go directly to hierarchy inference/model training
- Goal is annotating a new query dataset with the pretrained model — use census_main
  rather than rebuilding pseudo-bulk training profiles
- Reference lacks cell-type labels — pseudo-bulk aggregation groups by type, so unlabeled
  data must first be annotated or clustered
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: R framework for loading the scRNA-seq reference and its cell-type metadata
    used as the aggregation input; must be Seurat version 4, loaded before the Census
    library
- name: Census
  role: Consumes the pseudo-bulk-derived hierarchy to train hierarchically organized
    xgboost classifiers; installed via remotes::install_github('sjdlabgroup/Census',
    ref = 'main')
  repo: https://github.com/sjdlabgroup/Census
- name: xgboost
  role: Gradient-boosted decision tree learner used to train one binary classifier
    per internal node of the pseudo-bulk-derived hierarchy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad714
    title: Hierarchical and automated cell-type annotation and inference of cancer
      cell of origin with Census
  source_content_hash: sha256:f845e26a936ed0d04b4e553c2691c800f56bb16e9136bdc4c37ab23d7c2ae179
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sjdlabgroup/Census
  tools:
  - Census
  - Seurat
  - xgboost
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# pseudo-bulk-profile-aggregation-from-scrna-seq

## Summary

Aggregates single-cell RNA-seq counts into per-cell-type pseudo-bulk expression profiles from a labeled reference (e.g., Tabula Sapiens) so that stable, bulk-like profiles can be used to infer a cell-type hierarchy and train per-node Census xgboost classifiers. This is the first stage of building a custom Census model and mitigates single-cell sparsity during reference construction.

## When to use

Use when you have a scRNA-seq reference with per-cell cell-type annotations (e.g., Tabula Sapiens) and need pseudo-bulk per-type profiles as training input for a hierarchical annotation model.

## When NOT to use

- Input is already bulk or pseudo-bulk expression data — no per-cell aggregation step is needed; go directly to hierarchy inference/model training
- Goal is annotating a new query dataset with the pretrained model — use census_main rather than rebuilding pseudo-bulk training profiles
- Reference lacks cell-type labels — pseudo-bulk aggregation groups by type, so unlabeled data must first be annotated or clustered

## Inputs

- Labeled scRNA-seq reference as a Seurat object (e.g., Tabula Sapiens, 175 cell types / 24 organs) with cell-type annotations per cell
- Optional CCLE (Cancer Cell Line Encyclopedia) expression profiles to add cancer cell line reference nodes
- Cell-type-to-cell assignment table (metadata) used as the aggregation grouping variable

## Outputs

- Per-cell-type pseudo-bulk count/expression profile matrix
- Inferred cell-type hierarchy (hierarchy_mat) from Ward clustering of pseudo-bulk pairwise distances
- Collection of per-node xgboost binary classifiers (e.g., census_model.rds) with per-node training AUCs (main model: 351 nodes, median AUC 0.99)

## How to apply

In R, load Seurat v4 before the Census library, then load the labeled scRNA-seq reference (e.g., Tabula Sapiens) and aggregate raw single-cell counts into pseudo-bulk profiles per cell type (one profile per annotated type; Tabula Sapiens covers 175 cell types from 24 organs). Compute pairwise distances between the pseudo-bulk profiles and apply Ward hierarchical clustering to infer the cell-type hierarchy, optionally merging CCLE cancer cell line expression profiles as additional reference nodes. Train a binary xgboost gradient-boosted decision tree classifier at each internal node of the resulting hierarchy so that classification proceeds root-to-terminal. Serialize the trained node collection (e.g., census_model.rds) and record per-node training AUC; for the main Census model this yields a 351-node collection with median training AUC of 0.99. Validate that profile aggregation preserved per-type totals and that the hierarchy reproduces the expected cell-type relationships before training.

## Related tools

- **Seurat** (R framework for loading the scRNA-seq reference and its cell-type metadata used as the aggregation input; must be Seurat version 4, loaded before the Census library)
- **Census** (Consumes the pseudo-bulk-derived hierarchy to train hierarchically organized xgboost classifiers; installed via remotes::install_github('sjdlabgroup/Census', ref = 'main')) — https://github.com/sjdlabgroup/Census
- **xgboost** (Gradient-boosted decision tree learner used to train one binary classifier per internal node of the pseudo-bulk-derived hierarchy)

## Evaluation signals

- The number of aggregated pseudo-bulk profiles equals the number of annotated cell types in the reference (e.g., 175 Tabula Sapiens types plus added CCLE nodes)
- Per-type pseudo-bulk totals equal the sum of the underlying single-cell counts for that type (no cells dropped or double-counted)
- Ward hierarchical clustering of pseudo-bulk pairwise distances yields a hierarchy consistent with known cell-type (organ-to-type) relationships
- Trained model collection contains the expected node count (351 for the main Census model built from Tabula Sapiens + CCLE)
- Per-node training AUC distribution matches the reported benchmark (median AUC = 0.99)

## Limitations

- Census requires Seurat version 4 to run; Seurat v4 must be loaded prior to loading the Census library
- Pseudo-bulk quality depends on reference annotation accuracy — mislabeled cells contaminate per-type profiles
- The main pretrained model covers only 175 cell types from 24 organs and its cancer models are limited to Kidney, Large_Intestine, Liver, Lung, Mammary, Pancreas
- Ward clustering of pseudo-bulk profiles defines the hierarchy, so biologically wrong groupings will propagate to per-node classifiers

## Evidence

- [abstract] Pseudo-bulk aggregation precedes hierarchy inference: "Census is pretrained on the Tabula Sapiens to classify 175 cell-types from 24 organs; however, users can seamlessly train their own models for customized applications."
- [intro] Downstream use of aggregated profiles: "Census implements a collection of hierarchically organized gradient-boosted decision tree models that successively classify individual cells according to a predefined cell-type hierarchy"
- [readme] Seurat v4 requirement for the R workflow: "Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R."
