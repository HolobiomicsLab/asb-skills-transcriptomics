---
name: hierarchical-cell-type-classification
description: Use when annotating scRNA-seq data (Seurat v4 objects with a UMAP reduction)
  at the cell-type level, or when identifying malignant cells and predicting their
  likely cell of origin in normal or cancer tissues.
when_to_use_negative:
- The input is not a scRNA-seq dataset processed into a Seurat object with a UMAP
  reduction (e.g. bulk RNA-seq, feature tables without per-cell barcodes).
- The tissue organ is not among the 24 supported organs (e.g. Census pretrained model
  covers Bladder through Vasculature), and no custom model has been trained for the
  tissue of interest.
- An organ-specific cancer model does not exist for the tissue (only Kidney, Large_Intestine,
  Liver, Lung, Mammary, Pancreas cancer models are available).
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Census
  role: 'Main annotation tool: runs the hierarchically organized gradient-boosted
    decision tree models (census_main) for cell-type classification and cancer cell-of-origin
    prediction'
  repo: https://github.com/sjdlabgroup/Census
- name: Seurat 4
  role: 'Required preprocessing library: builds the Seurat object and generates the
    UMAP reduction that Census consumes; must be loaded before the Census library'
  repo: https://github.com/satijalab/seurat
- name: ggpubr
  role: ggplot2-based publication-ready plotting used to visualize Census annotations
  repo: https://github.com/kassambara/ggpubr
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad714
    title: Hierarchical and automated cell-type annotation and inference of cancer
      cell of origin with Census
  source_content_hash: sha256:e43123f6aa6bc4c1389ee391258c736caafa8fa4cd80ec50f42a3933695cea98
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sjdlabgroup/Census
  tools:
  - Census
  - Seurat 4
  - ggpubr
  - scHPL
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# hierarchical-cell-type-classification

## Summary

Apply Census, a fast and fully automated R method for single-cell RNA-seq cell-type annotation, which uses hierarchically organized gradient-boosted decision trees trained on Tabula Sapiens and CCLE to classify 175 cell-types from 24 organs and infer cancer cell of origin.

## When to use

Use when annotating scRNA-seq data (Seurat v4 objects with a UMAP reduction) at the cell-type level, or when identifying malignant cells and predicting their likely cell of origin in normal or cancer tissues.

## When NOT to use

- The input is not a scRNA-seq dataset processed into a Seurat object with a UMAP reduction (e.g. bulk RNA-seq, feature tables without per-cell barcodes).
- The tissue organ is not among the 24 supported organs (e.g. Census pretrained model covers Bladder through Vasculature), and no custom model has been trained for the tissue of interest.
- An organ-specific cancer model does not exist for the tissue (only Kidney, Large_Intestine, Liver, Lung, Mammary, Pancreas cancer models are available).

## Inputs

- Seurat v4 object with a UMAP dimensionality reduction (processed via NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunUMAP dims=1:50)
- Tissue organ specification (e.g. 'Lung', 'Pancreas') from the supported organ list
- predict_cancer flag (T/F) for cancer tissue annotation
- Tumor scRNA-seq expression matrices restricted to malignant cells (e.g. Massalha 2020, Peng 2019 datasets)

## Outputs

- Per-cell terminal cell-type predictions in res$pred (celltype, umap1, umap2 columns)
- Cancer cell-of-origin predictions (cancer_origin field) for malignant cells
- Compiled per-cell annotation tables, e.g. cell_of_origin_predictions.csv with cell barcode, dataset, organ, and predicted cell of origin columns

## How to apply

Load Seurat v4 before the Census library (Census requires Seurat version 4). Process the expression matrix with the standard Seurat pipeline (CreateSeuratObject, NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunUMAP dims=1:50) so the Seurat object contains a UMAP dimensionality reduction. Run census_main on the Seurat object specifying the tissue organ(s) from the supported list (Bladder through Vasculature); the model starts at the root node of the cell-type hierarchy and successively classifies cells according to their node identities until terminal classifications are reached. For cancer tissues, set predict_cancer=T so an organ-specific CCLE-trained model predicts whether classified epithelial cells are normal or malignant while retaining the normal cell-type as the predicted cell of origin; if multiple organs are given, the first one is used for the cancer model (available cancer models: Kidney, Large_Intestine, Liver, Lung, Mammary, Pancreas). New datasets are annotated with pretrained models followed by a label-stabilizing algorithm that uses clusters and prediction contours in UMAP space to propagate higher confidence binary prediction outcomes. Extract cell-type predictions (and cancer_origin calls) from res$pred for downstream tables and plotting.

## Related tools

- **Census** (Main annotation tool: runs the hierarchically organized gradient-boosted decision tree models (census_main) for cell-type classification and cancer cell-of-origin prediction) — https://github.com/sjdlabgroup/Census
- **Seurat 4** (Required preprocessing library: builds the Seurat object and generates the UMAP reduction that Census consumes; must be loaded before the Census library) — https://github.com/satijalab/seurat
- **ggpubr** (ggplot2-based publication-ready plotting used to visualize Census annotations) — https://github.com/kassambara/ggpubr

## Examples

```
res = census_main(obj, organ = 'Pancreas', predict_cancer = T)
```

## Evaluation signals

- Reproduced balanced accuracy against curated expert labels matches the reported benchmark (mean balanced accuracy of 0.8 on Tabula Muris mouse tissues across droplet and plate-based samples).
- Label similarity between Census annotations and reference labels matches reported values (label similarity 0.89 on Tabula Muris mouse tissues).
- Run time of annotation matches the reported scale (13 s across all Tabula Muris tissues).
- Cell-of-origin calls agree with clinical annotations, e.g. correct separation of hepatocellular versus cholangiocarcinoma cells in liver cancer datasets.
- Install sanity check: Census installs from GitHub main branch and requires Seurat v4 loaded before the Census library.

## Limitations

- Census requires Seurat version 4 to run; other Seurat versions are not supported and Seurat must be loaded before the Census library.
- The main pretrained model covers 175 cell-types from 24 organs; tissues outside this list require training a custom model with other references.
- Only six organ-specific cancer models are currently available (Kidney, Large_Intestine, Liver, Lung, Mammary, Pancreas).
- If more than one organ is specified in the organ parameter, only the first one is used for the cancer model.
- The input Seurat object must already contain a UMAP dimensionality reduction.

## Evidence

- [abstract] Census is a fast and fully automated cell-type identification method for single-cell RNA-seq data: "We developed Census, a biologically intuitive and fully automated cell-type identification method for single-cell RNA-seq data"
- [readme] Census requires Seurat version 4: "Note: a demo script (demo.r) and dataset (peng_sub.RDS) are also included in this directory. Also note: Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census"
- [intro] hierarchical gradient-boosted classification workflow: "The Census model starts at the root node of the cell-type hierarchy and successively classifies cells according to their node identities until terminal classifications are reached"
- [intro] label-stabilizing algorithm: "New datasets are annotated using the pretrained models followed by a custom-developed label-stabilizing algorithm (Fig. 1b). This involves using clusters and prediction contours in UMAP space"
- [readme] cancer prediction parameters: "Make sure to set the parameter `predict_cancer=T`. If more than one organ is specified in the `organ` parameter, the first one is used for the cancer model. Currently available cancer models include:"
- [readme] Seurat processing pipeline: "obj = obj %>% CreateSeuratObject() %>% NormalizeData() %>% ScaleData() %>% FindVariableFeatures() %>% RunPCA() %>% RunUMAP(dims=1:50)"
