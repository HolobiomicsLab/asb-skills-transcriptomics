---
name: online-inductive-label-inference
description: Use when newly arriving (streaming) data points must be assigned to existing
  clusters or evaluated for novel cluster emergence, without refitting the clustering
  model on the full combined dataset.
when_to_use_negative:
- Data has only one cluster — Monte Carlo internal validation cannot accurately estimate
  uncertainty since every point receives a single label type
- Data must be analyzed in very high dimensions without nonlinear dimensionality reduction
  — the method mimics a low-dimensional fire propagation process and suffers from
  the curse of dimensionality
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: forest-fire-clustering
  role: Provides the FFC API (preprocess, fit) to fit training clusters whose labels
    are extended to test batches
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: scikit-learn (sklearn)
  role: Generates synthetic Gaussian cluster training data and test batches
- name: numpy
  role: Linear algebra operations and averaging of ARI/purity scores over conditions
- name: scipy
  role: Sparse matrix acceleration for the joint affinity computations
- name: numba
  role: Numerical acceleration of the propagation routines
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:582f31e0d20b80968fbce445165d4e0d670bed5c66b3cc4276d64c1da8219e3f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gersteinlab/forest-fire-clustering
  tools:
  - Python
  - forest-fire-clustering
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
yes_rate: 0.7142857142857143
n_claims: 14
---

# online-inductive-label-inference

## Summary

Extends Forest Fire Clustering labels to newly arriving (out-of-sample) data via low-dimensional projection, mutual nearest neighbor batch correction, and joint-affinity-based label propagation, enabling robust online-learning inference of new clusters.

## When to use

Use when newly arriving (streaming) data points must be assigned to existing clusters or evaluated for novel cluster emergence, without refitting the clustering model on the full combined dataset.

## When NOT to use

- Data has only one cluster — Monte Carlo internal validation cannot accurately estimate uncertainty since every point receives a single label type
- Data must be analyzed in very high dimensions without nonlinear dimensionality reduction — the method mimics a low-dimensional fire propagation process and suffers from the curse of dimensionality

## Inputs

- Training data matrix X of shape [sample x feature] with known cluster labels (e.g., four Gaussian clusters generated with scikit-learn at sigma=0.1)
- Streaming test batches containing increasing counts of unseen clusters and increasing additive Gaussian noise

## Outputs

- Extended cluster labels for out-of-sample test points
- Joint affinity matrix of training and testing points
- ARI and purity scores averaged over unseen-cluster-count and noise conditions

## How to apply

Fit Forest Fire Clustering on the training data (a [sample x feature] matrix) using the FFC API: cluster_obj.preprocess(X) then cluster_obj.fit(fire_temp=100). Project each incoming test batch onto the low-dimensional training representations, then correct batch effects between training and testing data with mutual nearest neighbor (MNN) batch correction. Compute a joint affinity matrix spanning training and testing points, and propagate (extend) the training cluster labels to out-of-sample points through that matrix. Validate by scoring extended labels against reference labels with adjusted Rand index (ARI) and purity — in the benchmark with four training Gaussian clusters (sigma=0.1), average ARI and purity both stayed above 0.90 as unseen-cluster count and additive Gaussian noise increased, so treat >0.90 as the acceptance threshold.

## Related tools

- **forest-fire-clustering** (Provides the FFC API (preprocess, fit) to fit training clusters whose labels are extended to test batches) — https://github.com/gersteinlab/forest-fire-clustering
- **scikit-learn (sklearn)** (Generates synthetic Gaussian cluster training data and test batches)
- **numpy** (Linear algebra operations and averaging of ARI/purity scores over conditions)
- **scipy** (Sparse matrix acceleration for the joint affinity computations)
- **numba** (Numerical acceleration of the propagation routines)

## Examples

```
from forest_fire_clustering.forest_fire_clustering import FFC; cluster_obj = FFC(); cluster_obj.preprocess(X); cluster_obj.fit(fire_temp=100)
```

## Evaluation signals

- Average ARI over all unseen-cluster-count and Gaussian-noise conditions exceeds 0.90
- Average purity score over the same conditions exceeds 0.90
- Out-of-sample cluster discovery remains robust (ARI/purity stable) as the number of unseen clusters increases
- Training labels are reproduced consistently after preprocessing and fit with fire_temp=100 before extension
- MNN batch correction yields a joint affinity matrix in which test points align with their corresponding training clusters

## Limitations

- Fire temperature c strongly influences the size and number of discovered clusters (resolution-like sensitivity), so the training fit may need tuning before extension
- Sensitive to high dimensionality; must be performed with nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE)
- For datasets over 1 million points the distance matrix is computationally expensive and must be approximated via KNN distances
- Curse of dimensionality degrades clustering as distances become more uniform

## Evidence

- [discussion] Online clustering via joint affinity projection: "the testing data are projected onto the low-dimensional representations of the training dataset. A joint affinity matrix of the training and testing data is then calculated"
- [discussion] MNN batch correction: "we correct batch effects between the training and testing data with mutual nearest neighbor batch correction"
- [abstract] Online-learning capability claim: "can make robust, inductive inferences in an online-learning context and can readily scale to millions of cells"
- [readme] FFC API usage: "from forest_fire_clustering.forest_fire_clustering import FFC
cluster_obj = FFC()
cluster_obj.preprocess(X)"
- [discussion] Evaluation metrics: "There are three main ways to evaluate the performance of unsupervised methods. First, purity scores evaluate the cluster labels based on a set of reference labels."
