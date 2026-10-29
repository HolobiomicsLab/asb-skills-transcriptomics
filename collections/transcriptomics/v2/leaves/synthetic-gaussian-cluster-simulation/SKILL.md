---
name: synthetic-gaussian-cluster-simulation
description: Use when benchmarking whether a clustering method can infer newly emerging
  clusters in streaming out-of-sample data, and you need controllable ground-truth
  labels and tunable cluster-count/noise conditions.
when_to_use_negative:
- The goal is clustering real experimental data (e.g., scRNA-seq like PBMC or MCA)
  rather than validating online inductive inference with ground-truth synthetic labels
- The data has only one cluster, where posterior/uncertainty estimation and multi-cluster
  discovery evaluation are not meaningful
- The data is very high-dimensional without nonlinear dimensionality reduction, since
  the method is limited by the curse of dimensionality and distances become more uniform
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scikit-learn (sklearn)
  role: Generate synthetic Gaussian cluster training data and provide metric implementations
    for benchmarking
- name: numpy
  role: Linear algebra operations and averaging ARI/purity scores over unseen-cluster-count
    and noise conditions
- name: scipy
  role: Sparse matrix acceleration for affinity computations in the joint training/testing
    affinity matrix
- name: numba
  role: Numerical acceleration of the clustering pipeline used in the benchmark
- name: forest-fire-clustering (FFC)
  role: 'Method under test: fitted on training clusters via preprocess(X) and fit(fire_temp=100),
    then extended to out-of-sample batches via joint affinity propagation'
  repo: https://github.com/gersteinlab/forest-fire-clustering
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:cb38a5e7b768590602437b0a303bd6fefc81e4d4c1fe21475cdc4c6386708af4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gersteinlab/forest-fire-clustering
  tools:
  - Python
  - forest-fire-clustering (FFC)
  - numba
  - numpy
  - scikit-learn (sklearn)
  - scipy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# synthetic-gaussian-cluster-simulation

## Summary

Generate synthetic Gaussian cluster benchmarks (e.g., four clusters with sigma=0.1 via scikit-learn) and streaming test batches with increasing unseen-cluster counts and additive Gaussian noise, to evaluate a clustering method's online out-of-sample cluster discovery (e.g., Forest Fire Clustering) using ARI and purity.

## When to use

Use when benchmarking whether a clustering method can infer newly emerging clusters in streaming out-of-sample data, and you need controllable ground-truth labels and tunable cluster-count/noise conditions.

## When NOT to use

- The goal is clustering real experimental data (e.g., scRNA-seq like PBMC or MCA) rather than validating online inductive inference with ground-truth synthetic labels
- The data has only one cluster, where posterior/uncertainty estimation and multi-cluster discovery evaluation are not meaningful
- The data is very high-dimensional without nonlinear dimensionality reduction, since the method is limited by the curse of dimensionality and distances become more uniform

## Inputs

- Synthetic training data as a [sample x feature] matrix of four Gaussian clusters with sigma=0.1 (scikit-learn)
- Streaming test batches with increasing counts of unseen Gaussian clusters
- Additive Gaussian noise levels applied to test batches
- Reference (ground-truth) cluster labels for training and test points

## Outputs

- In-sample cluster labels from the fitted clustering model
- Extended (propagated) cluster labels for out-of-sample test points
- Average ARI and purity scores across unseen-cluster-count and noise conditions

## How to apply

Generate synthetic training data of four Gaussian clusters with sigma=0.1 using scikit-learn, formatted as a [sample x feature] matrix. Simulate streaming test batches with increasing counts of unseen clusters and increasing additive Gaussian noise. Fit the method under test on the training data (for Forest Fire Clustering: cluster_obj.preprocess(X) then cluster_obj.fit(fire_temp=100)) to obtain in-sample labels. Project each test batch onto the low-dimensional training representations, apply mutual nearest neighbor batch correction between training and testing data, compute the joint affinity matrix, and propagate (extend) the training cluster labels to the out-of-sample test points. Score extended labels against reference labels with adjusted Rand index (ARI) and purity, averaging over unseen-cluster-count and noise conditions with numpy; accept the run when average ARI and purity both exceed 0.90.

## Related tools

- **scikit-learn (sklearn)** (Generate synthetic Gaussian cluster training data and provide metric implementations for benchmarking)
- **numpy** (Linear algebra operations and averaging ARI/purity scores over unseen-cluster-count and noise conditions)
- **scipy** (Sparse matrix acceleration for affinity computations in the joint training/testing affinity matrix)
- **numba** (Numerical acceleration of the clustering pipeline used in the benchmark)
- **forest-fire-clustering (FFC)** (Method under test: fitted on training clusters via preprocess(X) and fit(fire_temp=100), then extended to out-of-sample batches via joint affinity propagation) — https://github.com/gersteinlab/forest-fire-clustering

## Examples

```
from forest_fire_clustering.forest_fire_clustering import FFC; cluster_obj = FFC(); cluster_obj.preprocess(X_train); cluster_obj.fit(fire_temp=100)
```

## Evaluation signals

- Average ARI of extended out-of-sample labels exceeds 0.90 as the number of unseen clusters increases
- Average purity of extended out-of-sample labels exceeds 0.90 as Gaussian noise increases
- In-sample fit succeeds via FFC API (preprocess(X) then fit(fire_temp=100)) and yields coherent cluster_labels
- Test batches are correctly projected onto the training low-dimensional representations with mutual nearest neighbor batch correction applied before label propagation
- Online-learning performance remains robust across all simulated unseen-cluster-count and noise conditions rather than degrading sharply

## Limitations

- The fire temperature c strongly influences the size and number of discovered clusters (resolution-like sensitivity), so the fit(fire_temp=100) choice affects benchmark outcomes
- The method is limited by the curse of dimensionality and should be performed with nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE)
- For datasets over 1 million points, representing the distance matrix is computationally expensive and must be approximated via KNN

## Evidence

- [methods] Synthetic training data generation: "Synthetic data of different shapes were produced by the sklearn package in Python"
- [discussion] Online out-of-sample projection workflow: "the testing data are projected onto the low-dimensional representations of the training dataset. A joint affinity matrix of the training and testing data is then calculated"
- [discussion] Batch correction between training and testing: "we correct batch effects between the training and testing data with mutual nearest neighbor batch correction"
- [discussion] Benchmark metrics: "There are three main ways to evaluate the performance of unsupervised methods. First, purity scores evaluate the cluster labels based on a set of reference labels."
- [readme] FFC API usage from README: "cluster_obj = FFC()
cluster_obj.preprocess(X)
cluster_obj.fit(fire_temp=100)
cluster_obj.cluster_labels"
