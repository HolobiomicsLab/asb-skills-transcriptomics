---
name: f1-score-benchmark-evaluation
description: Use when evaluating annotation or classification accuracy of single-cell
  labels — e.g. Census predictions vs original cluster-level labels — across perturbations
  such as cell-number down-sampling (5–500 cells per cluster), producing F1 distributions
  and non-parametric significance tests.
when_to_use_negative:
- The goal is method development or training of a new annotator rather than benchmarking
  an existing pretrained model's prediction accuracy
- Ground-truth reference labels per cell/cluster are unavailable, making F1 computation
  against original cluster-level labels impossible
- Comparing only two paired conditions where a paired test or direct per-cell concordance
  would be more appropriate than a Kruskal-Wallis test across many sampling levels
edam_operation: http://edamontology.org/operation_0474
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Census
  role: Pretrained hierarchical gradient-boosted decision tree annotator whose predictions
    are benchmarked via F1 scores against original cluster-level labels at each down-sampling
    level
  repo: https://github.com/sjdlabgroup/Census
- name: Seurat
  role: Standard Seurat pipeline (NormalizeData, ScaleData, FindVariableFeatures,
    RunPCA, RunUMAP) re-embeds each down-sampled dataset in UMAP space before annotation;
    Seurat v4 must be loaded before Census
- name: ggpubr
  role: Runs the Kruskal-Wallis non-parametric test over per-cell-type F1 score distributions
    across down-sampling levels and produces publication-ready comparison plots
  repo: https://github.com/kassambara/ggpubr
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad714
    title: Hierarchical and automated cell-type annotation and inference of cancer
      cell of origin with Census
  source_content_hash: sha256:7b330a8908073dea2cf0c1752e1e049d99bdf65bad3e38d175226ad079910bff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sjdlabgroup/Census
  tools:
  - Census
  - Seurat
  - ggpubr
  edam_operation: http://edamontology.org/operation_0474
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# f1-score-benchmark-evaluation

## Summary

Benchmark a cell-type annotation method (here, Census) by computing per-cell-type F1 scores of predicted labels against held-out reference labels, then testing whether score distributions differ across experimental conditions (e.g. down-sampling levels) with a Kruskal-Wallis test via ggpubr.

## When to use

Use when evaluating annotation or classification accuracy of single-cell labels — e.g. Census predictions vs original cluster-level labels — across perturbations such as cell-number down-sampling (5–500 cells per cluster), producing F1 distributions and non-parametric significance tests.

## When NOT to use

- The goal is method development or training of a new annotator rather than benchmarking an existing pretrained model's prediction accuracy
- Ground-truth reference labels per cell/cluster are unavailable, making F1 computation against original cluster-level labels impossible
- Comparing only two paired conditions where a paired test or direct per-cell concordance would be more appropriate than a Kruskal-Wallis test across many sampling levels

## Inputs

- Seurat object(s) with UMAP embeddings and original cluster/cell-type annotations (e.g. Peng et al. 2019 pancreatic cancer scRNA-seq data, peng_sub.RDS)
- Down-sampled subsets at each granularity level (5, 10, 20, 50, 100, 500 cells per cluster)
- Census annotation predictions (res$pred with celltype labels) for each subsampled dataset

## Outputs

- Per-cell-type F1 scores for each down-sampling level (downsampled_f1_scores.csv)
- Kruskal-Wallis test statistics and P-values comparing F1 score distributions across levels (e.g. P = 0.98 for >=20 cells/cell-type)
- Optional ggpubr box/violin plots of F1 score distributions with significance brackets

## How to apply

For each perturbation level (e.g. down-sampled 5, 10, 20, 50, 100, and 500 cells per UMAP cluster from the Peng et al. 2019 pancreatic cancer dataset), re-run the full annotation pipeline: re-embed the subsampled data in UMAP space with Seurat, then annotate with the pretrained Census hierarchical gradient-boosted decision tree model including the UMAP-based label-stabilizing step. Compare the resulting per-cell annotations to the original cluster-level reference labels and compute per-cell-type F1 scores at each level. Assess whether F1 score distributions differ across levels using a Kruskal-Wallis non-parametric test run with ggpubr; in the reference analysis, F1 scores showed no significant difference at 20 or more cells per cell-type (Kruskal-Wallis P = 0.98) but decreased below 20 cells per cell-type. Write per-level F1 scores and test statistics to an output table such as downsampled_f1_scores.csv.

## Related tools

- **Census** (Pretrained hierarchical gradient-boosted decision tree annotator whose predictions are benchmarked via F1 scores against original cluster-level labels at each down-sampling level) — https://github.com/sjdlabgroup/Census
- **Seurat** (Standard Seurat pipeline (NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunUMAP) re-embeds each down-sampled dataset in UMAP space before annotation; Seurat v4 must be loaded before Census)
- **ggpubr** (Runs the Kruskal-Wallis non-parametric test over per-cell-type F1 score distributions across down-sampling levels and produces publication-ready comparison plots) — https://github.com/kassambara/ggpubr

## Examples

```
res = census_main(obj, organ = 'Pancreas', predict_cancer = T); f1 <- compute_f1(res$pred, ref_labels); kruskal.test(f1 ~ level, data = downsampled_f1_scores)
```

## Evaluation signals

- Kruskal-Wallis test over F1 distributions for >=20 cells per cell-type reproduces the reported non-significant result (P = 0.98)
- F1 scores visibly decrease for down-sampling levels below 20 cells per cell-type relative to levels >=20
- downsampled_f1_scores.csv contains one F1 row per cell-type per down-sampling level (5, 10, 20, 50, 100, 500) plus test statistics
- Each subsampled dataset was re-embedded in UMAP space with Seurat before Census annotation, not annotated in the original embedding
- F1 scores are computed per cell type by comparing Census annotations to the original cluster-level labels

## Limitations

- Census requires Seurat version 4 to run and Seurat v4 must be loaded prior to loading the Census library in R
- Stable annotations require a minimum sample size: performance decreases below 20 cells per cell-type, so conclusions at very low sampling depths are unreliable
- Kruskal-Wallis is non-parametric and tests distributional differences only; it does not identify which levels differ without post-hoc pairwise comparisons

## Evidence

- [readme] Down-sampling sensitivity workflow: "Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R."
- [intro] Label-stabilizing annotation: "using clusters and prediction contours in UMAP space to propagate higher confidence binary prediction outcomes"
- [readme] Seurat processing pipeline: "obj = obj %>% CreateSeuratObject() %>% NormalizeData() %>% ScaleData() %>%
  FindVariableFeatures() %>% RunPCA() %>% RunUMAP(dims=1:50)"
- [results] Kruskal-Wallis result on F1 scores: [evidence span withheld — anchor status: not_found]
