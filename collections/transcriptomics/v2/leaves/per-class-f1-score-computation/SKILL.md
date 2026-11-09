---
name: per-class-f1-score-computation
description: Use when you have predicted cell-type annotations from a tool like Census
  plus reference labels for the same cells and need per-class F1 scores, or need to
  test annotation robustness across embedding parameter grids.
when_to_use_negative:
- There is no ground-truth reference annotation for the dataset, so no F1 can be computed
  against a reference
- The question is overall annotation accuracy across a benchmark of many tissues rather
  than robustness of per-class scores to embedding parameters
- Input predictions are probabilistic scores rather than discrete cell-type labels
  without an additional argmax/thresholding step
tools:
- name: Census
  role: Produces the predicted per-cell celltype annotations (and UMAP-space label
    stabilization) that are scored against reference labels
  repo: https://github.com/sjdlabgroup/Census
- name: Seurat 4
  role: Builds the Seurat object, recomputes UMAP embeddings across the n.neighbors/min.dist
    grid (RunUMAP), and is required by Census
  repo: https://zenodo.org/records/7017103
- name: ggpubr
  role: Provides kruskal.test-based non-parametric testing of per-cell-type F1 scores
    across hyperparameter settings
  repo: https://github.com/kassambara/ggpubr
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad714
    title: Hierarchical and automated cell-type annotation and inference of cancer
      cell of origin with Census
  source_content_hash: sha256:ecf9d7a8e7e2acee85171aaef6f71c81f4ae7b7a0158ab7624d2b7a9e5c43676
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sjdlabgroup/Census
  tools:
  - Census
  - Seurat 4
  - ggpubr
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# per-class-f1-score-computation

## Summary

Computes per-cell-type F1 scores by comparing an automated annotation method's predictions (e.g., Census cell-type calls from UMAP-embedded scRNA-seq data) against reference labels, and tests whether scores are stable across conditions (e.g., UMAP n.neighbors and min.dist settings) with Kruskal-Wallis tests.

## When to use

Use when you have predicted cell-type annotations from a tool like Census plus reference labels for the same cells and need per-class F1 scores, or need to test annotation robustness across embedding parameter grids.

## When NOT to use

- There is no ground-truth reference annotation for the dataset, so no F1 can be computed against a reference
- The question is overall annotation accuracy across a benchmark of many tissues rather than robustness of per-class scores to embedding parameters
- Input predictions are probabilistic scores rather than discrete cell-type labels without an additional argmax/thresholding step

## Inputs

- Seurat v4 object with a UMAP dimensionality reduction and processed scRNA-seq counts (e.g., Peng et al. 2019 pancreatic cancer count matrix)
- Census prediction results (res$pred data frame with per-cell celltype labels)
- Reference cell-type labels per cell (ground-truth annotations)
- A grid of UMAP hyperparameter settings (n.neighbors, min.dist) defining embedding variants

## Outputs

- Per-cell-type F1 score table (one row per cell type per condition)
- Kruskal-Wallis test statistics and P-values across parameter settings
- sensitivity_analysis_results.csv

## How to apply

For each cell, compare the predicted celltype column (returned by census_main on a Seurat v4 object with a UMAP reduction, e.g., organ = 'Pancreas') against the dataset's reference cell-type labels, computing F1 separately for each cell type so rare classes are not masked by majority classes. Build one F1 table per condition — in the source task, one per UMAP embedding variant across a grid of n.neighbors and min.dist values — because Census's label-stabilizing step operates on clusters and prediction contours in UMAP space, making F1 sensitive to the embedding. Stack per-condition F1 tables and run a Kruskal-Wallis non-parametric test over per-cell-type F1 scores across conditions (kruskal.test, exposed via the ggpubr package), which avoids distributional assumptions. In the source study this reproduced P = 0.99 for n.neighbors and P = 0.89 for min.dist, indicating Census F1 scores were robust to UMAP hyperparameters on the Peng et al. 2019 pancreatic cancer dataset. Write the F1 table together with Kruskal-Wallis statistics and P-values to sensitivity_analysis_results.csv.

## Related tools

- **Census** (Produces the predicted per-cell celltype annotations (and UMAP-space label stabilization) that are scored against reference labels) — https://github.com/sjdlabgroup/Census
- **Seurat 4** (Builds the Seurat object, recomputes UMAP embeddings across the n.neighbors/min.dist grid (RunUMAP), and is required by Census) — https://zenodo.org/records/7017103
- **ggpubr** (Provides kruskal.test-based non-parametric testing of per-cell-type F1 scores across hyperparameter settings) — https://github.com/kassambara/ggpubr

## Examples

```
library(Seurat); library(Census); obj <- obj %>% NormalizeData() %>% ScaleData() %>% FindVariableFeatures() %>% RunPCA() %>% RunUMAP(dims=1:50); res <- census_main(obj, organ='Pancreas', predict_cancer=T)
```

## Evaluation signals

- All per-class F1 scores lie in [0, 1]; per-class F1 equals the harmonic mean of precision and recall computed only on that cell type's cells
- Kruskal-Wallis P-values reproduce the reported values (P = 0.99 for n.neighbors, P = 0.89 for min.dist) on the Peng et al. 2019 pancreatic cancer dataset
- Number of F1 rows equals number of cell types x number of UMAP parameter combinations; each cell in the reference labels is scored exactly once per condition
- sensitivity_analysis_results.csv contains both the F1 table and the Kruskal-Wallis statistics with P-values
- Census was run with Seurat v4 loaded first, and each embedding variant was regenerated with Seurat's RunUMAP before annotation

## Limitations

- Census requires Seurat version 4 and must be loaded after Seurat v4 in R; using Seurat v5 breaks the workflow
- F1 scores depend on Census's label-stabilizing algorithm, which uses clusters and prediction contours in UMAP space, so results are not independent of the embedding; the Peng dataset supports only the pancreas cancer model among available cancer models (Kidney, Large_Intestine, Liver, Lung, Mammary, Pancreas)
- Kruskal-Wallis P-values reflect stability of per-class F1 across the tested hyperparameter grid, not annotation correctness; both P = 0.99 and P = 0.89 are far from significance, so tiny grids or heterogeneous cell-type counts could mask real variation

## Evidence

- [intro] label-stabilizing step depends on UMAP space: "New datasets are annotated using the pretrained models followed by a custom-developed label-stabilizing algorithm (Fig. 1b). This involves using clusters and prediction contours in UMAP space"
- [readme] Seurat v4 load order requirement: "Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R."
- [abstract] benchmarking uses multiple performance metrics: "When benchmarked on 44 atlas-scale normal and cancer, human and mouse tissues, Census significantly outperforms state-of-the-art methods across multiple metrics and naturally predicts the"
- [readme] concrete Census invocation on the pancreatic cancer dataset: "res = census_main(obj, organ = 'Pancreas', predict_cancer = T)"
