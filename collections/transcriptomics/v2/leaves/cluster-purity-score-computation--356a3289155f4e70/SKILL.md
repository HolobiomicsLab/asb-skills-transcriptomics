---
name: cluster-purity-score-computation
description: Use when you have predicted cluster labels plus reference labels used
  for data generation and need a single score per method to rank clustering performance,
  e.g., benchmarking on synthetic ring/half-moon data.
when_to_use_negative:
- No reference labels exist (e.g., purely exploratory clustering of new data) — purity
  is an external metric requiring ground truth; use internal validation like FFC's
  cluster_obj.validate(), entropy(), or pval() instead
- Data are in very high dimensions without nonlinear dimensionality reduction — FFC
  clustering that feeds the labels into purity is limited by the curse of dimensionality
  and should be performed after UMAP/t-SNE/PHATE
- The clustering produces only one cluster for the whole dataset — every point receives
  the same label and purity collapses to the majority-class fraction, carrying no
  cluster-quality signal
edam_operation: http://edamontology.org/operation_3432
tools:
- name: forest-fire-clustering (FFC)
  role: Produces cluster_labels via preprocess(X) and fit(fire_temp=100) whose purity
    is benchmarked against baselines
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: scikit-learn (sklearn)
  role: Supplies baseline clusterers (KMeans, GaussianMixture, DBSCAN, OPTICS, SpectralClustering)
    and synthetic ring/half-moon data generators whose labels are compared via purity
- name: numpy
  role: Generates synthetic Gaussian-mixture datasets and array operations for matching
    predicted labels to reference labels
- name: scipy
  role: Sparse matrix acceleration during preprocessing of the data fed to clustering
- name: numba
  role: Numerical acceleration of the clustering pipeline whose labels are scored
    by purity
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:30f311cf393901ef7f16d3c09af2394c72263347e6936162aa666e266bc8c40d
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
yes_rate: 0.75
n_claims: 12
---

# cluster-purity-score-computation

## Summary

Computes purity scores that evaluate predicted cluster labels against a set of reference (ground-truth) labels, the primary external-validation metric used to benchmark Forest Fire Clustering (FFC) against KMeans, GaussianMixture, DBSCAN, OPTICS, and SpectralClustering on synthetic Gaussian-mixture, ring-shaped, and half-moon-shaped datasets.

## When to use

Use when you have predicted cluster labels plus reference labels used for data generation and need a single score per method to rank clustering performance, e.g., benchmarking on synthetic ring/half-moon data.

## When NOT to use

- No reference labels exist (e.g., purely exploratory clustering of new data) — purity is an external metric requiring ground truth; use internal validation like FFC's cluster_obj.validate(), entropy(), or pval() instead
- Data are in very high dimensions without nonlinear dimensionality reduction — FFC clustering that feeds the labels into purity is limited by the curse of dimensionality and should be performed after UMAP/t-SNE/PHATE
- The clustering produces only one cluster for the whole dataset — every point receives the same label and purity collapses to the majority-class fraction, carrying no cluster-quality signal

## Inputs

- Predicted cluster label vector (one label per sample) from each clustering method, e.g., cluster_obj.cluster_labels or sklearn labels_
- Reference label vector used for synthetic data generation (ground truth)
- [sample x feature] data matrices of synthetic datasets (Gaussian mixtures sigma=0.15 and sigma=0.2, n=500; ring-shaped and half-moon-shaped data, n=1000) with fixed random seeds

## Outputs

- Purity score in [0, 1] for every (method, dataset) pair
- purity_scores.csv table with rows = clustering methods (FFC, KMeans, GaussianMixture, DBSCAN, OPTICS, SpectralClustering) and columns = datasets

## How to apply

Generate or load each dataset with fixed random seeds (e.g., Gaussian mixtures on the unit circle with sigma=0.15 and sigma=0.2, n=500 each; sklearn ring-shaped and half-moon-shaped data, n=1000 each) and retain the labels used for generation as the reference. Run each clustering method on the same [sample x feature] matrix — for FFC call FFC().preprocess(X), then cluster_obj.fit(fire_temp=100), and read cluster_obj.cluster_labels; for scikit-learn retrieve labels_ or fit-predict from KMeans, GaussianMixture, DBSCAN, OPTICS, and SpectralClustering. For every (method, dataset) pair, compute purity as the sum over clusters of the maximum count of the dominant reference label within that cluster, divided by the total number of samples, giving a value in [0, 1]. Assemble a table with rows as methods and columns as datasets and save as purity_scores.csv. Validate by checking the file reproduces the reported benchmark values — GMM purity of 0.5 on ring-shaped data and 0.85 on half-moon-shaped data — with FFC achieving the top scores.

## Related tools

- **forest-fire-clustering (FFC)** (Produces cluster_labels via preprocess(X) and fit(fire_temp=100) whose purity is benchmarked against baselines) — https://github.com/gersteinlab/forest-fire-clustering
- **scikit-learn (sklearn)** (Supplies baseline clusterers (KMeans, GaussianMixture, DBSCAN, OPTICS, SpectralClustering) and synthetic ring/half-moon data generators whose labels are compared via purity)
- **numpy** (Generates synthetic Gaussian-mixture datasets and array operations for matching predicted labels to reference labels)
- **scipy** (Sparse matrix acceleration during preprocessing of the data fed to clustering)
- **numba** (Numerical acceleration of the clustering pipeline whose labels are scored by purity)

## Examples

```
from sklearn.metrics.cluster import contingency_matrix; import numpy as np; purity = np.sum(np.amax(contingency_matrix(y_true, y_pred), axis=0)) / len(y_true)
```

## Evaluation signals

- purity_scores.csv exists and contains a numeric purity value in [0, 1] for every (method, dataset) pair
- GMM's purity equals 0.5 on ring-shaped data and 0.85 on half-moon-shaped data, matching the reported benchmark
- FFC achieves the top purity scores across the synthetic datasets (Gaussian mixtures, ring, half-moon)
- Same data matrix and fixed random seeds are used for all six methods, so score differences reflect method not data variation
- Table schema is rows = methods, columns = datasets, with all six clustering methods represented

## Limitations

- Purity is biased toward methods producing many small, pure clusters and does not penalize fragmentation
- Reproducibility of the broader benchmark depends on fixed random seeds for data generation; the paper notes the number of Monte Carlo trials varies with the number of clusters, so FFC runs are not fully fixed
- Reported reference values (e.g., GMM 0.5 on rings) apply to the specific sigma=0.15/0.2, n=500/1000 configurations; other data settings will yield different baselines

## Evidence

- [discussion] purity scores evaluate the cluster labels based on a set of reference labels: "There are three main ways to evaluate the performance of unsupervised methods. First, purity scores evaluate the cluster labels based on a set of reference labels."
- [discussion] Synthetic data produced by sklearn package: "Synthetic data of different shapes were produced by the sklearn package in Python"
- [full_text] GMM low purity on ring and half-moon data: [evidence span withheld — anchor status: not_found]
- [readme] README FFC usage producing cluster_labels: "cluster_obj = FFC()
cluster_obj.preprocess(X)
cluster_obj.fit(fire_temp=100)
cluster_obj.cluster_labels"
