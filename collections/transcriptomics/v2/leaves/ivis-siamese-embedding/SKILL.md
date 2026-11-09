---
name: ivis-siamese-embedding
description: Use when a high-dimensional dataset (e.g. single-cell expression, CyTOF,
  or synthetic benchmarks with redundant polynomial features) must be embedded in
  2–3 dimensions while preserving global cluster shapes and relative locations, or
  when t-SNE's spurious clustering is a concern.
when_to_use_negative:
- When the goal is strictly local structure at the expense of global geometry, or
  when a well-validated non-neural method is mandated — the paper shows t-SNE and
  related pairwise-(dis)similarity methods (e.g. LargeVis) tend to yield spurious
  clusters, but they remain standard for other goals
- When data cannot be scaled to [0, 1] or the analysis requires interpretability of
  the original feature axes rather than an embedding
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_2229
tools:
- name: ivis
  role: 'Primary algorithm: siamese neural network trained with a triplet loss that
    produces structure-preserving low-dimensional embeddings'
  repo: https://github.com/beringresearch/ivis
- name: t-SNE
  role: Baseline comparison embedding used to demonstrate ivis's preservation of global
    structure versus t-SNE's spurious clusters
- name: scikit-learn
  role: Provides MinMaxScaler for [0, 1] preprocessing, t-SNE implementation, PCA
    (Melanoma reduction to n=50), and the SVM classifier used for accuracy evaluation
- name: Python
  role: Implementation language for the ivis interface used to run the embedding workflow
  repo: https://github.com/beringresearch/ivis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-019-45301-0
    title: Structure-preserving visualisation of high dimensional single-cell datasets
  source_content_hash: sha256:549e3854f095c90b563e9c0040faaf5967b03d727cc958be7502c98bfcef8109
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beringresearch/ivis
  tools:
  - Python
  - ivis
  - scikit-learn
  - t-SNE
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# ivis-siamese-embedding

## Summary

Dimensionality reduction of high-dimensional datasets with ivis, a siamese neural network trained with a triplet loss, which preserves both local and global structure in low-dimensional embeddings — unlike t-SNE, which forms spurious compact clusters in the presence of uniform random noise.

## When to use

Use when a high-dimensional dataset (e.g. single-cell expression, CyTOF, or synthetic benchmarks with redundant polynomial features) must be embedded in 2–3 dimensions while preserving global cluster shapes and relative locations, or when t-SNE's spurious clustering is a concern.

## When NOT to use

- When the goal is strictly local structure at the expense of global geometry, or when a well-validated non-neural method is mandated — the paper shows t-SNE and related pairwise-(dis)similarity methods (e.g. LargeVis) tend to yield spurious clusters, but they remain standard for other goals
- When data cannot be scaled to [0, 1] or the analysis requires interpretability of the original feature axes rather than an embedding

## Inputs

- Numeric feature matrix (NumPy array, sparse matrix, or HDF5 file) scaled to [0, 1] with MinMaxScaler
- Synthetic benchmark datasets (uniform random noise, Cassini, Smiley; 5000 points, expanded to 9 features via polynomial combinations of degree ≤ 10)
- Single-cell expression data (e.g. CyTOF Levine-32-dim with arcsinh(X/5) transformation; scRNA-seq Melanoma SCP11 reduced to n=50 PCs)

## Outputs

- 2D (or low-dimensional) embedding coordinates per observation
- Side-by-side scatter plots (PNG) comparing ivis vs t-SNE embeddings colored by cluster membership or spatial position
- Overlay of ≥10 ivis embeddings of the Smiley dataset from different random initializations demonstrating stability

## How to apply

Scale the input features to the [0, 1] range with sklearn's MinMaxScaler, since ivis expects preprocessed values in this range. Fit ivis with embedding_dims=2 and a nearest-neighbour parameter such as k=15; the paper states default parameters were used to fit multiple models, and triplets are constructed by sampling positive points from the KNN vector and negative points from outside the KNN vector. Embed the scaled matrix via model.fit_transform to obtain the 2D coordinates. As a control, embed the same data with t-SNE (scikit-learn, default perplexity) and render side-by-side scatter plots: ivis should retain the global structure (no artificial clusters in random-noise data) while t-SNE yields spurious clusters with clean boundaries. To confirm robustness, repeat the embedding with different random initializations and overlay the results — cluster shapes and relative positions should be stable across runs. Quantitative validation can use the Mantel test with Pearson correlation between original and embedded distance matrices, and SVM classifier accuracy (train on 75%, evaluate on 25%) on the embedding.

