---
name: organ-specific-malignant-cell-model-application
description: Use when tumor scRNA-seq data (Seurat object with malignant epithelial
  cells pre-identified) must be assigned a cancer cell of origin using Census organ-specific
  CCLE-trained models for Liver, Pancreas, or Large_Intestine (colon).
when_to_use_negative:
- The tissue organ has no Census cancer model — only Kidney, Large_Intestine, Liver,
  Lung, Mammary, Pancreas are available; other organs cannot be run with predict_cancer=T
- Malignant cells have not been pre-identified and the goal is general normal-tissue
  annotation — census_main without predict_cancer is the appropriate entry point
- The environment has Seurat v5 loaded — Census requires Seurat version 4 and will
  not run correctly
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Census
  role: Provides the hierarchically organized gradient-boosted decision tree classifiers
    and organ-specific CCLE-trained cancer models via census_main(predict_cancer =
    T), returning per-cell cancer_origin predictions
  repo: https://github.com/sjdlabgroup/Census
- name: Seurat 4
  role: Preprocessing (NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunUMAP)
    to produce the Seurat object with UMAP required by Census; must be loaded as version
    4
- name: ggpubr
  role: Publication-ready ggplot2-based plotting support for visualizing Census prediction
    outputs
  repo: https://github.com/kassambara/ggpubr
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad714
    title: Hierarchical and automated cell-type annotation and inference of cancer
      cell of origin with Census
  source_content_hash: sha256:b18aef4dd90d3ed67a9d7e3d7f4a6a81e665c113ac906f5d8fede11f27446e2a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sjdlabgroup/Census
  tools:
  - Census
  - Seurat 4
  - ggpubr
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# organ-specific-malignant-cell-model-application

## Summary

Apply Census's organ-specific CCLE-trained cancer models to scRNA-seq tumor datasets to classify malignant epithelial cells and predict their likely cell of origin (e.g., hepatocellular vs cholangiocarcinoma in liver, ductal vs acinar-like states in pancreatic PDAC, enterocyte-lineage assignments in colon cancer).

## When to use

Use when tumor scRNA-seq data (Seurat object with malignant epithelial cells pre-identified) must be assigned a cancer cell of origin using Census organ-specific CCLE-trained models for Liver, Pancreas, or Large_Intestine (colon).

## When NOT to use

- The tissue organ has no Census cancer model — only Kidney, Large_Intestine, Liver, Lung, Mammary, Pancreas are available; other organs cannot be run with predict_cancer=T
- Malignant cells have not been pre-identified and the goal is general normal-tissue annotation — census_main without predict_cancer is the appropriate entry point
- The environment has Seurat v5 loaded — Census requires Seurat version 4 and will not run correctly

## Inputs

- Seurat v4 object containing scRNA-seq expression matrix restricted to previously identified malignant cells, with a UMAP dimensionality reduction
- Pretrained Census organ-specific CCLE-trained cancer models (Liver, Pancreas, Large_Intestine) from the Census library
- Six tumor scRNA-seq datasets: Massalha 2020, Ma 2021 (liver); Peng 2019, Hwang 2022 (pancreatic); Lee 2020, Pelka 2021 (colon)

## Outputs

- Per-cell terminal cell-of-origin calls: hepatocellular vs cholangiocarcinoma (liver), ductal vs acinar-like states (pancreatic), enterocyte-lineage assignments (colon)
- res$pred data frame with celltype and cancer_origin columns per cell
- cell_of_origin_predictions.csv with columns: cell barcode, dataset, organ, predicted cell of origin

## How to apply

Load Seurat v4 before the Census library (Census is incompatible with Seurat v5), then run the hierarchically organized gradient-boosted decision tree models with `predict_cancer = T`, passing the matching organ string ('Liver', 'Pancreas', 'Large_Intestine' for colon) so the organ-specific CCLE-trained cancer model is applied. Census starts at the root node of the cell-type hierarchy and successively classifies cells until terminal classifications are reached; afterwards the organ-specific model predicts whether classified epithelial cells are normal or malignant, with the normal cell-type retained as the predicted cell of origin. Apply the label-stabilizing step, which uses clusters and prediction contours in UMAP space to propagate higher-confidence binary prediction outcomes — so the input Seurat object must contain a UMAP reduction produced by the standard Seurat pipeline (NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunUMAP). Extract the `cancer_origin` field from `res$pred` for each malignant cell and compile per-cell predictions across the six tumor datasets (Massalha 2020, Ma 2021, Peng 2019, Hwang 2022, Lee 2020, Pelka 2021) into cell_of_origin_predictions.csv with cell barcode, dataset, organ, and predicted cell-of-origin columns. Validate calls against the original studies' clinical annotations, e.g., hepatocellular vs cholangiocarcinoma splits in liver.

## Related tools

- **Census** (Provides the hierarchically organized gradient-boosted decision tree classifiers and organ-specific CCLE-trained cancer models via census_main(predict_cancer = T), returning per-cell cancer_origin predictions) — https://github.com/sjdlabgroup/Census
- **Seurat 4** (Preprocessing (NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunUMAP) to produce the Seurat object with UMAP required by Census; must be loaded as version 4)
- **ggpubr** (Publication-ready ggplot2-based plotting support for visualizing Census prediction outputs) — https://github.com/kassambara/ggpubr

## Examples

```
res = census_main(obj, organ = 'Pancreas', predict_cancer = T)
```

## Evaluation signals

- Reproduced liver cell-of-origin calls correctly distinguish known hepatocellular versus cholangiocarcinoma cells, consistent with the original study's clinical annotations
- Pancreatic calls split into ductal vs acinar-like PDAC states and colon calls resolve to enterocyte-lineage assignments, matching the article's figures and findings
- cell_of_origin_predictions.csv contains all four required columns (cell barcode, dataset, organ, predicted cell of origin) and covers malignant cells from all six datasets
- cancer_origin values are populated only for cells classified as malignant epithelial, with normal cell-type retained as the predicted cell of origin otherwise
- Census predictions visually align with original author annotations when plotted in UMAP space (as in the Peng 2019 example)

## Limitations

- Census cancer models are only available for a subset of organs: Kidney, Large_Intestine, Liver, Lung, Mammary, Pancreas
- If more than one organ is specified in the organ parameter, only the first one is used for the cancer model
- Census requires Seurat version 4 specifically; loading Seurat v5 first will break execution
- The label-stabilizing algorithm depends on clusters and prediction contours in UMAP space, so a UMAP reduction is required and poor UMAP embeddings can affect label propagation

## Evidence

- [readme] Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R.: "Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R."
- [readme] an organ specific model predicts whether classified epithelial cells are normal or malignant, retaining the normal cell-type as the predicted cell of origin: "an organ specific model predicts whether classified epithelial cells are normal or malignant. The normal cell-type is also retained as the predicted cell of origin. Make sure to set the parameter"
- [readme] Currently available cancer models include: Kidney, Large_Intestine, Liver, Lung, Mammary, Pancreas.: [evidence span withheld — anchor status: external_doc]
- [intro] using clusters and prediction contours in UMAP space to propagate higher confidence binary prediction outcomes: "using clusters and prediction contours in UMAP space to propagate higher confidence binary prediction outcomes"
- [intro] The Census model starts at the root node of the cell-type hierarchy and successively classifies cells according to their node identities until terminal classifications are reached: "The Census model starts at the root node of the cell-type hierarchy and successively classifies cells according to their node identities until terminal classifications are reached"
