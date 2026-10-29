---
name: heat-based-label-propagation-clustering
description: Use when you need to cluster a preprocessed sample-by-feature matrix
  (e.g., PBMC scRNA-seq reduced to 30 PCs) with a method that yields per-point posterior
  label confidences and works on a KNN affinity graph.
when_to_use_negative:
- 'Data contains only a single true cluster: Monte Carlo internal validation cannot
  estimate uncertainty because every point can only receive one type of label'
- 'Very high-dimensional data without nonlinear dimensionality reduction: the method
  is sensitive to high dimensionality and limited by the curse of dimensionality'
- Datasets over 1 million points where a full distance matrix is infeasible unless
  distances are approximated via KNN
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scanpy
  role: Provides sc.pp.neighbors for the KNN search and adaptive kernel used to build
    the affinity matrix from the data matrix
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: numpy
  role: Linear algebra operations during propagation
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: scipy
  role: Sparse matrix acceleration of the affinity graph
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: numba
  role: Numerical acceleration of the iterative propagation loop
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: scikit-learn (sklearn)
  role: General machine-learning utilities in the clustering implementation
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: Python
  role: Runtime environment for the forest-fire-clustering package (python >= 3.6)
  repo: https://github.com/gersteinlab/forest-fire-clustering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:0a906a59501686620a1e089c23cb2bb3e134d0817cee02d17204d280c792440e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gersteinlab/forest-fire-clustering
  tools:
  - Python
  - numba
  - numpy
  - scanpy
  - scikit-learn (sklearn)
  - scipy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# heat-based-label-propagation-clustering

## Summary

Cluster data by casting labels on seed vertices and propagating 'heat' influence across a KNN affinity graph until equilibrium, as implemented in Forest Fire Clustering (Forest Fire Clustering). Each vertex accepts a seed label when average influence exceeds its acceptance threshold (inverse of vertex degree).

## When to use

Use when you need to cluster a preprocessed sample-by-feature matrix (e.g., PBMC scRNA-seq reduced to 30 PCs) with a method that yields per-point posterior label confidences and works on a KNN affinity graph.

## When NOT to use

- Data contains only a single true cluster: Monte Carlo internal validation cannot estimate uncertainty because every point can only receive one type of label
- Very high-dimensional data without nonlinear dimensionality reduction: the method is sensitive to high dimensionality and limited by the curse of dimensionality
- Datasets over 1 million points where a full distance matrix is infeasible unless distances are approximated via KNN

## Inputs

- Preprocessed sample-by-feature matrix X (e.g., PBMC gene expression processed with scanpy 'zheng17' recipe, first 30 principal components)
- Affinity matrix A from KNN search (sc.pp.neighbors) with adaptive kernel, KNN-approximated L2 distances
- Fire temperature parameter c (e.g., fire_temp=100)

## Outputs

- Per-point cluster labels (cluster_obj.cluster_labels), exportable as cluster_labels.csv with one row per input sample
- Point-wise posterior label probabilities from internal validation (cluster_obj.validate())
- Point-wise label entropies (cluster_obj.entropy_list) and posterior significance values (cluster_obj.pval_list)

## How to apply

Load the preprocessed sample-by-feature matrix X (e.g., PBMC gene expression preprocessed with scanpy's 'zheng17' recipe, keeping the first 30 principal components). Compute the affinity matrix A from X using scanpy's 'sc.pp.neighbors' KNN search combined with an adaptive kernel, approximating the L2 distance matrix by distances between the KNNs of each data point. Initialize Forest Fire Clustering with fire_temp=100; a labeled seed vertex radiates influence equal to c × affinity to each unlabeled vertex, and Ti = 1/Di (inverse vertex degree) is the acceptance threshold. Iteratively cast a label on a seed vertex and propagate it through the affinity graph until equilibrium; deterministically select the next seed as the data point with the lowest heat from the previous cluster, repeating until every vertex is labeled. Write cluster_obj.cluster_labels to the output and verify that every data point receives exactly one cluster label.

## Related tools

- **scanpy** (Provides sc.pp.neighbors for the KNN search and adaptive kernel used to build the affinity matrix from the data matrix) — https://github.com/gersteinlab/forest-fire-clustering
- **numpy** (Linear algebra operations during propagation) — https://github.com/gersteinlab/forest-fire-clustering
- **scipy** (Sparse matrix acceleration of the affinity graph) — https://github.com/gersteinlab/forest-fire-clustering
- **numba** (Numerical acceleration of the iterative propagation loop) — https://github.com/gersteinlab/forest-fire-clustering
- **scikit-learn (sklearn)** (General machine-learning utilities in the clustering implementation) — https://github.com/gersteinlab/forest-fire-clustering
- **Python** (Runtime environment for the forest-fire-clustering package (python >= 3.6)) — https://github.com/gersteinlab/forest-fire-clustering

## Examples

```
from forest_fire_clustering.forest_fire_clustering import FFC; cluster_obj = FFC(); cluster_obj.preprocess(X); cluster_obj.fit(fire_temp=100); cluster_obj.cluster_labels
```

## Evaluation signals

- Every data point receives exactly one cluster label (all vertices are labeled at equilibrium)
- Output cluster_labels.csv contains one row per input sample
- Per-vertex acceptance threshold Ti = 1/Di is set from the vertex degree Di of the affinity matrix
- Validation produces posterior significance values (pval_list) and label entropies (entropy_list) for each data point
- Benchmark quality can be cross-checked with purity scores, ARI, and silhouette scores against reference labels

## Limitations

- The fire temperature c strongly influences the size and number of clusters discovered (resolution-like sensitivity)
- Limited by the curse of dimensionality; should be performed with nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE)
- For datasets over 1 million data points the distance matrix is computationally expensive and must be approximated via KNN distances

## Evidence

- [discussion] label propagation mechanism: [evidence span withheld — anchor status: not_found]
- [discussion] acceptance threshold: [evidence span withheld — anchor status: not_found]
- [discussion] affinity construction: [evidence span withheld — anchor status: not_found]
- [discussion] seed selection: "the next and subsequent seeds are deterministically chosen as the data point that experiences the lowest heat from the previous cluster"
- [readme] README usage: "cluster_obj = FFC()
cluster_obj.preprocess(X)
cluster_obj.fit(fire_temp=100)
cluster_obj.cluster_labels"
- [discussion] iteration to equilibrium: "we iteratively cast a label on a vertex and allow the label to propagate until an equilibrium is reached."
