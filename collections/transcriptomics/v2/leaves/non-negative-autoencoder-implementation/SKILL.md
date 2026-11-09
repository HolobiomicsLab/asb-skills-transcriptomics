---
name: non-negative-autoencoder-implementation
description: Use when raw scRNA-seq count matrices (genes x cells) must be filtered
  to an informative gene set and compressed to a ~50-dimensional representation before
  clustering, visualization, or classification with scDHA.
when_to_use_negative:
- When the input is already a reduced feature table or a low-dimensional embedding
  rather than a raw genes-by-cells count matrix — scDHA's main function expects a
  matrix with rows as samples and columns as genes.
- When the goal is dropout imputation via heavy statistical modelling (e.g. CIDR's
  implicit imputation) rather than part-based gene filtering through a non-negative
  autoencoder.
- When downstream analysis requires trajectory or pseudotime inference directly —
  that is handled by scDHA's second module (stacked Bayesian autoencoder / VAE) and
  scDHA.pt, not this first-module filter.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scDHA
  role: Primary framework providing the non-negative kernel autoencoder used for gene/component
    filtering and 50-dimensional compression of scRNA-seq data
  repo: github.com/duct317/scDHA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-21312-2
    title: Fast and precise single-cell data analysis using a hierarchical autoencoder
  source_content_hash: sha256:82e7f0955134d161a5d9fd64bfd443e452e409c74f1254c5f5855e703ae2e039
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scDHA
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# non-negative-autoencoder-implementation

## Summary

Implement the first module of scDHA as a one-layer non-negative kernel autoencoder with a 50-node bottleneck that produces a part-based representation of scRNA-seq data, then use the encoder weight distribution to remove genes or components with insignificant contributions. This gene filtering step yields a compact, denoised input for downstream clustering and dimension reduction.

## When to use

Use when raw scRNA-seq count matrices (genes x cells) must be filtered to an informative gene set and compressed to a ~50-dimensional representation before clustering, visualization, or classification with scDHA.

## When NOT to use

- When the input is already a reduced feature table or a low-dimensional embedding rather than a raw genes-by-cells count matrix — scDHA's main function expects a matrix with rows as samples and columns as genes.
- When the goal is dropout imputation via heavy statistical modelling (e.g. CIDR's implicit imputation) rather than part-based gene filtering through a non-negative autoencoder.
- When downstream analysis requires trajectory or pseudotime inference directly — that is handled by scDHA's second module (stacked Bayesian autoencoder / VAE) and scDHA.pt, not this first-module filter.

## Inputs

- Raw scRNA-seq count matrix (genes x cells), e.g. as loaded from a CSV or an R data object such as the Goolam dataset
- Number of bottleneck nodes (50) and number of genes to retain (top 5000) as module configuration

## Outputs

- selected_genes.csv — list of ~5000 selected genes ranked by encoder weight variance
- scdha_module1_representation.csv — 50-dimensional compressed representation, one row per cell

## How to apply

Load the raw scRNA-seq count matrix (genes x cells) and apply min-max scaling so each cell's expression profile lies in [0, 1]. Build a one-layer autoencoder with a 50-node bottleneck in the scDHA framework (Torch-backed R package; Python/TensorFlow environment is no longer required), constraining encoder weights to be non-negative (e.g. clamping/rectifying weights after each update) so the learned representation is non-negative and part-based. Train with a reconstruction loss until convergence, then rank genes by the variance of their encoder weights across the 50 hidden nodes and retain the top 5000 genes. Rationale: the non-negative kernel structure lets scDHA identify and discard genes or components with insignificant contributions to the part-based representation, detaching technical noise and dropout from informative biological signals.

## Related tools

- **scDHA** (Primary framework providing the non-negative kernel autoencoder used for gene/component filtering and 50-dimensional compression of scRNA-seq data) — github.com/duct317/scDHA

## Examples

```
library(scDHA); data <- log2(t(Goolam$data) + 1); result <- scDHA(data, seed = 1)
```

## Evaluation signals

- Exactly ~5000 genes are present in the selected gene set (selected_genes.csv).
- The compressed representation has exactly one row per input cell and exactly 50 columns.
- The autoencoder's reconstruction loss decreases monotonically (or near-monotonically) over training epochs.
- All encoder weights remain non-negative after training (part-based representation invariant).
- Clustering on the 50-dimensional representation recovers known cell labels with a high adjusted Rand index, as in the Goolam example evaluated with mclust::adjustedRandIndex.

## Limitations

- Results depend on stochastic training; reproducibility requires explicitly setting the seed (scDHA functions accept a seed parameter).
- The paper's discussion states accuracy claims as expectations rather than demonstrated limits, and defers the comparative evaluation against scVI to supplementary materials.
- The framework targets high-throughput scRNA-seq challenges (increasing cell numbers, technical noise, high dropout rate); generalization claims to other domains are not supported by cited experiments in the discussion.

## Evidence

- [abstract] The first module is a non-negative kernel autoencoder able to remove genes or components that have insignificant contributions to the part-based representation of the data.: "The first module is a non-negative kernel autoencoder able to remove genes or components that have insignificant contributions to the part-based representation of the data."
- [intro] Based on the weight distribution of the encoder, scDHA removes genes or components that have insignificant contributions to the representation: "Based on the weight distribution of the encoder, scDHA removes genes or components that have insignificant contributions to the representation"
- [readme] The package is now available on CRAN. The machine learning framework is changed from Tensorflow to Torch. Torch can be called directly from R.: "The machine learning framework is changed from Tensorflow to Torch. Torch can be called directly from R. Python environment is no longer required."
- [readme] The input is a matrix with rows as samples and columns as genes.: [evidence span withheld — anchor status: external_doc]
