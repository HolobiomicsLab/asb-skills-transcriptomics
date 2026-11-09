---
name: synthetic-benchmark-dataset-generation
description: 'Use when you must compare clustering algorithms under known ground truth:
  non-convex low-dimensional shapes (ring, half-moon) or Gaussian mixtures where reference
  labels are available for purity computation.'
when_to_use_negative:
- Input is real experimental single-cell data needing biological interpretation —
  synthetic shape benchmarks serve only for method evaluation, not discovery
- The goal is realistic simulated scRNA-seq count data with gene-expression properties
  — use Splatter simulation instead of geometric shape datasets
- Data are high-dimensional without prior nonlinear dimensionality reduction, where
  geometric benchmark intuition (ring/half-moon separability) does not transfer
edam_operation: http://edamontology.org/operation_3432
tools:
- name: scikit-learn (sklearn)
  role: Produces ring-shaped and half-moon-shaped synthetic data (make_moons and related
    generators) and supplies KMeans, GaussianMixture, DBSCAN, OPTICS, and SpectralClustering
    baselines
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: numpy
  role: Generates the unit-circle Gaussian mixture datasets (sigma=0.15 and sigma=0.2)
    and linear algebra for the benchmark matrices
- name: forest-fire-clustering (FFC)
  role: Method under benchmark; clusters via preprocess(X) then fit(fire_temp=100),
    exposing cluster_labels for purity scoring
  repo: https://github.com/gersteinlab/forest-fire-clustering
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:5f08b82a00741a9e3ffe8b99ad095fa228a5a2ab4269083e98fa361ac96170a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gersteinlab/forest-fire-clustering
  tools:
  - Python
  - forest-fire-clustering (FFC)
  - numba
  - numpy
  - scikit-learn
  - scikit-learn (sklearn)
  - scipy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# synthetic-benchmark-dataset-generation

## Summary

Generates controlled synthetic datasets (Gaussian mixtures on a ring, ring-shaped and half-moon shapes) with known reference labels to benchmark clustering methods such as Forest Fire Clustering against K-means, GMM, DBSCAN, OPTICS, and spectral clustering using purity scores.

## When to use

Use when you must compare clustering algorithms under known ground truth: non-convex low-dimensional shapes (ring, half-moon) or Gaussian mixtures where reference labels are available for purity computation.

## When NOT to use

- Input is real experimental single-cell data needing biological interpretation — synthetic shape benchmarks serve only for method evaluation, not discovery
- The goal is realistic simulated scRNA-seq count data with gene-expression properties — use Splatter simulation instead of geometric shape datasets
- Data are high-dimensional without prior nonlinear dimensionality reduction, where geometric benchmark intuition (ring/half-moon separability) does not transfer

## Inputs

- Random seed(s) for reproducible generation
- Sample sizes: n=500 for unit-circle Gaussian mixtures (sigma=0.15 and sigma=0.2), n=1000 for sklearn ring-shaped and half-moon-shaped data
- [sample x feature] matrices passed to each clustering method

## Outputs

- Synthetic datasets with reference labels used for generation
- Cluster labels from each method (FFC cluster_labels plus KMeans, GaussianMixture, DBSCAN, OPTICS, SpectralClustering labels)
- Purity score table (rows = methods, columns = datasets), e.g., purity_scores.csv

## How to apply

Generate datasets with numpy and scikit-learn using fixed random seeds: Gaussian mixtures centered on the unit circle with sigma=0.15 (n=500) and sigma=0.2 (n=500), plus sklearn ring-shaped and half-moon-shaped data (n=1000 each). Run each clustering method (FFC via FFC().preprocess(X) then fit(fire_temp=100); scikit-learn KMeans, GaussianMixture, DBSCAN, OPTICS, SpectralClustering) on the same [sample x feature] matrices and retrieve cluster labels. Compute purity by comparing predicted labels to the reference labels used at generation time, and assemble a methods x datasets table (e.g., purity_scores.csv). The rationale is that distribution-based methods like GMM make strong shape assumptions and fail on ring/half-moon data, so synthetic shapes expose weaknesses that simulated scRNA-seq alone may not.

## Related tools

- **scikit-learn (sklearn)** (Produces ring-shaped and half-moon-shaped synthetic data (make_moons and related generators) and supplies KMeans, GaussianMixture, DBSCAN, OPTICS, and SpectralClustering baselines) — https://github.com/gersteinlab/forest-fire-clustering
- **numpy** (Generates the unit-circle Gaussian mixture datasets (sigma=0.15 and sigma=0.2) and linear algebra for the benchmark matrices)
- **forest-fire-clustering (FFC)** (Method under benchmark; clusters via preprocess(X) then fit(fire_temp=100), exposing cluster_labels for purity scoring) — https://github.com/gersteinlab/forest-fire-clustering

## Examples

```
from sklearn.datasets import make_moons; from forest_fire_clustering.forest_fire_clustering import FFC; X, y = make_moons(n_samples=1000, random_state=seed); obj = FFC(); obj.preprocess(X); obj.fit(fire_temp=100); labels = obj.cluster_labels
```

## Evaluation signals

- purity_scores.csv exists and contains a methods x datasets purity table
- GMM purity matches reported values: 0.5 on ring-shaped data and 0.85 on half-moon-shaped data
- Forest Fire Clustering achieves the top purity scores across the synthetic datasets
- All datasets are generated with fixed random seeds, so reruns reproduce the same matrices and scores
- Purity is computed against the reference labels used during data generation, not against another method's labels

## Limitations

- Gaussian mixture models cannot accurately categorize ring-shaped and half-moon-shaped data (low purity), so these benchmarks penalize distribution-based shape assumptions
- The synthetic shapes are low-dimensional; Forest Fire Clustering is sensitive to high dimensionality because fire propagation mimics a low-dimensional process and suffers from the curse of dimensionality
- FFC benchmarks require nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE) for realistic single-cell embeddings, which geometric shape datasets do not capture

## Evidence

- [discussion] sklearn synthetic shapes: "Synthetic data of different shapes were produced by the sklearn package in Python"
- [discussion] Splatter simulated data: "Simulated scRNA-seq data were generated using the R package Splatter."
- [discussion] purity as reference-label evaluation: "First, purity scores evaluate the cluster labels based on a set of reference labels."
- [readme] FFC preprocessing and fit invocation: "cluster_obj = FFC()
cluster_obj.preprocess(X)
cluster_obj.fit(fire_temp=100)
cluster_obj.cluster_labels"
