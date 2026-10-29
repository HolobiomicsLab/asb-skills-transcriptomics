---
name: umap-re-embedding-of-subsampled-data
description: 'Use when assessing annotation robustness to sample size: after down-sampling
  cells (e.g., 5–500 per UMAP cluster), each subsampled dataset must be re-projected
  to a new UMAP before re-running cell-type prediction.'
when_to_use_negative:
- When the full original dataset is being annotated — use the original UMAP embedding
  and run census_main directly instead of re-embedding.
- When the goal is batch correction or integration across samples rather than cell-count
  sensitivity testing.
- When the annotation method used does not require a UMAP embedding as input.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: Runs the down-sampling and the re-processing/RunUMAP pipeline that produces
    the new UMAP embedding for each subsampled dataset; Seurat v4 must be loaded before
    Census.
  repo: https://github.com/sjdlabgroup/Census
- name: Census
  role: Consumes each re-embedded UMAP object via census_main (with label-stabilizing
    step using clusters and prediction contours in UMAP space) to produce cell-type
    annotations.
  repo: https://github.com/sjdlabgroup/Census
- name: ggpubr
  role: Used downstream for Kruskal-Wallis testing over F1 score distributions across
    down-sampling levels.
  repo: https://github.com/kassambara/ggpubr
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad714
    title: Hierarchical and automated cell-type annotation and inference of cancer
      cell of origin with Census
  source_content_hash: sha256:3bd0b7d6f2aaa881eeb5af35358894bbb5854caf39fddeb0cfba1650a5d5e98d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sjdlabgroup/Census
  tools:
  - Census
  - Seurat
  - ggpubr
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# UMAP re-embedding of subsampled data

## Summary

Re-embeds down-sampled single-cell RNA-seq datasets in UMAP space with Seurat so that a pretrained annotation model (e.g., Census) can be applied consistently across granularities when testing how cell-type predictions depend on cells-per-cluster count.

## When to use

Use when assessing annotation robustness to sample size: after down-sampling cells (e.g., 5–500 per UMAP cluster), each subsampled dataset must be re-projected to a new UMAP before re-running cell-type prediction.

## When NOT to use

- When the full original dataset is being annotated — use the original UMAP embedding and run census_main directly instead of re-embedding.
- When the goal is batch correction or integration across samples rather than cell-count sensitivity testing.
- When the annotation method used does not require a UMAP embedding as input.

## Inputs

- Seurat object with scRNA-seq counts and UMAP cluster annotations (e.g., Peng et al. 2019 pancreatic cancer dataset)
- List of down-sampling levels in cells per cluster (e.g., 5, 10, 20, 50, 100, 500)

## Outputs

- Re-embedded Seurat objects, one per down-sampling level, each with a new UMAP dimensionality reduction
- Census predictions (res$pred with celltype / cancer_origin) per re-embedded object, feeding per-cell-type F1 scores and Kruskal-Wallis statistics written to downsampled_f1_scores.csv

## How to apply

Load the scRNA-seq dataset (e.g., Peng et al. 2019 pancreatic cancer) with existing UMAP cluster annotations in R, loading Seurat v4 before the Census library as Census requires it. Iteratively down-sample to the target granularities (e.g., 5, 10, 20, 50, 100, 500 cells per UMAP cluster), then for each subsampled object re-run the standard Seurat pipeline — CreateSeuratObject, NormalizeData, ScaleData, FindVariableFeatures, RunPCA, and RunUMAP(dims=1:50) — to regenerate a UMAP dimensionality reduction on the reduced cell set. The rationale is that the annotation model (census_main with its label-stabilizing step using clusters and prediction contours in UMAP space) expects a UMAP embedding as input, so subsampled data must be freshly re-embedded rather than reusing the original coordinates. Feed each re-embedded object into Census prediction, then compare per-cell-type F1 scores (here against original cluster-level labels, with Kruskal-Wallis testing via ggpubr) to judge whether the subsampling preserves annotation quality.

## Related tools

- **Seurat** (Runs the down-sampling and the re-processing/RunUMAP pipeline that produces the new UMAP embedding for each subsampled dataset; Seurat v4 must be loaded before Census.) — https://github.com/sjdlabgroup/Census
- **Census** (Consumes each re-embedded UMAP object via census_main (with label-stabilizing step using clusters and prediction contours in UMAP space) to produce cell-type annotations.) — https://github.com/sjdlabgroup/Census
- **ggpubr** (Used downstream for Kruskal-Wallis testing over F1 score distributions across down-sampling levels.) — https://github.com/kassambara/ggpubr

## Examples

```
obj_sub = obj %>% CreateSeuratObject() %>% NormalizeData() %>% ScaleData() %>% FindVariableFeatures() %>% RunPCA() %>% RunUMAP(dims=1:50); res = census_main(obj_sub, organ = 'Pancreas')
```

## Evaluation signals

- Each subsampled Seurat object contains a valid UMAP reduction generated from RunPCA on the subsampled cell set, not reused original coordinates.
- Cell counts per cluster in each subsampled object match the target level (e.g., 20 cells per UMAP cluster for the >=20-cell threshold analysis).
- Census predictions run without error on each re-embedded object (Seurat v4 loaded before the Census library).
- Per-cell-type F1 scores computed against original cluster-level labels show no significant Kruskal-Wallis difference (P = 0.98) for levels >=20 cells per cell-type and degraded performance below 20 cells per cell-type.
- downsampled_f1_scores.csv contains F1 scores per down-sampling level plus test statistics.

## Limitations

- Census requires Seurat version 4 to run; Seurat v4 must be loaded prior to loading the Census library in R.
- The input Seurat object must contain a UMAP dimensionality reduction produced by the standard Seurat pipeline for Census prediction to work.
- Cancer prediction models are organ-specific (currently Kidney, Large_Intestine, Liver, Lung, Mammary, Pancreas); if multiple organs are specified, the first is used for the cancer model.

## Evidence

- [readme] Census requires Seurat v4 before loading: "Also note: Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R."
- [readme] Seurat pipeline with RunUMAP dims=1:50: "obj = obj %>% CreateSeuratObject() %>% NormalizeData() %>% ScaleData() %>%
  FindVariableFeatures() %>% RunPCA() %>% RunUMAP(dims=1:50)"
- [intro] Label-stabilizing step uses UMAP space: "using clusters and prediction contours in UMAP space to propagate higher confidence binary prediction outcomes"
- [readme] Census input requires a UMAP reduction: "The input data is a Seurat object that contains a UMAP dimensionality reduction."
