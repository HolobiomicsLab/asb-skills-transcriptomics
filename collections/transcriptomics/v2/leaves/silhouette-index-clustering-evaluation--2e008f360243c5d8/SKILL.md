---
name: silhouette-index-clustering-evaluation
description: Use when comparing dimension-reduction or visualization methods for scRNA-seq
  data and you need a label-referenced metric of cell-type separation for each 2D
  embedding.
when_to_use_negative:
- The input is not a 2D embedding or dimension-reduction output — silhouette evaluation
  here is defined on visualization embeddings, not on raw high-dimensional count matrices
- No ground-truth cell-type labels are available, since the silhouette index must
  be computed with true labels as the cluster reference
- The goal is pseudotime or trajectory benchmarking (use scDHA.pt / Monocle-style
  evaluation instead of silhouette index)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scDHA
  role: Produces the candidate 2D visualization embeddings via scDHA.vis that are
    scored with the silhouette index against the competing methods
  repo: github.com/duct317/scDHA
- name: SCANPY
  role: Competing visualization/dimension-reduction method whose 2D embeddings are
    baselines in the silhouette-index comparison
- name: CIDR
  role: Competing imputation-and-dimensionality-reduction method for single-cell scRNA-seq
    data included among the baselines
  repo: VCCRI/CIDR
- name: Monocle3
  role: Trajectory-inference toolkit in the scDHA comparison landscape; not scored
    by silhouette index itself
  repo: cole-trapnell-lab/monocle3
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-21312-2
    title: Fast and precise single-cell data analysis using a hierarchical autoencoder
  source_content_hash: sha256:e493f593fb9e94bc2aefbb7f9ff53c6d5970681367f3f68d24aa76cc4db598eb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CIDR
  - Monocle3
  - SCANPY
  - scDHA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# silhouette-index-clustering-evaluation

## Summary

Evaluates the quality of low-dimensional single-cell embeddings (e.g. 2D visualization embeddings from scDHA.vis, PCA, t-SNE, UMAP, SCANPY) by computing the silhouette index against ground-truth cell-type labels, and comparing methods statistically across benchmark datasets.

## When to use

Use when comparing dimension-reduction or visualization methods for scRNA-seq data and you need a label-referenced metric of cell-type separation for each 2D embedding.

## When NOT to use

- The input is not a 2D embedding or dimension-reduction output — silhouette evaluation here is defined on visualization embeddings, not on raw high-dimensional count matrices
- No ground-truth cell-type labels are available, since the silhouette index must be computed with true labels as the cluster reference
- The goal is pseudotime or trajectory benchmarking (use scDHA.pt / Monocle-style evaluation instead of silhouette index)

## Inputs

- scRNA-seq count/expression datasets (e.g. 34 benchmark sets such as Goolam, Kolodziejczyk, Segerstolpe)
- Ground-truth cell-type label vectors aligned to the cells
- 2D embedding matrices produced by each method (scDHA.vis, PCA, t-SNE, UMAP, SCANPY)

## Outputs

- Per-dataset silhouette-index score table for every method
- Average silhouette-index comparison figure (e.g. average_si_comparison.pdf)
- Example visualization panels showing cell-type separation (Kolodziejczyk, Segerstolpe)
- One-sided Wilcoxon test p-value comparing scDHA.vis to each competitor (reported: p = 1.7e-6)

## How to apply

Load each benchmark scRNA-seq dataset together with its ground-truth cell-type labels, then compute a 2D embedding per dataset for each competing method under comparable settings (in the scDHA paper: 34 scRNA-seq datasets embedded with scDHA.vis vs PCA, t-SNE, UMAP, and SCANPY). Compute the silhouette index of each 2D embedding using the true cell-type labels as the cluster reference. Average the silhouette index per method across all datasets and test whether scDHA.vis exceeds each competitor with a one-sided Wilcoxon test; the paper reports significance at p = 1.7e-6 in scDHA.vis's favor. Render per-dataset score tables plus an average-SI comparison figure, and inspect example panels (Kolodziejczyk and Segerstolpe) for visible cell-type separation.

## Related tools

- **scDHA** (Produces the candidate 2D visualization embeddings via scDHA.vis that are scored with the silhouette index against the competing methods) — github.com/duct317/scDHA
- **SCANPY** (Competing visualization/dimension-reduction method whose 2D embeddings are baselines in the silhouette-index comparison)
- **CIDR** (Competing imputation-and-dimensionality-reduction method for single-cell scRNA-seq data included among the baselines) — VCCRI/CIDR
- **Monocle3** (Trajectory-inference toolkit in the scDHA comparison landscape; not scored by silhouette index itself) — cole-trapnell-lab/monocle3

## Examples

```
library(scDHA); data('Goolam'); data <- log2(t(Goolam$data) + 1); result <- scDHA(data, seed = 1); embed2d <- scDHA.vis(result, seed = 1)
```

## Evaluation signals

- scDHA.vis must attain the highest average silhouette index across all 34 benchmark datasets
- The advantage of scDHA.vis must be statistically significant against all four competing methods at the reported level (one-sided Wilcoxon, p = 1.7e-6)
- Kolodziejczyk and Segerstolpe example panels must show visibly better cell-type separation for scDHA.vis than competing embeddings
- Each silhouette computation must use the true cell-type labels as the cluster reference, not inferred clusters
- The saved per-dataset score table must contain one silhouette value per dataset per method under comparable embedding settings

## Limitations

- Validation depends on the availability of ground-truth cell-type labels for the benchmark datasets
- The scDHA discussion states accuracy claims as expectations ('the hierarchical autoencoder and the modified VAE (second module of scDHA) are expected to be more efficient than other autoencoders') and defers the scVI comparison to supplementary sections
- No explicit limitations or caveats section is acknowledged in the scDHA discussion; code/data availability statements are not provided there

## Evidence

- [discussion] scDHA outperforms state-of-the-art techniques in clustering, visualization, classification, and pseudo-time inference sub-fields: "We demonstrate that scDHA outperforms state-of-the-art techniques in each research sub-field."
- [readme] The scDHA software package can perform cell segregation through unsupervised learning, dimension reduction and visualization, cell classification, and time-trajectory inference on single-cell RNA sequencing data.: "The scDHA software package can perform cell segregation through unsupervised learning, dimension reduction and visualization, cell classification, and time-trajectory inference"
- [readme] silhouette index computation uses true cell-type labels as cluster reference: "Calculating adjusted Rand Index using mclust package: `mclust::adjustedRandIndex(cluster,label)`"
