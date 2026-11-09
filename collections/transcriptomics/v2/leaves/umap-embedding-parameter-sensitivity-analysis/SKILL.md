---
name: umap-embedding-parameter-sensitivity-analysis
description: Use when evaluating whether cell-type annotations from an embedding-dependent
  annotation method (e.g. Census, whose label-stabilizing step uses UMAP clusters
  and prediction contours) remain stable across UMAP hyperparameter variations like
  n.neighbors and min.dist.
when_to_use_negative:
- The cell-type annotation method under test is not embedding-dependent (e.g. purely
  transcriptome-based classifiers), so UMAP hyperparameters cannot plausibly affect
  its output.
- The dataset lacks reference cell-type labels, making per-cell-type F1 score computation
  against ground truth impossible.
- The goal is to tune UMAP parameters for optimal visualization rather than to assess
  annotation robustness across parameters.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Census
  role: Annotates cell types on each UMAP embedding variant using its label-stabilizing
    algorithm on clusters and prediction contours; main model trained on Tabula Sapiens
    and Cancer Cell Line Encyclopedia.
  repo: https://github.com/sjdlabgroup/Census
- name: Seurat
  role: Builds the Seurat v4 object, runs the standard preprocessing pipeline (NormalizeData,
    ScaleData, FindVariableFeatures, RunPCA), and recomputes UMAP embeddings via RunUMAP
    for each hyperparameter combination.
  repo: https://github.com/sjdlabgroup/Census
- name: ggpubr
  role: Provides kruskal.test to run Kruskal-Wallis non-parametric tests over per-cell-type
    F1 scores across UMAP parameter settings.
  repo: https://github.com/kassambara/ggpubr
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad714
    title: Hierarchical and automated cell-type annotation and inference of cancer
      cell of origin with Census
  source_content_hash: sha256:204481c72738fccf35a63009d7bdee9c7d265a912f8307fa72ec16e4b8593ba8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sjdlabgroup/Census
  tools:
  - Census
  - Seurat
  - ggpubr
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# umap-embedding-parameter-sensitivity-analysis

## Summary

Tests whether Census cell-type annotations are robust to UMAP embedding choices by re-annotating a scRNA-seq dataset across a grid of n.neighbors and min.dist settings and statistically comparing per-cell-type F1 scores with Kruskal-Wallis tests. This directly probes the sensitivity of Census's label-stabilizing algorithm, which depends on UMAP-space clusters and prediction contours.

## When to use

Use when evaluating whether cell-type annotations from an embedding-dependent annotation method (e.g. Census, whose label-stabilizing step uses UMAP clusters and prediction contours) remain stable across UMAP hyperparameter variations like n.neighbors and min.dist.

## When NOT to use

- The cell-type annotation method under test is not embedding-dependent (e.g. purely transcriptome-based classifiers), so UMAP hyperparameters cannot plausibly affect its output.
- The dataset lacks reference cell-type labels, making per-cell-type F1 score computation against ground truth impossible.
- The goal is to tune UMAP parameters for optimal visualization rather than to assess annotation robustness across parameters.

## Inputs

- scRNA-seq count matrix (e.g. Peng et al. 2019 pancreatic cancer dataset, or a Seurat object / .RDS such as peng_sub.RDS)
- Reference cell-type labels for the dataset (ground-truth annotations for F1 computation)
- Grid of UMAP hyperparameter values for n.neighbors and min.dist

## Outputs

- Per-cell-type F1 scores of Census annotations for each UMAP parameter setting
- Kruskal-Wallis test statistics and P-values for F1 scores across n.neighbors and min.dist settings
- sensitivity_analysis_results.csv containing F1 scores and test statistics/P-values

## How to apply

Load the scRNA-seq count matrix (e.g. Peng et al. 2019 pancreatic cancer data) into R and build a Seurat v4 object, loading Seurat v4 before the Census library since Census requires Seurat version 4. Recompute UMAP embeddings with Seurat's RunUMAP across a grid of n.neighbors and min.dist values, producing one embedding variant per parameter combination, after the standard Seurat preprocessing (NormalizeData, ScaleData, FindVariableFeatures, RunPCA). Run census_main on each embedding variant (with organ specified, e.g. 'Pancreas', and predict_cancer=T for tumor data) so the label-stabilizing algorithm operates on clusters and prediction contours in each UMAP space. Compare each variant's annotations against reference cell-type labels to compute per-cell-type F1 scores, then run Kruskal-Wallis non-parametric tests over F1 scores across settings using kruskal.test via ggpubr. Reproduction of the reported result means non-significant tests (P = 0.99 for n.neighbors and P = 0.89 for min.dist on the pancreatic cancer dataset), indicating annotation robustness.

## Related tools

- **Census** (Annotates cell types on each UMAP embedding variant using its label-stabilizing algorithm on clusters and prediction contours; main model trained on Tabula Sapiens and Cancer Cell Line Encyclopedia.) — https://github.com/sjdlabgroup/Census
- **Seurat** (Builds the Seurat v4 object, runs the standard preprocessing pipeline (NormalizeData, ScaleData, FindVariableFeatures, RunPCA), and recomputes UMAP embeddings via RunUMAP for each hyperparameter combination.) — https://github.com/sjdlabgroup/Census
- **ggpubr** (Provides kruskal.test to run Kruskal-Wallis non-parametric tests over per-cell-type F1 scores across UMAP parameter settings.) — https://github.com/kassambara/ggpubr

## Examples

```
res = census_main(obj, organ = 'Pancreas', predict_cancer = T)
```

## Evaluation signals

- Kruskal-Wallis P-values replicate the reported non-significant results: P = 0.99 for n.neighbors and P = 0.89 for min.dist on the pancreatic cancer dataset.
- Per-cell-type F1 scores are computed against reference labels for every UMAP variant, and their distributions across settings are exported to sensitivity_analysis_results.csv.
- Each embedding variant is generated with the full standard Seurat pipeline (NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunUMAP dims=1:50) before census_main is applied.
- Seurat v4 is loaded before the Census library in every R session, since Census fails to run otherwise.

## Limitations

- Census requires Seurat version 4 specifically; loading a different Seurat version or loading Census first will prevent it from running.
- The label-stabilizing step depends on UMAP-space clusters and prediction contours, so annotation output is inherently coupled to the embedding parameters being tested.
- Sensitivity analysis was demonstrated on the Peng et al. 2019 pancreatic cancer dataset with Census's available cancer models (Kidney, Large_Intestine, Liver, Lung, Mammary, Pancreas); other tissues are not covered by the cancer model.
- Kruskal-Wallis tests compare per-cell-type F1 score distributions; results are specific to the grid of n.neighbors and min.dist values explored.

## Evidence

- [intro] Label-stabilizing step rationale: "New datasets are annotated using the pretrained models followed by a custom-developed label-stabilizing algorithm (Fig. 1b). This involves using clusters and prediction contours in UMAP space"
- [readme] Seurat v4 requirement: "Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R."
- [readme] Standard Seurat preprocessing before RunUMAP: "obj = obj %>% CreateSeuratObject() %>% NormalizeData() %>% ScaleData() %>%
  FindVariableFeatures() %>% RunPCA() %>% RunUMAP(dims=1:50)"
- [readme] Census invocation on pancreatic cancer data: "res = census_main(obj, organ = 'Pancreas', predict_cancer = T)"
