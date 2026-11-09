---
name: census-hierarchical-cell-type-annotation
description: Use when a Seurat object with a UMAP reduction needs automated cell-type
  labels (normal or cancer tissues), or when you must test how many cells per cluster
  (e.g. 5–500) are required for stable Census annotations.
when_to_use_negative:
- Input is already a labeled feature/cell-type table with no Seurat UMAP reduction
  — Census requires a UMAP-embedded Seurat object for its label-stabilizing step
- Tissue or organ falls outside the 24 organs of the pretrained Tabula Sapiens model
  and no custom model has been trained
- Task is differential expression or clustering itself — Census annotates clusters,
  it does not perform the dimensionality reduction pipeline
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Census
  role: Hierarchical gradient-boosted decision tree cell-type annotation and cancer
    cell-of-origin prediction
  repo: https://github.com/sjdlabgroup/Census
- name: Seurat 4
  role: Required preprocessing (NormalizeData/ScaleData/RunPCA/RunUMAP), down-sampling,
    and re-embedding; must be loaded before Census library
- name: ggpubr
  role: Kruskal-Wallis non-parametric testing over F1 score distributions across down-sampling
    levels
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad714
    title: Hierarchical and automated cell-type annotation and inference of cancer
      cell of origin with Census
  source_content_hash: sha256:52ae287c5db4e78299e1ded7780164052482b078b8b3c4b5f61b9a0f44bc6102
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sjdlabgroup/Census
  tools:
  - Census
  - Seurat 4
  - ggpubr
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# census-hierarchical-cell-type-annotation

## Summary

Annotate single-cell RNA-seq data with Census, an R package of hierarchically organized gradient-boosted decision tree models pretrained on Tabula Sapiens (plus Cancer Cell Line Encyclopedia) that classifies cells down a cell-type hierarchy and stabilizes labels using clusters and prediction contours in UMAP space. Also supports down-sampling-based sensitivity assessment of annotation robustness.

## When to use

Use when a Seurat object with a UMAP reduction needs automated cell-type labels (normal or cancer tissues), or when you must test how many cells per cluster (e.g. 5–500) are required for stable Census annotations.

## When NOT to use

- Input is already a labeled feature/cell-type table with no Seurat UMAP reduction — Census requires a UMAP-embedded Seurat object for its label-stabilizing step
- Tissue or organ falls outside the 24 organs of the pretrained Tabula Sapiens model and no custom model has been trained
- Task is differential expression or clustering itself — Census annotates clusters, it does not perform the dimensionality reduction pipeline

## Inputs

- Seurat object (v4) with normalized expression data and a UMAP dimensionality reduction
- Organ specification (e.g. 'Pancreas', 'Lung') matching Census's 24-organ list
- For sensitivity analysis: a dataset with existing UMAP cluster annotations (e.g. Peng et al. 2019 pancreatic cancer, peng_sub.RDS)

## Outputs

- Per-cell Census annotations (res$pred with celltype; cancer_origin for cancer runs)
- F1 scores per cell-type per down-sampling level and Kruskal-Wallis test statistics (downsampled_f1_scores.csv)

## How to apply

Process the Seurat object through the standard pipeline (NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunUMAP(dims=1:50)), then run census_main(obj, organ=<organ>) with the pretrained Tabula Sapiens model (175 cell-types across 24 organs); for cancer datasets set predict_cancer=TRUE so an organ-specific model flags malignant epithelial cells and their predicted cell of origin (Pancreas, Kidney, Lung, Liver, Large_Intestine, Mammary models available). Seurat v4 must be loaded before the Census library. For sensitivity analysis, iteratively down-sample 5, 10, 20, 50, 100, and 500 cells per UMAP cluster, re-embed each subsample in UMAP with Seurat, re-annotate with Census, and compute per-cell-type F1 scores against original cluster-level labels; test differences across levels >=20 cells/cell-type with a Kruskal-Wallis test via ggpubr (reported P = 0.98), expecting F1 decline below 20 cells per cell-type.

## Related tools

- **Census** (Hierarchical gradient-boosted decision tree cell-type annotation and cancer cell-of-origin prediction) — https://github.com/sjdlabgroup/Census
- **Seurat 4** (Required preprocessing (NormalizeData/ScaleData/RunPCA/RunUMAP), down-sampling, and re-embedding; must be loaded before Census library)
- **ggpubr** (Kruskal-Wallis non-parametric testing over F1 score distributions across down-sampling levels)

## Examples

```
library(Seurat); library(Census); obj <- obj %>% NormalizeData() %>% ScaleData() %>% FindVariableFeatures() %>% RunPCA() %>% RunUMAP(dims=1:50); res <- census_main(obj, organ='Pancreas', predict_cancer=T)
```

## Evaluation signals

- Kruskal-Wallis test over F1 distributions for >=20 cells per cell-type should be non-significant (article reports P = 0.98)
- F1 scores should visibly decrease for down-sampling levels below 20 cells per cell-type
- Annotations must exist for all 175 possible cell-types in the hierarchy and cancer runs must return both celltype and cancer_origin columns in res$pred
- R session must have Seurat v4 loaded before library(Census), otherwise Census fails to run
- Output file downsampled_f1_scores.csv must contain per-level F1 scores and test statistics for levels 5, 10, 20, 50, 100, 500

## Limitations

- Census requires Seurat version 4 and it must be loaded prior to the Census library
- The pretrained main model covers only 175 cell-types from 24 organs; out-of-scope cell types need a custom model trained on another reference
- When multiple organs are passed with predict_cancer=TRUE, only the first organ is used for the cancer model; cancer models exist only for Kidney, Large_Intestine, Liver, Lung, Mammary, Pancreas
- Annotation reliability degrades when fewer than 20 cells per cell-type/cluster are available

## Evidence

- [intro] Census implements a collection of hierarchically organized gradient-boosted decision tree models that successively classify individual cells according to a predefined cell-type hierarchy: "Census implements a collection of hierarchically organized gradient-boosted decision tree models that successively classify individual cells according to a predefined cell-type hierarchy"
- [readme] README Seurat v4 requirement: "Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R."
- [readme] README standard Seurat pipeline and census_main call: "obj = obj %>% CreateSeuratObject() %>% NormalizeData() %>% ScaleData() %>%
  FindVariableFeatures() %>% RunPCA() %>% RunUMAP(dims=1:50)"
- [readme] README cancer prediction parameter: "Make sure to set the parameter `predict_cancer=T`. If more than one organ is specified in the `organ` parameter, the first one is used for the cancer model."
- [abstract] Abstract pretrained model scope: "Census is pretrained on the Tabula Sapiens to classify 175 cell-types from 24 organs; however, users can seamlessly train their own models for customized applications."
- [intro] Label-stabilizing algorithm: "using clusters and prediction contours in UMAP space to propagate higher confidence binary prediction outcomes"
