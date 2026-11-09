---
name: visualization-method-benchmarking
description: Use when comparing dimension-reduction/visualization methods (e.g. scDHA.vis
  vs PCA, t-SNE, UMAP, SCANPY) on scRNA-seq data where cell-type labels are available
  to quantify embedding quality.
when_to_use_negative:
- No ground-truth cell-type labels exist, since the silhouette index requires cluster
  reference labels to score embeddings
- The goal is downstream biological interpretation of a single embedding rather than
  quantitative comparison of visualization methods
- Comparing clustering (e.g. adjusted Rand index via mclust) or pseudo-time inference
  performance rather than 2D transcriptome landscape visualization
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDHA
  role: 'Focal visualization method: scDHA.vis produces 2D embeddings from scDHA''s
    low-dimensional representation for benchmarking'
  repo: github.com/duct317/scDHA
- name: SCANPY
  role: Competing visualization/dimension-reduction baseline benchmarked against scDHA.vis
- name: CIDR
  role: Competing imputation-and-dimensionality-reduction method for scRNA-seq mentioned
    among widely used tools in this space
  repo: VCCRI/CIDR
- name: Monocle
  role: State-of-the-art single-cell analysis toolkit referenced among competing scRNA-seq
    analysis frameworks
  repo: cole-trapnell-lab/monocle3
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-21312-2
    title: Fast and precise single-cell data analysis using a hierarchical autoencoder
  source_content_hash: sha256:5a102d7ee341ebbc12410c1792efa4525cf60018d7ed896c1697818f355a45b1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CIDR
  - Monocle
  - SCANPY
  - scDHA
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# visualization-method-benchmarking

## Summary

Benchmark 2D dimension-reduction/visualization methods for single-cell RNA-seq by computing silhouette index values against ground-truth cell-type labels across many datasets, then testing for statistically significant differences (e.g. one-sided Wilcoxon test). Grounded in the scDHA paper's comparison of scDHA.vis against PCA, t-SNE, UMAP, and SCANPY on 34 scRNA-seq datasets.

## When to use

Use when comparing dimension-reduction/visualization methods (e.g. scDHA.vis vs PCA, t-SNE, UMAP, SCANPY) on scRNA-seq data where cell-type labels are available to quantify embedding quality.

## When NOT to use

- No ground-truth cell-type labels exist, since the silhouette index requires cluster reference labels to score embeddings
- The goal is downstream biological interpretation of a single embedding rather than quantitative comparison of visualization methods
- Comparing clustering (e.g. adjusted Rand index via mclust) or pseudo-time inference performance rather than 2D transcriptome landscape visualization

## Inputs

- 34 scRNA-seq benchmark datasets (gene-by-cell expression matrices, e.g. log2-transformed counts such as log2(data + 1) as in the scDHA Goolam example)
- Ground-truth cell-type label vector for each dataset
- 2D embeddings produced by scDHA.vis, PCA, t-SNE, UMAP, and SCANPY under comparable settings

## Outputs

- Per-dataset silhouette index score table for every method
- Average silhouette index comparison figure across the 34 datasets (average_si_comparison.pdf)
- Example visualization panels (e.g. Kolodziejczyk and Segerstolpe embeddings)
- One-sided Wilcoxon test statistic and p-value (paper value: p = 1.7e-6 favoring scDHA.vis)

## How to apply

For each benchmark dataset, load the expression matrix together with its ground-truth cell-type labels, then compute a 2D embedding with every method under comparable settings — here scDHA.vis (fed from scDHA's non-negative kernel autoencoder plus stacked Bayesian Self-learning Network low-dimensional projection) and the four competitors PCA, t-SNE, UMAP, and SCANPY. Score each 2D embedding with the silhouette index using the true cell-type labels as cluster reference, so the metric directly rewards embeddings that separate known cell types. Average the silhouette index per method across all 34 datasets and assess pairwise differences between the focal method (scDHA.vis) and each competitor with a one-sided Wilcoxon signed-rank test; the paper reports p = 1.7e-6 in favor of scDHA.vis. Finally, render per-dataset example panels (e.g. Kolodziejczyk and Segerstolpe) to visually confirm cell-type separation, and export the per-dataset score table plus the averaged comparison figure.

## Related tools

- **scDHA** (Focal visualization method: scDHA.vis produces 2D embeddings from scDHA's low-dimensional representation for benchmarking) — github.com/duct317/scDHA
- **SCANPY** (Competing visualization/dimension-reduction baseline benchmarked against scDHA.vis)
- **CIDR** (Competing imputation-and-dimensionality-reduction method for scRNA-seq mentioned among widely used tools in this space) — VCCRI/CIDR
- **Monocle** (State-of-the-art single-cell analysis toolkit referenced among competing scRNA-seq analysis frameworks) — cole-trapnell-lab/monocle3

## Examples

```
library(scDHA); data <- log2(t(Goolam$data) + 1); result <- scDHA(data, seed = 1); vis <- scDHA.vis(result$low_dim);
```

## Evaluation signals

- scDHA.vis attains the highest average silhouette index across all 34 datasets
- One-sided Wilcoxon test confirms scDHA.vis significantly exceeds all four competing methods at the reported significance level (p = 1.7e-6)
- Kolodziejczyk and Segerstolpe example panels show visibly better cell-type separation for scDHA.vis than competing embeddings
- Every method is run under comparable settings on identical input datasets with the same ground-truth labels used for silhouette computation
- Per-dataset silhouette-index table is complete for all 5 methods across all 34 datasets

## Limitations

- Benchmark validity depends on silhouette index with true labels as proxy for visual quality; other metrics may rank methods differently
- The paper's discussion states accuracy claims partly as expectations ('expected to be more efficient than other autoencoders') rather than demonstrated results
- Comparative claim against scVI is deferred to supplementary materials (Supplementary Section 6 and Fig. 25–32) rather than presented in the main discussion

## Evidence

- [results] Across the 34 scRNA-seq data sets, scDHA consistently and significantly outperforms the competing visualization methods: [evidence span withheld — anchor status: not_found]
- [abstract] silhouette index with true labels and Wilcoxon test: "we demonstrate that scDHA outperforms state-of-the-art techniques in many research sub-fields of scRNA-seq analysis, including cell segregation through unsupervised learning, visualization of"
- [readme] scDHA.vis usage from README: [evidence span withheld — anchor status: external_doc]
- [readme] reproducibility via seed: [evidence span withheld — anchor status: external_doc]
- [abstract] low-dimensional projection module: "The second module is a stacked Bayesian autoencoder that projects the data onto a low-dimensional space (compressed)."
