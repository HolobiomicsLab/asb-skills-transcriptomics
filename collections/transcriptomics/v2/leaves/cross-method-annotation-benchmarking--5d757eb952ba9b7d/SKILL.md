---
name: cross-method-annotation-benchmarking
description: Use when a labeled scRNA-seq dataset with author/expert-provided cell-type
  annotations is available and you need to compare the prediction quality and speed
  of several automated annotation methods on identical inputs.
when_to_use_negative:
- The dataset has no reference/author cell-type annotations, so F1, balanced accuracy,
  and label similarity cannot be computed against ground truth.
- You only need a single method's annotations rather than a head-to-head comparison
  of multiple annotators.
- The tissue of interest is not covered by the available pretrained models (Census
  supports 24 organs and cancer models for Kidney, Large_Intestine, Liver, Lung, Mammary,
  Pancreas only).
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Census
  role: Hierarchical automated cell-type annotation method under evaluation; runs
    pretrained gradient-boosted decision tree models with label-stabilizing on UMAP
    clusters and prediction contours
  repo: https://github.com/sjdlabgroup/Census
- name: Seurat 4
  role: Required dependency (load Seurat v4 before the Census library); builds the
    Seurat object, normalizes, scales, and produces the UMAP reduction used by the
    annotators
  repo: https://zenodo.org/records/7017103
- name: ggpubr
  role: Publication-ready plotting/statistical annotation for visualizing per-method
    benchmark comparisons
  repo: https://github.com/kassambara/ggpubr
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad714
    title: Hierarchical and automated cell-type annotation and inference of cancer
      cell of origin with Census
  source_content_hash: sha256:cabd7d80d324644233a45f84d2de192ec557cedf119040fe72d53d8fbb4dde14
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sjdlabgroup/Census
  tools:
  - Census
  - Seurat 4
  - ggpubr
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cross-method-annotation-benchmarking

## Summary

Benchmark multiple automated scRNA-seq cell-type annotators (Census, scType, scATOMIC, scibet, scCATCH) on the same dataset by comparing each method's predicted labels against author-provided annotations using F1, balanced accuracy, label similarity, total accuracy, and runtime.

## When to use

Use when a labeled scRNA-seq dataset with author/expert-provided cell-type annotations is available and you need to compare the prediction quality and speed of several automated annotation methods on identical inputs.

## When NOT to use

- The dataset has no reference/author cell-type annotations, so F1, balanced accuracy, and label similarity cannot be computed against ground truth.
- You only need a single method's annotations rather than a head-to-head comparison of multiple annotators.
- The tissue of interest is not covered by the available pretrained models (Census supports 24 organs and cancer models for Kidney, Large_Intestine, Liver, Lung, Mammary, Pancreas only).

## Inputs

- scRNA-seq dataset as a Seurat v4 object with a UMAP dimensionality reduction (e.g., Peng et al. 2019 pancreatic cancer, 57,530 cells)
- Author-provided / expert cell-type annotation labels (ground truth)
- Pretrained reference models: Census Tabula Sapiens + Cancer Cell Line Encyclopedia model; scibet '30_major_human_cell_types' model

## Outputs

- Per-method predicted cell-type labels for every cell (Census returns res$pred with celltype and, for cancer tissue, cancer_origin)
- benchmark_table.csv with F1, balanced accuracy, label similarity, total accuracy, and runtime per method
- Statistical significance of between-method differences (Wilcoxon, P < 0.05)

## How to apply

Load the scRNA-seq dataset (e.g., the Peng et al. 2019 pancreatic cancer dataset, 57,530 cells) as a Seurat object, loading Seurat v4 before the Census library since Census requires Seurat v4. Process each object through the standard Seurat pipeline (NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunUMAP dims=1:50) so it contains a UMAP reduction, then run Census hierarchical annotation with the pretrained Tabula Sapiens + Cancer Cell Line Encyclopedia gradient-boosted decision tree model (census_main with organ='Pancreas', predict_cancer=T for tumor tissue), including its label-stabilizing step that uses clusters and prediction contours in UMAP space. Run each baseline annotator with its specified settings (e.g., scibet with the '30_major_human_cell_types' pretrained model; scType with pancreas tissue plus immune cell reference selection). Compute F1 score, balanced accuracy, label similarity, overall accuracy, and per-method runtime for each method against the author-provided cell-type annotations, then assemble the metrics into a per-method comparison table (benchmark_table.csv). Judge performance statistically — in the source benchmark, Census had the highest mean F1 and balanced accuracy, with significant differences from other methods assessed by Wilcoxon tests (P < 0.05).

## Related tools

- **Census** (Hierarchical automated cell-type annotation method under evaluation; runs pretrained gradient-boosted decision tree models with label-stabilizing on UMAP clusters and prediction contours) — https://github.com/sjdlabgroup/Census
- **Seurat 4** (Required dependency (load Seurat v4 before the Census library); builds the Seurat object, normalizes, scales, and produces the UMAP reduction used by the annotators) — https://zenodo.org/records/7017103
- **ggpubr** (Publication-ready plotting/statistical annotation for visualizing per-method benchmark comparisons) — https://github.com/kassambara/ggpubr

## Examples

```
library(Seurat); library(Census); obj = obj %>% NormalizeData() %>% ScaleData() %>% FindVariableFeatures() %>% RunPCA() %>% RunUMAP(dims=1:50); res = census_main(obj, organ = 'Pancreas', predict_cancer = T)
```

## Evaluation signals

- benchmark_table.csv contains rows for all five methods (Census, scType, scATOMIC, scibet, scCATCH) with F1, balanced accuracy, label similarity, accuracy, and runtime columns populated.
- Census metrics are consistent with the reported benchmark: higher mean F1 and balanced accuracy than the second-place method.
- Between-method score differences are statistically significant by Wilcoxon test with P < 0.05 where reported.
- Each method was run on the identical Seurat-processed object (same UMAP reduction) so runtime and label comparisons are apples-to-apples.
- Census predictions on cancer tissue include cancer_origin labels for malignant epithelial cells when predict_cancer = T.

## Limitations

- Census requires Seurat version 4; loading a different Seurat version or the wrong load order will break the workflow.
- Census cancer models are limited to Kidney, Large_Intestine, Liver, Lung, Mammary, and Pancreas; the organ parameter's first entry is used for the cancer model.
- The Census main pretrained model covers 175 cell-types from 24 organs; cell types outside this reference require training a custom model.

## Evidence

- [results] Benchmark design uses author annotations as ground truth: [evidence span withheld — anchor status: not_found]
- [readme] Seurat v4 dependency: "Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R."
- [intro] Label-stabilizing step: "New datasets are annotated using the pretrained models followed by a custom-developed label-stabilizing algorithm (Fig. 1b). This involves using clusters and prediction contours in UMAP space"
- [intro] Pretrained model scope: "The main Census model is trained on the Tabula Sapiens and Cancer Cell Line Encyclopedia and can identify 175 cell-types from 24 organs"
- [readme] Census invocation for pancreatic cancer: "res = census_main(obj, organ = 'Pancreas', predict_cancer = T)"
- [abstract] Benchmarked performance claim: "When benchmarked on 44 atlas-scale normal and cancer, human and mouse tissues, Census significantly outperforms state-of-the-art methods across multiple metrics"
