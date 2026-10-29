---
name: adaptive-kernel-bandwidth-estimation
description: Use when a sample-by-feature single-cell matrix (e.g., PBMC zheng17-preprocessed
  data, first 30 principal components) must be converted into a KNN-based affinity
  graph to feed label-propagation clustering, and a full pairwise L2 distance matrix
  would be computationally expensive (>1 million points).
when_to_use_negative:
- 'Data is in a very high-dimensional representation (beyond the first 30 principal
  components): the curse of dimensionality makes distances more uniform, degrading
  KNN-based affinity estimates.'
- Input is already a computed affinity matrix or graph — regenerating affinities from
  raw features would discard it; the skill applies only when X is a sample-by-feature
  matrix.
- The downstream analysis requires nonlinear-dimensionality-reduced embeddings (UMAP,
  t-SNE, PHATE) rather than PCA-space affinities, which the article states Forest
  Fire Clustering should be performed with.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scanpy
  role: Provides the sc.pp.neighbors KNN search combined with the adaptive kernel
    to obtain the affinity matrix A from the data matrix
- name: scipy
  role: Sparse matrix acceleration for storing the KNN-based affinity matrix
- name: numba
  role: Numerical acceleration of the kernel/affinity computations
- name: numpy
  role: Linear algebra operations on the feature matrix and distance arrays
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:4bc682a9a72c29666967221304a9b7ded28185c39829c40ff018a782b387e6a3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - numba
  - numpy
  - scanpy
  - scikit-learn (sklearn)
  - scipy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# adaptive-kernel-bandwidth-estimation

## Summary

This skill constructs the affinity matrix A for Forest Fire Clustering by approximating the pairwise L2 distance matrix with KNN-to-KNN distances (via scanpy's sc.pp.neighbors) and converting those distances into affinities with an adaptive kernel, avoiding the prohibitive cost of full distance matrices on large single-cell datasets.

## When to use

Use when a sample-by-feature single-cell matrix (e.g., PBMC zheng17-preprocessed data, first 30 principal components) must be converted into a KNN-based affinity graph to feed label-propagation clustering, and a full pairwise L2 distance matrix would be computationally expensive (>1 million points).

## When NOT to use

- Data is in a very high-dimensional representation (beyond the first 30 principal components): the curse of dimensionality makes distances more uniform, degrading KNN-based affinity estimates.
- Input is already a computed affinity matrix or graph — regenerating affinities from raw features would discard it; the skill applies only when X is a sample-by-feature matrix.
- The downstream analysis requires nonlinear-dimensionality-reduced embeddings (UMAP, t-SNE, PHATE) rather than PCA-space affinities, which the article states Forest Fire Clustering should be performed with.

## Inputs

- Sample-by-feature matrix X (e.g., PBMC gene expression preprocessed with scanpy's 'zheng17' recipe, first 30 principal components)
- KNN search results (neighbor indices and distances) from scanpy sc.pp.neighbors

## Outputs

- Sparse affinity matrix A (KNN-approximated adaptive-kernel affinities) consumed by FFC.preprocess/fit
- Per-vertex degrees Di used to set acceptance thresholds Ti = 1/Di in the clustering fit

## How to apply

Start from the preprocessed sample-by-feature matrix X (PBMC or MCA gene expression normalized with scanpy's 'zheng17' recipe, first 30 principal components retained). Approximate the full L2 distance matrix by using only the L2 distances between the KNNs of each data point, obtained with scanpy's 'sc.pp.neighbors' KNN search. Convert these restricted distances into affinities with an adaptive kernel to yield the affinity matrix A, held as a scipy sparse matrix and accelerated with numba. The rationale is that representing the complete distance matrix for datasets with >1 million data points is computationally expensive, so the KNN-approximated kernel keeps affinity construction tractable. The resulting affinity matrix A and vertex degrees Di directly parameterize the downstream Forest Fire Clustering fit, where the per-vertex acceptance threshold is Ti = 1/Di. Cluster quality then depends on the fire temperature c (fire_temp=100 in the reference implementation), which strongly influences the size and number of discovered clusters.

## Related tools

- **scanpy** (Provides the sc.pp.neighbors KNN search combined with the adaptive kernel to obtain the affinity matrix A from the data matrix)
- **scipy** (Sparse matrix acceleration for storing the KNN-based affinity matrix)
- **numba** (Numerical acceleration of the kernel/affinity computations)
- **numpy** (Linear algebra operations on the feature matrix and distance arrays)

## Examples

```
from forest_fire_clustering.forest_fire_clustering import FFC; cluster_obj = FFC(); cluster_obj.preprocess(X); cluster_obj.fit(fire_temp=100); cluster_obj.cluster_labels
```

## Evaluation signals

- Every data point downstream receives exactly one cluster label after FFC.fit(fire_temp=100) — all vertices in the affinity graph are labeled at equilibrium.
- The affinity matrix A is sparse and restricted to KNN neighborhoods (built via sc.pp.neighbors), not a dense N x N distance matrix.
- Per-vertex acceptance thresholds equal the inverse degree, Ti = 1/Di, computed from the affinity matrix's vertex degrees.
- Clustering results (cluster_labels.csv) contain one row per input sample and benchmark competitively on purity, ARI, and silhouette scores.
- validate()/entropy()/pval() produce per-point posterior confidence, entropy list, and significance values without errors on the KNN-graph built data.

## Limitations

- The curse of dimensionality degrades clustering as distances become more uniform, so affinities from high-dimensional raw features are unreliable; use reduced representations (first 30 PCs) or nonlinear dimensionality reduction.
- The full distance matrix for datasets over 1 million data points is computationally expensive and must be approximated via KNN distances.
- The fire temperature c strongly influences the size and number of clusters discovered, so affinity construction must be paired with fire_temp sensitivity checks.

## Evidence

- [discussion] KNN distance approximation of L2 matrix: [evidence span withheld — anchor status: not_found]
- [discussion] scanpy sc.pp.neighbors for affinity matrix: "we directly use this function in our preprocessing step to obtain the affinity matrix A from the data matrix W."
- [discussion] scanpy recommendation: "the scanpy package in Python has a highly efficient and stable function called “sc.pp.neighbors”"
- [discussion] First 30 PCs input representation: "After normalization and standardization, the first 30 principal components were used for clustering."
- [readme] Dependencies for the implementation: "* python >= 3.6
    * numpy
    * scipy
    * scikit-learn
    * numba"
