---
name: per-cell-prediction-table-generation
description: Use when scRNA-seq Seurat objects from tumor datasets (e.g. Massalha
  2020, Peng 2019, Pelka 2021 liver/pancreatic/colon cohorts) need per-cell malignant
  classification and cell-of-origin labels compiled into one prediction table.
when_to_use_negative:
- The goal is de novo discovery of novel malignant subtypes rather than assigning
  cells to Census's predefined hierarchy of 175 cell-types from 24 organs
- The tumor type has no available Census cancer model — only Kidney, Large_Intestine,
  Liver, Lung, Mammary, and Pancreas cancer models currently exist
- The input is bulk RNA-seq or a processed feature/summary table rather than single-cell
  data in a Seurat object with a UMAP reduction
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Census
  role: Provides the hierarchically organized gradient-boosted decision tree classifiers
    (census_main) and label-stabilizing algorithm that produce the per-cell cell-of-origin
    predictions
  repo: https://github.com/sjdlabgroup/Census
- name: Seurat 4
  role: Required dependency for Census; used to build and normalize the Seurat objects
    (NormalizeData, ScaleData, RunPCA, RunUMAP) that census_main consumes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad714
    title: Hierarchical and automated cell-type annotation and inference of cancer
      cell of origin with Census
  source_content_hash: sha256:dab9d1cc0d804b7842958798b03289d5b452b969c8441976ef853fff52625734
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sjdlabgroup/Census
  tools:
  - Census
  - Seurat 4
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# per-cell-prediction-table-generation

## Summary

Runs Census's hierarchically organized gradient-boosted decision tree classifiers (pretrained on Tabula Sapiens and CCLE) over tumor scRNA-seq Seurat objects to assign each malignant cell a terminal cell-of-origin call, then compiles all per-cell predictions into a single table (e.g. cell_of_origin_predictions.csv with cell barcode, dataset, organ, and predicted cell of origin columns).

## When to use

Use when scRNA-seq Seurat objects from tumor datasets (e.g. Massalha 2020, Peng 2019, Pelka 2021 liver/pancreatic/colon cohorts) need per-cell malignant classification and cell-of-origin labels compiled into one prediction table.

## When NOT to use

- The goal is de novo discovery of novel malignant subtypes rather than assigning cells to Census's predefined hierarchy of 175 cell-types from 24 organs
- The tumor type has no available Census cancer model — only Kidney, Large_Intestine, Liver, Lung, Mammary, and Pancreas cancer models currently exist
- The input is bulk RNA-seq or a processed feature/summary table rather than single-cell data in a Seurat object with a UMAP reduction

## Inputs

- Seurat v4 objects containing scRNA-seq expression matrices restricted to previously identified malignant cells (e.g. Massalha 2020, Ma 2021, Peng 2019, Hwang 2022, Lee 2020, Pelka 2021)
- Organ-specific Census CCLE-trained models for liver, pancreatic, and colon (Large_Intestine) tissues
- UMAP dimensionality reduction within each Seurat object (required by census_main)

## Outputs

- cell_of_origin_predictions.csv with per-cell columns: cell barcode, dataset, organ, and predicted cell of origin
- res$pred data frame with per-cell UMAP coordinates and celltype annotations from census_main
- res$cancer_origin per-cell cell-of-origin labels for malignant cells

## How to apply

Load Seurat v4 before the Census library, since Census requires Seurat version 4 to run, and install Census from GitHub (remotes::install_github('sjdlabgroup/Census', ref = 'main')). Process each expression matrix through the standard Seurat pipeline — CreateSeuratObject, NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunUMAP(dims=1:50) — because census_main requires a Seurat object containing a UMAP dimensionality reduction. Run census_main on each malignant-cell set with the organ parameter (Liver, Pancreas, Large_Intestine for colon) and predict_cancer = T so that classified epithelial cells are further evaluated by the organ-specific cancer model; if more than one organ is specified, only the first is used for the cancer model. The classifier starts at the root node of the 175-cell-type hierarchy and classifies cells successively until terminal classifications are reached, after which the label-stabilizing algorithm propagates higher-confidence binary predictions using clusters and prediction contours in UMAP space. Extract per-cell annotations from res$pred and the cell-of-origin labels from res$cancer_origin, then aggregate all six datasets into cell_of_origin_predictions.csv keyed by cell barcode, dataset, organ, and predicted cell of origin (hepatocellular vs cholangiocarcinoma for liver, ductal vs acinar-like for PDAC, enterocyte-lineage for colon).

## Related tools

- **Census** (Provides the hierarchically organized gradient-boosted decision tree classifiers (census_main) and label-stabilizing algorithm that produce the per-cell cell-of-origin predictions) — https://github.com/sjdlabgroup/Census
- **Seurat 4** (Required dependency for Census; used to build and normalize the Seurat objects (NormalizeData, ScaleData, RunPCA, RunUMAP) that census_main consumes)

## Examples

```
library(Seurat); library(Census); res = census_main(obj, organ = 'Pancreas', predict_cancer = T); write.csv(res$pred, 'cell_of_origin_predictions.csv')
```

## Evaluation signals

- Output CSV contains the required schema: cell barcode, dataset, organ, and predicted cell of origin for every malignant cell across all six datasets
- Liver cancer calls reproduce the hepatocellular versus cholangiocarcinoma split consistent with the original studies' clinical annotations
- Pancreatic calls separate ductal versus acinar-like PDAC states and colon calls assign enterocyte-lineage origins, matching the article's reported figures
- Every classified epithelial cell reaches a terminal classification after traversal from the root node of the cell-type hierarchy
- After label stabilization, cancer_origin labels form spatially coherent clusters/contours in UMAP space rather than scattered isolated calls

## Limitations

- If more than one organ is specified in the organ parameter, the first one is used for the cancer model, so multi-organ runs may silently apply the wrong cancer model
- Census requires Seurat version 4 specifically and a UMAP dimensionality reduction must already exist in the Seurat object
- Only epithelial cells flagged as such by the main model are evaluated for malignancy and cell of origin; available cancer models are limited to Kidney, Large_Intestine, Liver, Lung, Mammary, and Pancreas

## Evidence

- [intro] hierarchical GBT succession: "The Census model starts at the root node of the cell-type hierarchy and successively classifies cells according to their node identities until terminal classifications are reached"
- [intro] label stabilization: "using clusters and prediction contours in UMAP space to propagate higher confidence binary prediction outcomes"
- [readme] predict_cancer parameter: "Make sure to set the parameter `predict_cancer=T`."
- [readme] Seurat dependency: "Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R."
- [abstract] GBDT accuracy rationale: [evidence span withheld — anchor status: not_found]
- [abstract] model scope: "Census is pretrained on the Tabula Sapiens to classify 175 cell-types from 24 organs"
