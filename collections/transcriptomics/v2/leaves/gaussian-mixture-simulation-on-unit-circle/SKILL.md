---
name: gaussian-mixture-simulation-on-unit-circle
description: Use when benchmarking clustering algorithms on non-convex synthetic shapes
  (rings, half-moons) where ground-truth labels are available and purity scores against
  GMM, K-means, DBSCAN, OPTICS, or spectral clustering baselines are required.
when_to_use_negative:
- Data with convex, well-separated Gaussian-shaped clusters where distribution-based
  methods like GMM suffice — the benchmark exists to expose non-convex failure modes
- No reference (generation-time) labels exist, since purity scores are computed against
  the labels used for data generation
- High-dimensional real data without dimensionality reduction — Forest Fire Clustering
  is limited by the curse of dimensionality and should be preceded by nonlinear dimensionality
  reduction (e.g., UMAP, t-SNE, PHATE)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3306
tools:
- name: scikit-learn (sklearn)
  role: Produces the synthetic ring-shaped and half-moon-shaped datasets and supplies
    the KMeans, GaussianMixture, DBSCAN, OPTICS, and SpectralClustering baselines
  repo: gersteinlab/forest-fire-clustering
- name: numpy
  role: Generates the unit-circle Gaussian mixtures and handles linear algebra operations
  repo: gersteinlab/forest-fire-clustering
- name: scipy
  role: Sparse matrix acceleration for the clustering pipeline
  repo: gersteinlab/forest-fire-clustering
- name: numba
  role: Numerical acceleration of the FFC implementation
  repo: gersteinlab/forest-fire-clustering
- name: Forest Fire Clustering (FFC)
  role: 'Method under test: FFC().preprocess(X), fit(fire_temp=100), cluster_labels'
  repo: gersteinlab/forest-fire-clustering
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:6e6ac50d399ab3034e4be27dbc6360a92c500bf6ea03590f25b40c44cfd79529
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gersteinlab/forest-fire-clustering
  tools:
  - Forest Fire Clustering (FFC)
  - Python
  - numba
  - numpy
  - scikit-learn
  - scikit-learn (sklearn)
  - scipy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3306
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# gaussian-mixture-simulation-on-unit-circle

## Summary

Generate reproducible synthetic benchmark datasets — Gaussian mixtures centered on the unit circle (sigma=0.15 and sigma=0.2, n=500 each) plus sklearn ring-shaped and half-moon-shaped data (n=1000 each) — to benchmark clustering methods by purity score against generation-time reference labels.

## When to use

Use when benchmarking clustering algorithms on non-convex synthetic shapes (rings, half-moons) where ground-truth labels are available and purity scores against GMM, K-means, DBSCAN, OPTICS, or spectral clustering baselines are required.

## When NOT to use

- Data with convex, well-separated Gaussian-shaped clusters where distribution-based methods like GMM suffice — the benchmark exists to expose non-convex failure modes
- No reference (generation-time) labels exist, since purity scores are computed against the labels used for data generation
- High-dimensional real data without dimensionality reduction — Forest Fire Clustering is limited by the curse of dimensionality and should be preceded by nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE)

## Inputs

- Synthetic Gaussian mixture data centered on the unit circle with sigma=0.15 (n=500) and sigma=0.2 (n=500), generated with numpy using fixed random seeds
- sklearn ring-shaped dataset (n=1000) with reference labels
- sklearn half-moon-shaped dataset (n=1000) with reference labels
- [sample x feature] numpy matrix X passed to FFC().preprocess(X)

## Outputs

- cluster_labels from Forest Fire Clustering and label vectors from the five scikit-learn baselines
- Purity score table saved as purity_scores.csv (rows = clustering methods, columns = synthetic datasets)

## How to apply

Generate the synthetic datasets with numpy and scikit-learn using fixed random seeds: Gaussian mixtures centered on the unit circle with sigma=0.15 (n=500) and sigma=0.2 (n=500), plus sklearn ring-shaped and half-moon-shaped data (n=1000 each). Run Forest Fire Clustering on the [sample x feature] matrix via FFC().preprocess(X), fit(fire_temp=100), and read cluster_obj.cluster_labels; run the five scikit-learn baselines (KMeans, GaussianMixture, DBSCAN, OPTICS, SpectralClustering) on identical data. Compute purity scores for every method by comparing predicted labels to the reference labels used for data generation. Assemble a table (rows = methods, columns = datasets) and save as purity_scores.csv; the rationale is that ring/half-moon geometry exposes the failure of distribution-based assumptions like Gaussian mixtures.

## Related tools

- **scikit-learn (sklearn)** (Produces the synthetic ring-shaped and half-moon-shaped datasets and supplies the KMeans, GaussianMixture, DBSCAN, OPTICS, and SpectralClustering baselines) — gersteinlab/forest-fire-clustering
- **numpy** (Generates the unit-circle Gaussian mixtures and handles linear algebra operations) — gersteinlab/forest-fire-clustering
- **scipy** (Sparse matrix acceleration for the clustering pipeline) — gersteinlab/forest-fire-clustering
- **numba** (Numerical acceleration of the FFC implementation) — gersteinlab/forest-fire-clustering
- **Forest Fire Clustering (FFC)** (Method under test: FFC().preprocess(X), fit(fire_temp=100), cluster_labels) — gersteinlab/forest-fire-clustering

## Examples

```
from sklearn.datasets import make_moons; X, y = make_moons(n_samples=1000, random_state=42)
```

## Evaluation signals

- purity_scores.csv exists and contains purity values for every method-dataset pair
- GMM purity is ~0.5 on ring-shaped data and ~0.85 on half-moon-shaped data, matching the reported benchmark
- Forest Fire Clustering achieves the top purity scores across the synthetic datasets
- Re-running with the same fixed random seeds regenerates identical datasets (n=500 and n=1000 per dataset)
- Purity is computed against the reference labels used for data generation, not against any method's output

## Limitations

- Distribution-based methods like Gaussian mixture modeling cannot accurately categorize ring/half-moon datasets (purity 0.5 and 0.85), so this benchmark should not be interpreted as evidence about GMM performance on convex data
- The number of Monte Carlo trials in FFC validation varies with the number of clusters, and a large number of trials can be computationally intensive (though parallelizable)
- Fire temperature c (set to 100 here) strongly influences the size and number of discovered clusters, so benchmark results are resolution-sensitive

## Evidence

- [discussion] sklearn synthetic shapes: "Synthetic data of different shapes were produced by the sklearn package in Python"
- [discussion] purity evaluation: "There are three main ways to evaluate the performance of unsupervised methods. First, purity scores evaluate the cluster labels based on a set of reference labels."
- [discussion] GMM failure on non-convex shapes: "distribution-based methods like Gaussian mixture modeling could not accurately categorize these datasets"
- [readme] FFC invocation: "from forest_fire_clustering.forest_fire_clustering import FFC
cluster_obj = FFC()
cluster_obj.preprocess(X)"
