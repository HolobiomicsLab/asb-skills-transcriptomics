---
name: 2d-shape-clustering-reproducibility
description: Use when validating or reproducing the Forest Fire Clustering benchmarks
  on ring-shaped, half-moon-shaped, or circular Gaussian-mixture synthetic datasets,
  where GMM is expected to fail and FFC should achieve top purity.
when_to_use_negative:
- Data are real single-cell gene expression matrices requiring preprocessing (zheng17
  recipe, 30 principal components) rather than controlled 2D shape benchmarks
- The goal is quantitative internal validation of FFC results (validate(), entropy(),
  pval()) rather than cross-method benchmarking on shapes with known ground truth
- Datasets have intrinsically Gaussian/convex cluster shapes, where distribution-based
  methods like GMM are expected to work and the ring/moon failure-mode comparison
  is uninformative
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3316
tools:
- name: numpy
  role: Generates the Gaussian-mixture synthetic datasets centered on the unit circle
    and handles linear algebra
- name: scikit-learn (sklearn)
  role: Produces ring-shaped and half-moon-shaped synthetic data and supplies the
    KMeans, GaussianMixture, DBSCAN, OPTICS, and SpectralClustering baselines
- name: forest-fire-clustering (FFC)
  role: 'Clustering method under test: preprocess(X), fit(fire_temp=100), cluster_labels'
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: scipy
  role: Sparse matrix acceleration supporting the FFC pipeline
- name: numba
  role: Numerical acceleration for the FFC implementation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:d9c538190af8f2416e9e820a24049f880f634dc79a681fa1c51c3fb39101b396
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
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# 2d-shape-clustering-reproducibility

## Summary

A reproducibility workflow for benchmarking Forest Fire Clustering against KMeans, GaussianMixture, DBSCAN, OPTICS, and SpectralClustering on non-convex 2D synthetic data (Gaussian mixtures on a unit circle, ring-shaped and half-moon shapes from sklearn), scored by purity against generation labels.

## When to use

Use when validating or reproducing the Forest Fire Clustering benchmarks on ring-shaped, half-moon-shaped, or circular Gaussian-mixture synthetic datasets, where GMM is expected to fail and FFC should achieve top purity.

## When NOT to use

- Data are real single-cell gene expression matrices requiring preprocessing (zheng17 recipe, 30 principal components) rather than controlled 2D shape benchmarks
- The goal is quantitative internal validation of FFC results (validate(), entropy(), pval()) rather than cross-method benchmarking on shapes with known ground truth
- Datasets have intrinsically Gaussian/convex cluster shapes, where distribution-based methods like GMM are expected to work and the ring/moon failure-mode comparison is uninformative

## Inputs

- [sample x feature] synthetic dataset: Gaussian mixtures centered on the unit circle, sigma=0.15 (n=500) and sigma=0.2 (n=500), generated with numpy
- sklearn ring-shaped synthetic dataset (n=1000) with generation reference labels
- sklearn half-moon-shaped synthetic dataset (n=1000) with generation reference labels
- Fixed random seeds for dataset generation

## Outputs

- cluster_labels per method per dataset (FFC cluster_labels plus KMeans, GaussianMixture, DBSCAN, OPTICS, SpectralClustering labels)
- purity_scores.csv: table of purity scores (rows = methods, columns = datasets)

## How to apply

Generate the synthetic datasets with numpy and scikit-learn using fixed random seeds: Gaussian mixtures centered on the unit circle with sigma=0.15 (n=500) and sigma=0.2 (n=500), plus sklearn ring-shaped and half-moon-shaped data (n=1000 each), keeping the reference labels used for generation. Run Forest Fire Clustering on each [sample x feature] matrix via FFC().preprocess(X), cluster_obj.fit(fire_temp=100), and retrieve cluster_obj.cluster_labels. Run the five sklearn baselines (KMeans, GaussianMixture, DBSCAN, OPTICS, SpectralClustering) on the same data and retrieve each method's labels. Compute purity scores for every method on every dataset by comparing predicted labels to the generation reference labels, then assemble a methods-by-datasets table saved as purity_scores.csv. Validation hinges on matching the reported benchmarks: GMM purity of 0.5 on ring-shaped data and 0.85 on half-moon-shaped data, with FFC achieving the top scores.

## Related tools

- **numpy** (Generates the Gaussian-mixture synthetic datasets centered on the unit circle and handles linear algebra)
- **scikit-learn (sklearn)** (Produces ring-shaped and half-moon-shaped synthetic data and supplies the KMeans, GaussianMixture, DBSCAN, OPTICS, and SpectralClustering baselines)
- **forest-fire-clustering (FFC)** (Clustering method under test: preprocess(X), fit(fire_temp=100), cluster_labels) — https://github.com/gersteinlab/forest-fire-clustering
- **scipy** (Sparse matrix acceleration supporting the FFC pipeline)
- **numba** (Numerical acceleration for the FFC implementation)

## Examples

```
from forest_fire_clustering.forest_fire_clustering import FFC; cluster_obj = FFC(); cluster_obj.preprocess(X); cluster_obj.fit(fire_temp=100); labels = cluster_obj.cluster_labels
```

## Evaluation signals

- purity_scores.csv exists and is populated with a purity value for every method-dataset pair
- GMM purity on ring-shaped data matches the reported 0.5
- GMM purity on half-moon-shaped data matches the reported 0.85
- FFC achieves the top purity scores across the ring-shaped and half-moon-shaped datasets
- Fixed random seeds make re-running the generation and clustering pipeline reproduce the same labels and purity table

## Limitations

- Forest Fire Clustering should be performed with nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE) and is sensitive to high dimensionality because it mimics a low-dimensional fire propagation process
- The fire temperature c strongly influences the size and number of discovered clusters, acting like a resolution parameter (here fixed at fire_temp=100)
- Distribution-based methods like Gaussian mixture modeling cannot accurately categorize ring-shaped and half-moon-shaped data, so GMM rows serve as expected-failure controls rather than competitive baselines

## Evidence

- [readme] FFC usage for clustering: "cluster_obj = FFC()
cluster_obj.preprocess(X)
cluster_obj.fit(fire_temp=100)
cluster_obj.cluster_labels"
- [results] GMM failure on shape benchmarks: [evidence span withheld — anchor status: not_found]
- [discussion] purity as evaluation metric: "purity scores evaluate the cluster labels based on a set of reference labels"
- [discussion] nonlinear embedding requirement: "Forest Fire Clustering should be performed with nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE)"
- [discussion] fire temperature sensitivity: "The fire temperature c plays an important role in clustering and strongly influences the size and number of the clusters discovered"