## Related tools

- **ivis** (Primary algorithm: siamese neural network trained with a triplet loss that produces structure-preserving low-dimensional embeddings) — https://github.com/beringresearch/ivis
- **t-SNE** (Baseline comparison embedding used to demonstrate ivis's preservation of global structure versus t-SNE's spurious clusters)
- **scikit-learn** (Provides MinMaxScaler for [0, 1] preprocessing, t-SNE implementation, PCA (Melanoma reduction to n=50), and the SVM classifier used for accuracy evaluation)
- **Python** (Implementation language for the ivis interface used to run the embedding workflow) — https://github.com/beringresearch/ivis

## Examples

```
from ivis import Ivis; from sklearn.preprocessing import MinMaxScaler; X_scaled = MinMaxScaler().fit_transform(X); embeddings = Ivis(embedding_dims=2, k=15).fit_transform(X_scaled)
```

## Evaluation signals

- ivis panel of the uniform random-noise control contains no artificial clusters, while the t-SNE panel shows spurious clusters with clean boundaries
- For Cassini and Smiley benchmarks, ivis preserves both the shape and relative locations of each cluster in the embedding space
- Ten ivis embeddings of the Smiley dataset with different random initializations are qualitatively consistent (stable cluster shapes and relative positions)
- Mantel test with Pearson correlation between the original distance matrix and the embedded distance matrices yields a high correlation continuum over stratified subsamples (n=50 of 1000 observations)
- SVM classifier trained on 75% of each subsample and evaluated on the remaining 25% retains high label accuracy in the ivis embedding

## Limitations

- t-SNE (and possibly related methods like LargeVis) tends to yield spurious clusters with clean boundaries in the presence of random noise, so comparisons must be interpreted carefully
- IVis is robust to hyperparameter settings per the paper, but the paper references these robustness analyses only via Supplementary Figures without in-text hyperparameter values
- Sensitivity to preprocessing: values must be scaled to the [0, 1] range (and, for CyTOF data like Levine, arcsinh-transformed as X = np.arcsinh(X/5)) before embedding

## Evidence

- [abstract] ivis utilizes a siamese neural network architecture that is trained using a novel triplet loss function.: "ivis utilizes a siamese neural network architecture that is trained using a novel triplet loss function."
- [abstract] ivis is made publicly available through Python and R interfaces on https://github.com/beringresearch/ivis: "ivis is made publicly available through Python and R interfaces on https://github.com/beringresearch/ivis"
- [methods] Prior to preprocessing, values in both datasets were scaled to [0, 1] range.: "Prior to preprocessing, values in both datasets were scaled to [0, 1] range."
- [methods] The level of correlation between the original distance matrix and the distance matrices in the embedding spaces was then assessed using the Mantel test: "The level of correlation between the original distance matrix and the distance matrices in the embedding spaces was then assessed using the `Mantel test"
- [discussion] in the presence of random noise, the algorithm tends to yield spurious clusters with clean boundaries, potentially hindering accurate interpretation and discovery (Fig. 1C): "in the presence of random noise, the algorithm tends to yield spurious clusters with clean boundaries, potentially hindering accurate interpretation and discovery (Fig. 1C)"
- [discussion] in the synthetic Smiley dataset, ivis preserved both the shape and relative locations of each cluster in the embedding space: "For example, in the synthetic Smiley dataset, ivis preserved both the shape and relative locations of each cluster in the embedding space"
- [readme] model = Ivis(embedding_dims=2, k=15) ... embeddings = model.fit_transform(X_scaled): "model = Ivis(embedding_dims=2, k=15)

embeddings = model.fit_transform(X_scaled)"
