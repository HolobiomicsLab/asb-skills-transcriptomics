---
name: hierarchical-gradient-boosted-tree-classification
description: Use when annotating cell types in a Seurat scRNA-seq object (with UMAP
  computed) for human or mouse tissues, or when identifying malignant epithelial cells
  and their cancer cell-of-origin, without manual marker curation.
when_to_use_negative:
- Input is not a Seurat object or lacks a UMAP dimensionality reduction (Census requires
  Seurat v4 and a UMAP; load Seurat v4 prior to the Census library)
- Tissue organ is not among the 24 supported Tabula Sapiens organs, or a cancer model
  is needed for an organ outside the six available (Kidney, Large_Intestine, Liver,
  Lung, Mammary, Pancreas) — train a custom model instead
- Cell types of interest fall outside the 175 cell-types/24 organs covered by the
  pretrained model and no custom reference is available
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Census
  role: Provides the hierarchical gradient-boosted decision tree classifier (census_main)
    with pretrained Tabula Sapiens + CCLE models and the label-stabilizing algorithm
  repo: https://github.com/sjdlabgroup/Census
- name: Seurat
  role: 'Seurat v4 supplies the required input object: standard scRNA-seq processing
    pipeline and the UMAP reduction Census consumes'
  repo: https://github.com/sjdlabgroup/Census
- name: ggpubr
  role: R plotting package cited in the article for publication-ready visualization
    of results
  repo: https://github.com/kassambara/ggpubr
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad714
    title: Hierarchical and automated cell-type annotation and inference of cancer
      cell of origin with Census
  source_content_hash: sha256:ba517d801c4ae031777467e913eb1596e63c3c0d534b252df6075fd893f72bf2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sjdlabgroup/Census
  tools:
  - Census
  - Seurat
  - ggpubr
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# hierarchical-gradient-boosted-tree-classification

## Summary

Census performs fully automated cell-type annotation of scRNA-seq data using hierarchically organized gradient-boosted decision tree models that successively classify cells from the root node to terminal cell types, followed by a UMAP-space label-stabilizing step.

## When to use

Use when annotating cell types in a Seurat scRNA-seq object (with UMAP computed) for human or mouse tissues, or when identifying malignant epithelial cells and their cancer cell-of-origin, without manual marker curation.

## When NOT to use

- Input is not a Seurat object or lacks a UMAP dimensionality reduction (Census requires Seurat v4 and a UMAP; load Seurat v4 prior to the Census library)
- Tissue organ is not among the 24 supported Tabula Sapiens organs, or a cancer model is needed for an organ outside the six available (Kidney, Large_Intestine, Liver, Lung, Mammary, Pancreas) — train a custom model instead
- Cell types of interest fall outside the 175 cell-types/24 organs covered by the pretrained model and no custom reference is available

## Inputs

- Seurat v4 object with a UMAP dimensionality reduction (processed via NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunUMAP dims=1:50)
- Organ specification (one or more of 24 organs, e.g. Bladder, Blood, Bone_Marrow, Eye, Fat, Heart, Kidney, Large_Intestine, Liver, Lung, Lymph_Node, Mammary, Muscle, Pancreas, Skin, ...)
- Optional predict_cancer flag for tumor datasets (cancer models: Kidney, Large_Intestine, Liver, Lung, Mammary, Pancreas)

## Outputs

- Per-cell predicted cell-type labels (res$pred$celltype)
- For cancer runs: predicted cancer_origin (normal cell type of origin) for malignant epithelial cells
- UMAP-space plots of annotations (celltype or cancer_origin colored UMAP via ggplot2/ggrepel)

## How to apply

First process the data with the standard Seurat pipeline (CreateSeuratObject, NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunUMAP dims=1:50), since Census requires a Seurat v4 object containing a UMAP dimensionality reduction. Load Seurat v4 before the Census library, then call census_main with the appropriate organ (e.g. 'Lung', 'Pancreas' from the 24 supported organs) to run the pretrained Tabula Sapiens model covering 175 cell-types. The model starts at the root node of the cell-type hierarchy and successively classifies cells according to their node identities until terminal classifications are reached using gradient-boosted decision trees. For cancer tissues, set predict_cancer=T so an organ-specific model flags classified epithelial cells as normal or malignant and retains the normal cell-type as the predicted cell of origin (cancer models available for Kidney, Large_Intestine, Liver, Lung, Mammary, Pancreas). A label-stabilizing algorithm using clusters and prediction contours in UMAP space propagates higher confidence binary prediction outcomes; results are returned in res$pred with celltype and cancer_origin columns for plotting.

## Related tools

- **Census** (Provides the hierarchical gradient-boosted decision tree classifier (census_main) with pretrained Tabula Sapiens + CCLE models and the label-stabilizing algorithm) — https://github.com/sjdlabgroup/Census
- **Seurat** (Seurat v4 supplies the required input object: standard scRNA-seq processing pipeline and the UMAP reduction Census consumes) — https://github.com/sjdlabgroup/Census
- **ggpubr** (R plotting package cited in the article for publication-ready visualization of results) — https://github.com/kassambara/ggpubr

## Examples

```
library(Census); res = census_main(obj, organ = 'Pancreas', predict_cancer = T)
```

## Evaluation signals

- res$pred contains non-empty celltype labels for all cells, with predicted types drawn from the 175 Tabula Sapiens cell-types (plus 'Cancer' labels when predict_cancer=T)
- Cancer runs return a cancer_origin value for malignant epithelial cells, traceable to a normal pancreatic/organ cell type
- Predicted labels cluster coherently in UMAP space, consistent with the label-stabilizing algorithm propagating high-confidence binary outcomes across cluster contours
- Benchmark behavior: on the Peng et al. 2019 pancreatic cancer dataset (57,530 cells), Census achieves higher mean F1 and balanced accuracy than scType, scATOMIC, scibet, and scCATCH (Wilcoxon, P < 0.05)
- Runtime remains fast: benchmark table includes per-method runtime, with Census noted for high prediction speed

## Limitations

- Census requires Seurat version 4; load Seurat v4 prior to loading the Census library in R
- If more than one organ is specified in the organ parameter, only the first one is used for the cancer model
- The main pretrained model covers 175 cell-types from 24 organs; tissues/cell types outside this scope require training a custom model
- Cancer prediction currently limited to six organ models (Kidney, Large_Intestine, Liver, Lung, Mammary, Pancreas)

## Evidence

- [abstract] Census infers hierarchical cell-type relationships and uses gradient-boosted decision trees that capitalize on nodal cell-type relationships to achieve high prediction speed and accuracy.: "Census infers hierarchical cell-type relationships and uses gradient-boosted \decision trees that capitalize on nodal cell-type relationships to achieve high prediction speed and accuracy."
- [intro] The Census model starts at the root node of the cell-type hierarchy and successively classifies cells according to their node identities until terminal classifications are reached.: "The Census model starts at the root node of the cell-type hierarchy and successively classifies cells according to their node identities until terminal classifications are reached"
- [readme] Census requires Seurat version 4; load Seurat v4 before the Census library.: "Also note: Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R."
- [readme] Label-stabilizing algorithm uses clusters and prediction contours in UMAP space.: "New datasets are annotated using the pretrained models followed by a custom-developed label-stabilizing algorithm (Fig. 1b). This involves using clusters and prediction contours in UMAP space"
- [readme] Cancer annotation with predict_cancer=T retains normal cell type as predicted cell of origin.: "an organ specific model predicts whether classified epithelial cells are normal or malignant. The normal cell-type is also retained as the predicted cell of origin. Make sure to set the parameter"
