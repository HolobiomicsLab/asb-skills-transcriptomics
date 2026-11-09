---
name: scrnaseq-cell-type-label-assessment
description: Use when you need to quantitatively judge how well a dimension-reduction/visualization
  method separates known cell-type labels in scRNA-seq data, e.g. benchmarking an
  embedding against PCA, t-SNE, UMAP, or SCANPY.
when_to_use_negative:
- Cell-type labels are unavailable or unreliable — silhouette index requires true
  labels as cluster reference and cannot be computed without them
- The goal is clustering accuracy (e.g. adjusted Rand index) or pseudo-time inference
  rather than visual separation of labels in a 2D embedding
- Data are not single-cell transcriptomic profiles (e.g. bulk RNA-seq or non high-throughput
  data), which is outside the scDHA framework's stated scope
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDHA
  role: Provides scDHA for dimension reduction and scDHA.vis for 2D visualization
    whose embeddings are scored by silhouette index
  repo: github.com/duct317/scDHA
- name: SCANPY
  role: Competing visualization method benchmarked against scDHA.vis using the same
    silhouette-index protocol
- name: CIDR
  role: Alternative imputation-and-dimension-reduction method for scRNA-seq cited
    among clustering/dimensionality-reduction approaches
  repo: VCCRI/CIDR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-21312-2
    title: Fast and precise single-cell data analysis using a hierarchical autoencoder
  source_content_hash: sha256:e41cd6234a1cdbe3be6937f4fad46d355f6235af0db77102542e8b96bb82fc0d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CIDR
  - SCANPY
  - scDHA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# scrnaseq-cell-type-label-assessment

## Summary

Assesses whether a 2D embedding of single-cell RNA-seq data visually separates ground-truth cell types, by computing the silhouette index of each embedding against true labels and comparing methods statistically. Grounded in the scDHA paper's benchmark where scDHA.vis was compared to PCA, t-SNE, UMAP, and SCANPY across 34 scRNA-seq datasets.

## When to use

Use when you need to quantitatively judge how well a dimension-reduction/visualization method separates known cell-type labels in scRNA-seq data, e.g. benchmarking an embedding against PCA, t-SNE, UMAP, or SCANPY.

## When NOT to use

- Cell-type labels are unavailable or unreliable — silhouette index requires true labels as cluster reference and cannot be computed without them
- The goal is clustering accuracy (e.g. adjusted Rand index) or pseudo-time inference rather than visual separation of labels in a 2D embedding
- Data are not single-cell transcriptomic profiles (e.g. bulk RNA-seq or non high-throughput data), which is outside the scDHA framework's stated scope

## Inputs

- scRNA-seq expression matrices (rows = cells, columns = genes) for each benchmark dataset
- Ground-truth cell-type label vector per dataset
- Candidate 2D embeddings from each visualization method (scDHA.vis, PCA, t-SNE, UMAP, SCANPY)

## Outputs

- Per-dataset silhouette-index score table for every method
- Average silhouette-index comparison figure (e.g. average_si_comparison.pdf)
- Example 2D embedding panels colored by cell type (e.g. Kolodziejczyk and Segerstolpe panels)
- One-sided Wilcoxon test p-value comparing the best method against competitors

## How to apply

Load each scRNA-seq dataset together with its ground-truth cell-type labels (the paper's benchmark uses 34 scRNA-seq datasets, e.g. Kolodziejczyk and Segerstolpe). Compute a 2D embedding per dataset with each method under comparable settings — for the scDHA framework, first run scDHA for dimension reduction, then scDHA.vis on the reduced output for visualization. For every 2D embedding, compute the silhouette index using the true cell-type labels as the cluster reference. Average the silhouette index per method across datasets and test differences with a one-sided Wilcoxon signed-rank test (the paper reports p = 1.7e-6 in scDHA.vis's favor). Render an average-silhouette comparison figure plus example embedding panels (e.g. Kolodziejczyk, Segerstolpe) and save the per-dataset silhouette score table.

## Related tools

- **scDHA** (Provides scDHA for dimension reduction and scDHA.vis for 2D visualization whose embeddings are scored by silhouette index) — github.com/duct317/scDHA
- **SCANPY** (Competing visualization method benchmarked against scDHA.vis using the same silhouette-index protocol)
- **CIDR** (Alternative imputation-and-dimension-reduction method for scRNA-seq cited among clustering/dimensionality-reduction approaches) — VCCRI/CIDR

## Examples

```
library(scDHA); result <- scDHA(data, seed = 1); vis <- scDHA.vis(result); cluster_silhouette <- silhouette(as.numeric(label), dist(vis))
```

## Evaluation signals

- scDHA.vis attains the highest average silhouette index across all 34 datasets
- The advantage over PCA, t-SNE, UMAP, and SCANPY is significant at the reported significance level (one-sided Wilcoxon test p = 1.7e-6)
- Example panels (Kolodziejczyk, Segerstolpe) show visibly better cell-type separation than competing embeddings
- The per-dataset silhouette score table covers one score per (dataset, method) pair with no missing entries
- Embeddings are reproducible because scDHA results are reproducible by setting the seed (seed = 1 in the README example)

## Limitations

- Silhouette index depends entirely on the quality and completeness of the provided cell-type labels; noisy labels invalidate the metric
- scDHA requires the torch/libtorch backend (installed via torch::install_torch()); the framework originally depended on Tensorflow and now needs R with Torch
- Comparative claims against some methods (e.g. scVI) are deferred to supplementary materials rather than the main text, and the paper's discussion states accuracy claims as expectations rather than demonstrated results in that section

## Evidence

- [results] Across the 34 scRNA-seq data sets, scDHA consistently and significantly outperforms the competing visualization methods, with the highest average silhouette index values (one-sided Wilcoxon test p = 1.7 × 10−6).: [evidence span withheld — anchor status: not_found]
- [abstract] we demonstrate that scDHA outperforms state-of-the-art techniques in many research sub-fields of scRNA-seq analysis, including cell segregation through unsupervised learning, visualization of: "we demonstrate that scDHA outperforms state-of-the-art techniques in many research sub-fields of scRNA-seq analysis, including cell segregation through unsupervised learning, visualization of"
- [readme] - scDHA.vis: visualization. The input is demension reduction output.: [evidence span withheld — anchor status: external_doc]
- [readme] - The result is reproducible by setting seed for these functions.: [evidence span withheld — anchor status: external_doc]
