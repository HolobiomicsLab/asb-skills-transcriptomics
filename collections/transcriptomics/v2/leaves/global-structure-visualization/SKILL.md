---
name: global-structure-visualization
description: Use when a high-dimensional dataset (e.g. nine-feature synthetic benchmarks,
  CyTOF, or scRNA-seq expression matrices) must be visualised in 2D without artificial
  cluster artifacts, and relative cluster locations and shapes must be preserved.
when_to_use_negative:
- When the primary goal is strictly local neighborhood structure and cluster separation
  is more important than relative inter-cluster distances — t-SNE or SIMLR may be
  preferred per the paper's discussion
- When the data are not comparable points in a feature space, e.g. already 2D or categorical-only
  inputs with no KNN structure for triplet sampling
- When an exact linear projection is required for interpretation — PCA is the appropriate
  reduction (as used to reduce Melanoma to n=50 before embedding)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ivis
  role: 'Primary dimensionality-reduction method: siamese neural network trained with
    a triplet loss that preserves global structure; fitted with default parameters
    to produce 2D embeddings'
  repo: https://github.com/beringresearch/ivis
- name: t-SNE
  role: Comparison embedding used to reveal spurious cluster artifacts in random-noise
    controls
- name: scikit-learn
  role: Provides MinMaxScaler for [0, 1] preprocessing and the t-SNE implementation
- name: Python
  role: Implementation language for dataset generation, polynomial feature expansion,
    and scatter-plot rendering
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-019-45301-0
    title: Structure-preserving visualisation of high dimensional single-cell datasets
  source_content_hash: sha256:ad431bbb1273897885e114beb49432991b49c10e73661224f3b3343e9a375da3
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# global-structure-visualization

## Summary

A dimensionality-reduction skill using the ivis siamese neural-network algorithm (trained with a triplet loss) to embed high-dimensional data in 2D while preserving global structure, contrasting with t-SNE which yields spurious clusters under uniform random noise.

## When to use

Use when a high-dimensional dataset (e.g. nine-feature synthetic benchmarks, CyTOF, or scRNA-seq expression matrices) must be visualised in 2D without artificial cluster artifacts, and relative cluster locations and shapes must be preserved.

## When NOT to use

- When the primary goal is strictly local neighborhood structure and cluster separation is more important than relative inter-cluster distances — t-SNE or SIMLR may be preferred per the paper's discussion
- When the data are not comparable points in a feature space, e.g. already 2D or categorical-only inputs with no KNN structure for triplet sampling
- When an exact linear projection is required for interpretation — PCA is the appropriate reduction (as used to reduce Melanoma to n=50 before embedding)

## Inputs

- Nine-dimensional synthetic datasets generated from 2D bases (uniform random noise, Cassini, Smiley; 5000 points) via polynomial feature expansion (degree <= 10)
- Single-cell expression matrices, e.g. CyTOF Levine-32-dim (265,627 cells x 32 markers) or scRNA-seq data (Melanoma SCP11, 4,645 x 23,686, PCA-reduced to n=50)
- Cluster labels or spatial-position values for coloring scatter plots

## Outputs

- Two-dimensional embedding coordinates from ivis (and t-SNE for comparison)
- Side-by-side PNG scatter plots of ivis vs t-SNE embeddings colored by cluster membership or spatial position
- Overlay plot of ten repeated ivis embeddings demonstrating stability across random initializations

## How to apply

Scale the input feature matrix to the [0, 1] range with sklearn's MinMaxScaler, since ivis expects MinMax-scaled inputs. For synthetic benchmarks, expand the base 2D datasets (uniform random noise, Cassini, Smiley; 5000 points, fixed seed) to nine features using polynomial feature combinations of degree <= 10. Fit ivis with default parameters and embedding_dims=2 (README example uses k=15), and in parallel fit t-SNE (scikit-learn, default perplexity) as a comparison embedding. Render side-by-side 2D scatter plots colored by cluster membership or spatial position: ivis should retain the noise-free structure of the original data, while t-SNE tends to produce spurious clusters with clean boundaries. To confirm stability, repeat the ivis embedding (e.g. of the Smiley dataset) ten times with different random initializations and overlay the embeddings; cluster shapes and relative positions should remain consistent, reproducing Supplementary Fig. S2.

## Related tools

- **ivis** (Primary dimensionality-reduction method: siamese neural network trained with a triplet loss that preserves global structure; fitted with default parameters to produce 2D embeddings) — https://github.com/beringresearch/ivis
- **t-SNE** (Comparison embedding used to reveal spurious cluster artifacts in random-noise controls)
- **scikit-learn** (Provides MinMaxScaler for [0, 1] preprocessing and the t-SNE implementation)
- **Python** (Implementation language for dataset generation, polynomial feature expansion, and scatter-plot rendering)

## Examples

```
from ivis import Ivis; from sklearn.preprocessing import MinMaxScaler; X_scaled = MinMaxScaler().fit_transform(X); model = Ivis(embedding_dims=2, k=15); embeddings = model.fit_transform(X_scaled)
```

## Evaluation signals

- Random-noise control: the ivis embedding of uniform random noise must show no artificial clusters, whereas t-SNE must show spurious clusters with clean boundaries
- Cassini and Smiley datasets: ivis panels must preserve both the shape and relative locations of each cluster in the embedding space
- Stability check: ten repeated ivis embeddings of the Smiley dataset with different random initializations, when overlaid, must be qualitatively consistent
- Preprocessing invariant: input matrix values fall within [0, 1] after MinMaxScaler before fitting ivis
- Mantel test with Pearson correlation between original and embedding distance matrices can be used to quantify distance preservation (with random stratified subsamples of 1000 observations, n=50)

## Limitations

- t-SNE spurious clustering may be a general feature of algorithms that preserve pairwise (dis)similarities (e.g. LargeVis), so comparisons should account for this class of methods
- Similarity-based methods can fail or need adaptation in the presence of missing data; DeepCyTOF's denoising autoencoder addresses this for CyTOF
- In the presence of uniform random noise, t-SNE yields spurious clusters with clean boundaries that can hinder interpretation — always include a noise control when benchmarking
- Hyperparameter sensitivity: the paper reports ivis is robust to hyperparameter settings, but this is documented via supplementary figures rather than explicit values in the main text

## Evidence

- [abstract] ivis preserves global data structures in a low-dimensional space: "Results on simulated and real datasets demonstrate that ivis preserves global data structures in a low-dimensional space"
- [methods] Polynomial feature expansion: "we will artificially add reduntant features to the original datasets using polynomial combinations (degree ≤ 10) of the original features."
- [discussion] t-SNE spurious clusters: "in the presence of random noise, the algorithm tends to yield spurious clusters with clean boundaries, potentially hindering accurate interpretation and discovery (Fig. 1C)"
- [discussion] Smiley dataset preservation: "For example, in the synthetic Smiley dataset, ivis preserved both the shape and relative locations of each cluster in the embedding space"
- [readme] README ivis invocation with MinMaxScaler: "X_scaled = MinMaxScaler().fit_transform(X)

model = Ivis(embedding_dims=2, k=15)

embeddings = model.fit_transform(X_scaled)"
