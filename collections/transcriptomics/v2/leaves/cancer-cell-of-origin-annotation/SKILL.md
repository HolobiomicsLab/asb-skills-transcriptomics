---
name: cancer-cell-of-origin-annotation
description: Use when annotating epithelial cells in a cancer scRNA-seq dataset (e.g.,
  liver, pancreatic, colon, kidney, lung, mammary tumors) to identify malignant cells
  and infer their cell of origin from a Seurat object with a UMAP reduction.
when_to_use_negative:
- Input tissue is not one of the organs with a pretrained cancer model (only Kidney,
  Large_Intestine, Liver, Lung, Mammary, Pancreas are supported) — train a custom
  model instead
- Input is normal tissue rather than tumor data — use the standard census_main annotation
  without predict_cancer=T
- The Seurat object lacks a UMAP dimensionality reduction, which the Census prediction
  and label-stabilizing steps require
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: Census
  role: Provides the hierarchical gradient-boosted decision tree classifiers, organ-specific
    cancer models, and label-stabilizing algorithm used to call malignant cells and
    their cell of origin
  repo: https://github.com/sjdlabgroup/Census
- name: Seurat 4
  role: 'Required preprocessing framework: load Seurat v4 before the Census library
    to build the Seurat object, normalize/scale, and compute PCA/UMAP needed by Census'
- name: ggpubr
  role: ggplot2-based plotting used for publication-ready visualization of Census
    predictions
  repo: https://github.com/kassambara/ggpubr
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad714
    title: Hierarchical and automated cell-type annotation and inference of cancer
      cell of origin with Census
  source_content_hash: sha256:927a9e77d69991b228ebcc1bd092796edb8cb08eed0b1fd7856082bcf1d54727
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sjdlabgroup/Census
  tools:
  - Census
  - Seurat 4
  - ggpubr
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cancer-cell-of-origin-annotation

## Summary

Apply Census's organ-specific CCLE-trained cancer models to annotate malignant cells in scRNA-seq data and predict their likely cell of origin, using hierarchically organized gradient-boosted decision trees and a UMAP-space label-stabilizing algorithm.

## When to use

Use when annotating epithelial cells in a cancer scRNA-seq dataset (e.g., liver, pancreatic, colon, kidney, lung, mammary tumors) to identify malignant cells and infer their cell of origin from a Seurat object with a UMAP reduction.

## When NOT to use

- Input tissue is not one of the organs with a pretrained cancer model (only Kidney, Large_Intestine, Liver, Lung, Mammary, Pancreas are supported) — train a custom model instead
- Input is normal tissue rather than tumor data — use the standard census_main annotation without predict_cancer=T
- The Seurat object lacks a UMAP dimensionality reduction, which the Census prediction and label-stabilizing steps require

## Inputs

- Seurat v4 object containing a tumor scRNA-seq expression matrix (e.g., Peng 2019 PDAC, Massalha 2020, Ma 2021, Hwang 2022, Lee 2020, Pelka 2021) with a UMAP dimensionality reduction from the standard Seurat pipeline
- Organ name string matching an available cancer model: Kidney, Large_Intestine, Liver, Lung, Mammary, or Pancreas

## Outputs

- res$pred data frame with per-cell predictions including celltype and cancer_origin columns
- UMAP plots colored by Census cell-type annotation and by predicted cancer cell of origin
- Per-cell cell_of_origin predictions table (cell barcode, dataset, organ, predicted cell of origin), e.g., cell_of_origin_predictions.csv

## How to apply

Process the tumor scRNA-seq matrix through the standard Seurat pipeline (CreateSeuratObject, NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunUMAP(dims=1:50)), load Seurat v4 prior to the Census library, then call census_main with organ set to the tumor's tissue and predict_cancer=T, which first runs the pretrained Tabula Sapiens hierarchy (175 cell-types, 24 organs) from the root node to terminal classifications, then applies the organ-specific cancer model to classified epithelial cells. The cancer model — trained on the Cancer Cell Line Encyclopedia and available for Kidney, Large_Intestine, Liver, Lung, Mammary, and Pancreas — predicts malignant versus normal status and retains the normal cell-type as the predicted cell of origin (e.g., hepatocellular vs cholangiocarcinoma in liver, ductal vs acinar-like states in pancreatic). If multiple organs are given, the first one is used for the cancer model. Finally, run the label-stabilizing algorithm, which uses clusters and prediction contours in UMAP space to propagate higher-confidence binary prediction outcomes, and extract res$pred columns (celltype and cancer_origin) for per-cell output.

## Related tools

- **Census** (Provides the hierarchical gradient-boosted decision tree classifiers, organ-specific cancer models, and label-stabilizing algorithm used to call malignant cells and their cell of origin) — https://github.com/sjdlabgroup/Census
- **Seurat 4** (Required preprocessing framework: load Seurat v4 before the Census library to build the Seurat object, normalize/scale, and compute PCA/UMAP needed by Census)
- **ggpubr** (ggplot2-based plotting used for publication-ready visualization of Census predictions) — https://github.com/kassambara/ggpubr

## Examples

```
library(Seurat); library(Census); res = census_main(obj, organ = 'Liver', predict_cancer = T)
```

## Evaluation signals

- Reproduced cell-of-origin calls match the original study's clinical annotations, e.g., Census correctly distinguished hepatocellular versus cholangiocarcinoma cells in liver cancer datasets
- Pancreatic cancer cells resolve into ductal vs acinar-like states and colon cancer cells into enterocyte-lineage assignments, consistent with the paper's reported figures
- The res$pred output contains both celltype and cancer_origin columns populated for all epithelial cells when predict_cancer=T
- Label-stabilized predictions are spatially coherent in UMAP space (cells within a cluster/contour share the same propagated label)
- Benchmark behavior should reflect Census's reported outperformance of state-of-the-art annotation methods across 44 atlas-scale normal and cancer tissues

## Limitations

- Only one organ's cancer model can be applied per run — if more than one organ is specified in the organ parameter, the first one is used for the cancer model
- Cancer models are pretrained only for Kidney, Large_Intestine, Liver, Lung, Mammary, and Pancreas; other tissues require training a custom Census model
- Census requires Seurat version 4 specifically; loading a different Seurat version may break the library
- The pretrained model is trained on Tabula Sapiens and Cancer Cell Line Encyclopedia (175 cell-types, 24 organs), so cell types outside this reference space may not be captured

## Evidence

- [readme] Cancer models apply to classified epithelial cells: "the process is the same as for normal tissues, but afterwards an organ specific model predicts whether classified epithelial cells are normal or malignant. The normal cell-type is also retained as"
- [readme] Census requires Seurat v4: "Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R."
- [abstract] Hierarchical GBT classification mechanism: [evidence span withheld — anchor status: not_found]
- [intro] Label-stabilizing algorithm: "This involves using clusters and prediction contours in UMAP space to propagate higher confidence binary prediction outcomes."
- [abstract] Cell of origin inference validated: "When benchmarked on 44 atlas-scale normal and cancer, human and mouse tissues, Census significantly outperforms state-of-the-art methods across multiple metrics and naturally predicts the"
