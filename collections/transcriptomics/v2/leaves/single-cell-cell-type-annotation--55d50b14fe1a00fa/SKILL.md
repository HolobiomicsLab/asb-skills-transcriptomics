---
name: single-cell-cell-type-annotation
description: Use when a raw scRNA-seq count matrix (Seurat object) needs fully automated
  cell-type labels, or when cancer datasets need malignant cell identification and
  cell-of-origin inference, without manual marker curation.
when_to_use_negative:
- The input is not a scRNA-seq dataset processed into a Seurat object with a UMAP
  reduction (Census's label stabilization requires UMAP clusters and prediction contours).
- The tissue organ falls outside the 24 supported organs and no custom reference model
  can be trained (pretrained organ and cancer models cover a fixed list).
- The task requires the shortest runtime over annotation quality — scType had shorter
  run times than Census in the 44-tissue benchmark.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Census
  role: 'Performs the hierarchical cell-type annotation: gradient-boosted decision
    trees over the cell-type hierarchy plus UMAP-space label stabilization; installed
    via remotes::install_github(''sjdlabgroup/Census'', ref=''main'')'
  repo: https://github.com/sjdlabgroup/Census
- name: Seurat 4
  role: 'Required preprocessing framework: must be loaded before the Census library;
    builds the Seurat object, runs NormalizeData/ScaleData/FindVariableFeatures/RunPCA/RunUMAP
    to produce the UMAP reduction Census operates on'
- name: ggpubr
  role: Used for statistical comparisons of annotation quality metrics across methods
    or UMAP parameter settings (Wilcoxon signed-rank, Kruskal-Wallis via kruskal.test)
  repo: https://github.com/kassambara/ggpubr
provenance:
  source_task_ids:
  - task_007
  - task_004
  - task_003
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad714
    title: Hierarchical and automated cell-type annotation and inference of cancer
      cell of origin with Census
  source_content_hash: sha256:c0345cee08f1e285a72468d2c1e52fd47066774af37121f12d3f3047ec42d56b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sjdlabgroup/Census
  tools:
  - Census
  - Seurat 4
  - ggpubr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# single-cell-cell-type-annotation

## Summary

Automated hierarchical cell-type annotation of scRNA-seq data with Census, which applies gradient-boosted decision tree models organized along a cell-type hierarchy, followed by label stabilization in UMAP space; the pretrained Tabula Sapiens + CCLE model classifies 175 cell-types from 24 organs and can flag malignant cells and their cell of origin.

## When to use

Use when a raw scRNA-seq count matrix (Seurat object) needs fully automated cell-type labels, or when cancer datasets need malignant cell identification and cell-of-origin inference, without manual marker curation.

## When NOT to use

- The input is not a scRNA-seq dataset processed into a Seurat object with a UMAP reduction (Census's label stabilization requires UMAP clusters and prediction contours).
- The tissue organ falls outside the 24 supported organs and no custom reference model can be trained (pretrained organ and cancer models cover a fixed list).
- The task requires the shortest runtime over annotation quality — scType had shorter run times than Census in the 44-tissue benchmark.

## Inputs

- Seurat v4 object with a UMAP dimensionality reduction (built from a scRNA-seq count matrix, e.g. Peng et al. 2019 pancreatic cancer, 57,530 cells)
- organ parameter string (one or more of the 24 supported organs: Bladder, Blood, Bone_Marrow, Eye, Fat, Heart, Kidney, Large_Intestine, Liver, Lung, Lymph_Node, Mammary, Muscle, Pancreas, Prostate, Salivary_Gland, Skin, Small_Intestine, Spleen, Thymus, Tongue, Trachea, Uterus, Vasculature)
- predict_cancer logical flag (T for cancer tissues; currently available cancer models: Kidney, Large_Intestine, Liver, Lung, Mammary, Pancreas)
- optional: reference labeled dataset + custom model for training instead of the pretrained Tabula Sapiens model

## Outputs

- res$pred data frame with per-cell predicted celltype, umap1/umap2 coordinates, and cancer_origin when predict_cancer=T
- benchmark metric tables comparing predictions against reference labels (F1, balanced accuracy, label similarity, overall accuracy, runtime), e.g. benchmark_table.csv, benchmark_metrics.csv, sensitivity_analysis_results.csv, mouse_evaluation_results.csv

## How to apply

Load Seurat v4 before the Census library in R (Census requires Seurat v4), then process the data through the standard Seurat pipeline (NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunUMAP dims=1:50) so a UMAP reduction exists — Census's label-stabilizing step depends on clusters and prediction contours in UMAP space. Run census_main on the Seurat object, specifying the tissue organ (e.g. 'Lung' or 'Pancreas' from the 24 supported organs); for cancer tissues set predict_cancer=T so an organ-specific model classifies epithelial cells as normal vs malignant and retains the normal type as the predicted cell of origin. Census starts at the root node of the cell-type hierarchy and successively classifies cells with hierarchically organized gradient-boosted decision trees until terminal classifications are reached, then propagates higher-confidence binary prediction outcomes via the UMAP-space label-stabilizing algorithm. The result (res$pred) contains per-cell celltype (and cancer_origin for cancer mode), which can be plotted on UMAP coordinates and compared against reference labels using F1, balanced accuracy, label similarity, and overall accuracy.

## Related tools

- **Census** (Performs the hierarchical cell-type annotation: gradient-boosted decision trees over the cell-type hierarchy plus UMAP-space label stabilization; installed via remotes::install_github('sjdlabgroup/Census', ref='main')) — https://github.com/sjdlabgroup/Census
- **Seurat 4** (Required preprocessing framework: must be loaded before the Census library; builds the Seurat object, runs NormalizeData/ScaleData/FindVariableFeatures/RunPCA/RunUMAP to produce the UMAP reduction Census operates on)
- **ggpubr** (Used for statistical comparisons of annotation quality metrics across methods or UMAP parameter settings (Wilcoxon signed-rank, Kruskal-Wallis via kruskal.test)) — https://github.com/kassambara/ggpubr

## Examples

```
res = census_main(obj, organ = 'Pancreas', predict_cancer = T)
```

## Evaluation signals

- Predictions agree with reference/author-provided cell-type labels at benchmark-level F1 and balanced accuracy — in the 44-dataset benchmark (1,769,071 cells, 105 harmonized labels) Census significantly outperformed scType (all Wilcoxon P < 2e-16) and identified 81/105 subtypes.
- On Tabula Muris mouse tissues the reproduced mean balanced accuracy, label similarity, and run time match the reported values of 0.8, 0.89, and 13 s within rerun tolerance.
- Reported throughput of ~13,000 cells/min is achieved on held-out cells.
- Census annotations are stable to embedding perturbation: varying UMAP n.neighbors and min.dist on the Peng et al. 2019 pancreatic cancer data leaves cell-type F1 scores unchanged (Kruskal-Wallis P = 0.99 and P = 0.89).
- res$pred contains a non-NULL celltype column for every cell and, when predict_cancer=T, a cancer_origin value for cells classified as malignant.

## Limitations

- Census requires Seurat version 4 to run; load Seurat v4 prior to loading the Census library in R.
- The input Seurat object must already contain a UMAP dimensionality reduction produced by the standard Seurat pipeline.
- Cancer prediction is organ-specific — only Kidney, Large_Intestine, Liver, Lung, Mammary, and Pancreas cancer models are currently available, and if more than one organ is specified, only the first is used for the cancer model.
- scType had shorter run times than Census in the 44-dataset benchmark, so Census trades runtime for prediction quality.

## Evidence

- [readme] Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R.: "Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R."
- [intro] Census implements hierarchically organized gradient-boosted decision trees that successively classify cells.: "Census implements a collection of hierarchically organized gradient-boosted decision tree models that successively classify individual cells according to a predefined cell-type hierarchy"
- [intro] Label stabilization uses clusters and prediction contours in UMAP space.: "using clusters and prediction contours in UMAP space to propagate higher confidence binary prediction outcomes"
- [intro] The main Census model is trained on Tabula Sapiens and CCLE and identifies 175 cell-types from 24 organs.: "The main Census model is trained on the Tabula Sapiens and Cancer Cell Line Encyclopedia and can identify 175 cell-types from 24 organs"
- [readme] Cancer tissues are annotated with predict_cancer=T; the normal cell-type is retained as the predicted cell of origin.: "an organ specific model predicts whether classified epithelial cells are normal or malignant. The normal cell-type is also retained as the predicted cell of origin. Make sure to set the parameter"
- [abstract] Benchmark on 44 atlas-scale tissues shows Census significantly outperforms state-of-the-art methods.: "When benchmarked on 44 atlas-scale normal and cancer, human and mouse tissues, Census significantly outperforms state-of-the-art methods across multiple metrics and naturally predicts the"
