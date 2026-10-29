---
name: gene-selection-by-encoder-weight-variance
description: Use when a raw scRNA-seq count matrix (genes x cells) must be reduced
  to a compact informative gene set before dimension reduction or clustering, and
  noise/dropout-driven genes need to be filtered by contribution to a part-based representation.
when_to_use_negative:
- The input is already a reduced feature table or a low-dimensional embedding rather
  than a raw gene-by-cell count matrix
- The goal is differential-expression testing or marker-gene discovery, where genes
  must be judged individually by statistical tests rather than by contribution to
  a part-based autoencoder representation
- The downstream tool requires untransformed raw counts (this workflow applies min-max
  scaling and log2(data + 1) transformation, which changes count semantics)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDHA
  role: Provides the non-negative kernel autoencoder whose encoder weight distribution
    is used to rank and filter genes with insignificant contributions
  repo: github.com/duct317/scDHA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-21312-2
    title: Fast and precise single-cell data analysis using a hierarchical autoencoder
  source_content_hash: sha256:4d73fa1143523a101f09c324742eb5a2f862829234b851b56086a85ae40950ce
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scDHA
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# gene-selection-by-encoder-weight-variance

## Summary

A feature-selection step from scDHA's first module: a non-negative kernel autoencoder with a 50-node bottleneck produces a part-based representation, and genes are ranked by the variance of their encoder weights across hidden nodes so that only the top ~5000 informative genes are retained for downstream analysis.

## When to use

Use when a raw scRNA-seq count matrix (genes x cells) must be reduced to a compact informative gene set before dimension reduction or clustering, and noise/dropout-driven genes need to be filtered by contribution to a part-based representation.

## When NOT to use

- The input is already a reduced feature table or a low-dimensional embedding rather than a raw gene-by-cell count matrix
- The goal is differential-expression testing or marker-gene discovery, where genes must be judged individually by statistical tests rather than by contribution to a part-based autoencoder representation
- The downstream tool requires untransformed raw counts (this workflow applies min-max scaling and log2(data + 1) transformation, which changes count semantics)

## Inputs

- Raw scRNA-seq count matrix (genes x cells), e.g. CSV/MatrixMarket or the scDHA Goolam dataset matrix
- Number of genes to retain (default ~5000 per the scDHA workflow)

## Outputs

- selected_genes.csv — list of the top ~5000 selected genes
- scdha_module1_representation.csv — 50-dimensional compressed representation of the cells (one row per cell, 50 columns)
- Encoder weight matrix from the non-negative kernel autoencoder (genes x 50 hidden nodes)

## How to apply

Load the raw scRNA-seq count matrix (genes x cells) and apply min-max scaling so each cell's expression profile lies in [0, 1]. Build a one-layer autoencoder with a 50-node bottleneck in the scDHA framework (scDHA now uses Torch from R; Python/TensorFlow is no longer required), constraining encoder weights to be non-negative (e.g. clamping/rectifying weights after each update) so the representation is non-negative and part-based. Train with a reconstruction loss until convergence, then rank genes by the variance of their encoder weights across the 50 hidden nodes and retain the top 5000 genes. The rationale is that non-negative encoder weights make each hidden node a 'part', so a gene whose weights vary little across parts contributes insignificantly to the representation and can be removed. Save the selected gene list and the 50-dimensional compressed representation of the cells for input to scDHA's second module (the Stacked Bayesian Self-learning Network).

## Related tools

- **scDHA** (Provides the non-negative kernel autoencoder whose encoder weight distribution is used to rank and filter genes with insignificant contributions) — github.com/duct317/scDHA

## Examples

```
library(scDHA); data <- log2(t(Goolam$data) + 1); result <- scDHA(data, seed = 1, n_components = 50); cluster <- result$cluster
```

## Evaluation signals

- Exactly ~5000 genes are selected and written to selected_genes.csv
- The compressed representation has exactly one row per cell and 50 columns
- Reconstruction loss of the autoencoder decreased monotonically (or overall) over training
- Encoder weights remain non-negative after every training update (clamping verified)
- Downstream scDHA clustering on the filtered representation yields a high adjusted Rand index against known labels (e.g. via mclust::adjustedRandIndex on the Goolam example)

## Limitations

- Genes with insignificant contributions are removed based solely on encoder weight distribution; genes important for biology but weakly encoded by the part-based representation may be discarded
- The 50-node bottleneck and top-5000 gene cutoff are fixed defaults from the scDHA workflow and may need adjustment for datasets with far fewer or more genes
- The paper's discussion states accuracy advantages as expectations rather than demonstrated results in some sub-fields and defers comparative claims (e.g. vs scVI) to supplementary materials

## Evidence

- [abstract] Non-negative kernel autoencoder filter step: "The first module is a non-negative kernel autoencoder able to remove genes or components that have insignificant contributions to the part-based representation of the data."
- [intro] Encoder-weight-based removal: "Based on the weight distribution of the encoder, scDHA removes genes or components that have insignificant contributions to the representation"
- [intro] Part-based representation from first module: "The first module is a non-negative kernel autoencoder that provides a non-negative, part-based representation of the data."
- [readme] scDHA runs on Torch in R without Python: "The machine learning framework is changed from Tensorflow to Torch. Torch can be called directly from R. Python environment is no longer required."
- [readme] scDHA function input format: [evidence span withheld — anchor status: external_doc]
