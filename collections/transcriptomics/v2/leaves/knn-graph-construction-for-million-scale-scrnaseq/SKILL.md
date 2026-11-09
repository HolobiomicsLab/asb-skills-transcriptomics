---
name: knn-graph-construction-for-million-scale-scrnaseq
description: Use when a scRNA-seq or CyTOF expression matrix exceeds ~100k cells (e.g.
  1.3M-cell 10X mouse brain data) and a neighbourhood structure is needed for triplet-based
  dimensionality reduction where scikit-learn Barnes-Hut t-SNE would not finish within
  24 hours.
when_to_use_negative:
- Dataset is already a low-dimensional embedding or a precomputed distance/similarity
  matrix rather than a raw single-cell expression matrix
- Small datasets (e.g. iris, MNIST 70k, or Levine 265k CyTOF cells) where standard
  ivis k values suffice and million-scale hyperparameters (k=15–150) are unnecessary
- The goal is supervised classification of cell populations rather than scalable neighbourhood
  graph construction for unsupervised dimensionality reduction
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: ivis
  role: Builds the kNN graph and samples triplets (positives from the KNN vector,
    negatives outside it) to train the siamese network for linearly-scaling dimensionality
    reduction of million-cell datasets
  repo: https://github.com/beringresearch/ivis
- name: scikit-learn
  role: Provides MinMaxScaler for the required [0, 1] preprocessing of the expression
    matrix before kNN graph construction, and the Barnes-Hut t-SNE baseline that fails
    to complete at this scale
- name: Python
  role: Interface through which ivis is publicly available and used to orchestrate
    the million-scale kNN/triplet pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-019-45301-0
    title: Structure-preserving visualisation of high dimensional single-cell datasets
  source_content_hash: sha256:2cfa281194cbbb4983dd7d7e3826081c3fa941fbab74c8038b0be664eecab35d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beringresearch/ivis
  tools:
  - Python
  - ivis
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# knn-graph-construction-for-million-scale-scrnaseq

## Summary

Constructs a k-nearest-neighbour neighbourhood graph over a million-scale single-cell expression matrix (e.g. the 1.3M-cell 10X Genomics mouse brain dataset) to drive ivis triplet sampling, where positives are drawn from the KNN vector and negatives from outside it, enabling linearly-scaling dimensionality reduction.

## When to use

Use when a scRNA-seq or CyTOF expression matrix exceeds ~100k cells (e.g. 1.3M-cell 10X mouse brain data) and a neighbourhood structure is needed for triplet-based dimensionality reduction where scikit-learn Barnes-Hut t-SNE would not finish within 24 hours.

## When NOT to use

- Dataset is already a low-dimensional embedding or a precomputed distance/similarity matrix rather than a raw single-cell expression matrix
- Small datasets (e.g. iris, MNIST 70k, or Levine 265k CyTOF cells) where standard ivis k values suffice and million-scale hyperparameters (k=15–150) are unnecessary
- The goal is supervised classification of cell populations rather than scalable neighbourhood graph construction for unsupervised dimensionality reduction

## Inputs

- Full scRNA-seq expression matrix (e.g. 1.3M-cell 10X Genomics mouse brain dataset), dense numpy array, sparse matrix, or hdf5 file
- kNN neighbourhood size k (15–150 for datasets >1M observations)
- Min-max scaled feature values in [0, 1]

## Outputs

- Low-dimensional (embedding_dims=2) ivis embedding of all cells without subsampling
- Fitted ivis model with a transform method supporting addition of new data points to the existing embedding
- Runtime measurements (e.g. runtime_benchmark_results.csv rows) comparing ivis (<30 min) vs Barnes-Hut t-SNE (>24 h)

## How to apply

Scale the expression matrix to the [0, 1] range with sklearn's MinMaxScaler, per the paper's preprocessing convention for single-cell data. Then run ivis on the full (unsubsampled) matrix using the large-dataset hyperparameters recommended for >1M observations: k in the 15–150 range, n_epochs_without_progress of 2–3, and model='szubert'. ivis samples positive data-points for each triplet from the KNN vector and negative data-points from outside the KNN vector, so the kNN graph is the core structure that defines the triplets minimised by the siamese network's triplet loss. Because ivis scales linearly to hundreds of thousands of cells and extends to millions, no subsampling step is applied; on the 1.3M-cell 10X dataset the full run completes in under 30 minutes, whereas Barnes-Hut t-SNE on the same matrix exceeds 24 hours.

## Related tools

- **ivis** (Builds the kNN graph and samples triplets (positives from the KNN vector, negatives outside it) to train the siamese network for linearly-scaling dimensionality reduction of million-cell datasets) — https://github.com/beringresearch/ivis
- **scikit-learn** (Provides MinMaxScaler for the required [0, 1] preprocessing of the expression matrix before kNN graph construction, and the Barnes-Hut t-SNE baseline that fails to complete at this scale)
- **Python** (Interface through which ivis is publicly available and used to orchestrate the million-scale kNN/triplet pipeline)

## Examples

```
from sklearn.preprocessing import MinMaxScaler; from ivis import Ivis; X_scaled = MinMaxScaler().fit_transform(X); embeddings = Ivis(embedding_dims=2, k=15, model='szubert', n_epochs_without_progress=3).fit_transform(X_scaled)
```

## Evaluation signals

- Expression matrix values are in [0, 1] after MinMaxScaler before graph construction
- The ivis run completes on the full 1.3M-cell 10X matrix in under 30 minutes with no subsampling
- Runtime output contains a row for both ivis and Barnes-Hut t-SNE, with t-SNE exceeding the 24-hour timeout
- Hyperparameters match the >1M-observation recommendation: k in 15–150, n_epochs_without_progress=2–3, model='szubert'
- Resulting 2D embedding preserves global structure (e.g. distinct nuclei clusters corresponding to known cell types on comparable datasets)

## Limitations

- Runtime claims are validated against the paper's linear-scaling claim to hundreds of thousands of cells; exact wall-time on the 1.3M-cell dataset depends on hardware
- t-SNE comparisons require a timeout policy (24 h) since scikit-learn Barnes-Hut t-SNE may never finish at this scale
- ivis is robust to hyperparameter settings but the paper's >1M hyperparameters are documented via supplementary tables rather than the discussion text

## Evidence

- [methods] Prior to preprocessing, values in both datasets were scaled to [0, 1] range.: "Prior to preprocessing, values in both datasets were scaled to [0, 1] range."
- [discussion] ivis samples positives and negatives from KNN vector and outside it: "Given that ivis samples positive and negative data-points for each triplet, from the KNN vector and outside the KNN vector respectively"
- [abstract] ivis scales linearly to hundreds of thousands of cells: "and scales linearly to hundreds of thousands of cells"
- [readme] ivis is designed for very large datasets with a siamese neural network trained on triplets: "Ivis is designed to reduce dimensionality of very large datasets using a siamese neural network trained on triplets."
- [readme] ivis extends to millions of observations: [evidence span withheld — anchor status: external_doc]
