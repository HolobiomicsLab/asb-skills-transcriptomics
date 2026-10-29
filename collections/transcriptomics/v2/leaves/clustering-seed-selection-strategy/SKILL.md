---
name: clustering-seed-selection-strategy
description: Use when clustering single-cell (e.g., PBMC scRNA-seq) data on a KNN/adaptive-kernel
  affinity matrix and a deterministic, assumption-light label-propagation procedure
  is needed that guarantees every vertex receives exactly one cluster label without
  a predefined cluster count.
when_to_use_negative:
- When the data has only one cluster in it — Monte Carlo internal validation cannot
  accurately estimate uncertainty since every data point can only receive one type
  of label
- When distances are dominated by high dimensionality — the method is sensitive to
  high dimensionality because it mimics a low-dimensional fire propagation process,
  and the curse of dimensionality degrades clustering as distances become more uniform
- When a fixed, reproducible number of validation trials is required — the number
  of Monte Carlo trials varies with the number of clusters in the data
edam_operation: http://edamontology.org/operation_3432
tools:
- name: scanpy
  role: Provides 'sc.pp.neighbors' for the KNN search combined with an adaptive kernel
    to obtain the affinity matrix A used for label propagation
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: forest-fire-clustering (FFC)
  role: 'Implements the label-propagation fit: cluster_obj.preprocess(X) then cluster_obj.fit(fire_temp=100);
    exposes cluster_obj.cluster_labels'
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: numpy
  role: Linear algebra operations underlying heat/affinity computation
- name: scipy
  role: Sparse matrix acceleration for the affinity graph
- name: numba
  role: Numerical acceleration of the propagation loop
- name: scikit-learn (sklearn)
  role: Supporting machine-learning utilities in the pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:5c2a56d711bc0fb8511cac0db61f6329909e638be2cf40a98a166d7f72179961
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gersteinlab/forest-fire-clustering
  tools:
  - Python
  - forest-fire-clustering (FFC)
  - numba
  - numpy
  - scanpy
  - scikit-learn (sklearn)
  - scipy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# clustering-seed-selection-strategy

## Summary

Deterministic seed-selection strategy from Forest Fire Clustering (Chen et al., Nature Communications 2022): labels are iteratively cast on a seed vertex and propagated through an affinity graph until equilibrium, and each subsequent seed is chosen as the data point that experienced the lowest heat from the previous cluster, repeating until every vertex is labeled.

## When to use

Use when clustering single-cell (e.g., PBMC scRNA-seq) data on a KNN/adaptive-kernel affinity matrix and a deterministic, assumption-light label-propagation procedure is needed that guarantees every vertex receives exactly one cluster label without a predefined cluster count.

## When NOT to use

- When the data has only one cluster in it — Monte Carlo internal validation cannot accurately estimate uncertainty since every data point can only receive one type of label
- When distances are dominated by high dimensionality — the method is sensitive to high dimensionality because it mimics a low-dimensional fire propagation process, and the curse of dimensionality degrades clustering as distances become more uniform
- When a fixed, reproducible number of validation trials is required — the number of Monte Carlo trials varies with the number of clusters in the data

## Inputs

- Sample-by-feature matrix X (e.g., PBMC gene expression data; first 30 principal components after 'zheng17' preprocessing)
- Affinity matrix A computed via scanpy 'sc.pp.neighbors' (KNN search + adaptive kernel)
- User-defined fire temperature parameter c (paper default: fire_temp=100)

## Outputs

- Per-point cluster labels (cluster_obj.cluster_labels), one label per input sample
- Optionally written to cluster_labels.csv (one row per input sample)

## How to apply

Preprocess the sample-by-feature matrix X (e.g., PBMC data preprocessed with scanpy's 'zheng17' recipe and the first 30 principal components) into an affinity matrix A using scanpy's 'sc.pp.neighbors' for the KNN search combined with an adaptive kernel, approximating the L2 distance matrix by distances between the KNNs of each data point. Initialize FFC with fire_temp=100; the per-vertex acceptance threshold Ti is set to the inverse of the vertex degree Di (Ti = 1/Di). A labeled seed vertex radiates heat computed as fire temperature c multiplied by the affinity between the seed and each vertex; a vertex accepts the seed's label when the average influence from labeled vertices exceeds its acceptance threshold. Iteratively cast a label on a vertex and allow the label to propagate until an equilibrium is reached, then deterministically choose the next and subsequent seeds as the data point that experiences the lowest heat from the previous cluster, repeating until every vertex is labeled. Rationale: this strategy makes minimal prior assumptions about the data shape and avoids random seed initialization, supporting rare cell-type discovery.

## Related tools

- **scanpy** (Provides 'sc.pp.neighbors' for the KNN search combined with an adaptive kernel to obtain the affinity matrix A used for label propagation) — https://github.com/gersteinlab/forest-fire-clustering
- **forest-fire-clustering (FFC)** (Implements the label-propagation fit: cluster_obj.preprocess(X) then cluster_obj.fit(fire_temp=100); exposes cluster_obj.cluster_labels) — https://github.com/gersteinlab/forest-fire-clustering
- **numpy** (Linear algebra operations underlying heat/affinity computation)
- **scipy** (Sparse matrix acceleration for the affinity graph)
- **numba** (Numerical acceleration of the propagation loop)
- **scikit-learn (sklearn)** (Supporting machine-learning utilities in the pipeline)

## Examples

```
from forest_fire_clustering.forest_fire_clustering import FFC; cluster_obj = FFC(); cluster_obj.preprocess(X); cluster_obj.fit(fire_temp=100); cluster_obj.cluster_labels
```

## Evaluation signals

- Every data point receives exactly one cluster label (all vertices are labeled) after propagation terminates
- Output label file (e.g., cluster_labels.csv) contains exactly one row per input sample
- Determinism check: re-running with the same affinity matrix reproduces the same seed order, since subsequent seeds are deterministically chosen as the data point with the lowest heat from the previous cluster
- Cluster granularity is consistent with fire_temp=100; note that c strongly influences the size and number of the clusters discovered, so deviations in fire temperature should shift cluster count
- Benchmark agreement via purity scores, ARI, and silhouette scores against reference labels

## Limitations

- The fire temperature c strongly influences the size and the number of the clusters discovered (resolution-like sensitivity)
- Sensitive to high dimensionality because the approach mimics a low-dimensional fire propagation process; should be performed with nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE)
- Limited by the curse of dimensionality as distances become more uniform in high dimensions
- Representing the distance matrix for large datasets (>1 million data points) is computationally expensive and must be approximated via KNN

## Evidence

- [discussion] Iteratively cast a label on a vertex and propagate until equilibrium; repeat with new labels until all vertices are labeled: "we iteratively cast a label on a vertex and allow the label to propagate until an equilibrium is reached."
- [discussion] Deterministic seed selection: subsequent seeds chosen as the data point with lowest heat from the previous cluster: "the next and subsequent seeds are deterministically chosen as the data point that experiences the lowest heat from the previous cluster"
- [discussion] Acceptance threshold is the inverse of vertex degree: [evidence span withheld — anchor status: not_found]
- [readme] FFC API for clustering execution: "cluster_obj.preprocess(X)
cluster_obj.fit(fire_temp=100)
cluster_obj.cluster_labels"
