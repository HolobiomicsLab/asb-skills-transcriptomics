---
name: monte-carlo-label-propagation-validation
description: Use when a fitted FFC model's cluster_labels need per-cell confidence
  measures — posterior exclusion probabilities and label entropies — from Monte Carlo
  re-propagation on a single-cell sample-by-feature matrix.
when_to_use_negative:
- When the data contains only one cluster — every point can only receive one label,
  so the Monte Carlo internal validation cannot accurately estimate uncertainty
- When the goal is external validation against reference labels (purity, ARI, silhouette)
  rather than internal per-cell confidence estimation
- When working with very high-dimensional representations — FFC mimics a low-dimensional
  fire propagation process and should be performed after nonlinear dimensionality
  reduction (e.g., UMAP, t-SNE, PHATE)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: forest-fire-clustering (FFC)
  role: Provides the validate(), pval(), and entropy() methods that run Monte Carlo
    re-propagation and compute posterior exclusion probabilities and label entropies
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: numpy
  role: Linear algebra operations underlying affinity propagation and label frequency
    accumulation
- name: scipy
  role: Sparse matrix acceleration of the affinity graph used during label re-propagation
- name: numba
  role: Numerical acceleration to parallelize the computationally intensive Monte
    Carlo trials
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:6b0a783697f3fd331d0ef08634c26975b00409357efdf2c09984b0af5338a018
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gersteinlab/forest-fire-clustering
  tools:
  - Python
  - forest-fire-clustering (FFC)
  - numba
  - numpy
  - scipy
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# monte-carlo-label-propagation-validation

## Summary

Internal validation procedure from Forest Fire Clustering (FFC) that repeatedly re-propagates a randomly selected seed vertex's cluster label across the affinity graph over many Monte Carlo trials, builds a non-parametric posterior label distribution per cell, and derives point-wise posterior exclusion probabilities (pval) and label entropies.

## When to use

Use when a fitted FFC model's cluster_labels need per-cell confidence measures — posterior exclusion probabilities and label entropies — from Monte Carlo re-propagation on a single-cell sample-by-feature matrix.

## When NOT to use

- When the data contains only one cluster — every point can only receive one label, so the Monte Carlo internal validation cannot accurately estimate uncertainty
- When the goal is external validation against reference labels (purity, ARI, silhouette) rather than internal per-cell confidence estimation
- When working with very high-dimensional representations — FFC mimics a low-dimensional fire propagation process and should be performed after nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE)

## Inputs

- Preprocessed sample-by-feature matrix X (e.g., first 30 principal components of a scanpy 'zheng17'-preprocessed scRNA-seq count matrix) already loaded via cluster_obj.preprocess(X)
- Cluster labels emitted by cluster_obj.fit(fire_temp=100) (cluster_obj.cluster_labels)
- Affinity graph constructed from KNN L2 distances with an adaptive kernel (via scanpy sc.pp.neighbors)

## Outputs

- cluster_obj.pval_list: point-wise posterior exclusion probability per cell
- cluster_obj.entropy_list: point-wise label entropy per cell (high entropy marks transition cells)
- Empirical non-parametric posterior label distribution over cluster labels at each data point
- Optionally a per-cell table (one row per cell) of entropy_list and pval_list side-by-side (e.g., validation_metrics.csv)

## How to apply

After running FFC.preprocess(X) and FFC.fit(fire_temp=100) to obtain cluster_labels, call cluster_obj.validate() to launch repeated Monte Carlo trials: at trial t a random seed vertex r is selected and its assigned label Sr is re-propagated across the affinity graph until equilibrium. Accumulate per-cell label frequencies across trials to construct the non-parametric posterior label distribution for each data point. Then call cluster_obj.pval() to compute the point-wise posterior exclusion probability per cell (stored in cluster_obj.pval_list) and cluster_obj.entropy() to compute the point-wise label entropy (stored in cluster_obj.entropy_list), where high entropy flags candidate transition cells along developmental pseudo-time. Numba and scipy parallelize/accelerate the many trials, since a large number of Monte Carlo trials — which scales with the number of clusters — is computationally intensive.

## Related tools

- **forest-fire-clustering (FFC)** (Provides the validate(), pval(), and entropy() methods that run Monte Carlo re-propagation and compute posterior exclusion probabilities and label entropies) — https://github.com/gersteinlab/forest-fire-clustering
- **numpy** (Linear algebra operations underlying affinity propagation and label frequency accumulation)
- **scipy** (Sparse matrix acceleration of the affinity graph used during label re-propagation)
- **numba** (Numerical acceleration to parallelize the computationally intensive Monte Carlo trials)

## Examples

```
from forest_fire_clustering.forest_fire_clustering import FFC; cluster_obj = FFC(); cluster_obj.preprocess(X); cluster_obj.fit(fire_temp=100); cluster_obj.validate(); cluster_obj.entropy(); cluster_obj.pval()
```

## Evaluation signals

- validation_metrics.csv exists with one row per input cell and both entropy and p-value fields present
- pval_list values fall within [0,1] as posterior exclusion probabilities
- High-entropy points localize to cluster boundaries/transition regions along developmental pseudo-time rather than cluster interiors
- Number of Monte Carlo trials adapts to the number of clusters in the data (more clusters require more trials)
- pval_list and entropy_list are populated as Python lists with length equal to the number of cells in X

## Limitations

- Monte Carlo internal validation cannot accurately estimate uncertainty when there is only one cluster in the data, since every data point can only receive one type of label
- The number of Monte Carlo trials varies with the number of clusters and is not fixed, creating a reproducibility gap; a large number of trials is computationally intensive though parallelizable
- FFC is sensitive to high dimensionality (curse of dimensionality) and should be performed with nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE)

## Evidence

- [discussion] Monte Carlo internal validation: re-propagate seed labels across trials to build posterior label distributions: "In the tth Monte Carlo trial, a random seed vertex r is selected, and the seed vertex label Sr,t at trial t takes on the same label Sr"
- [methods] Point-wise statistics derived from the posterior: "cluster_obj.validate()
cluster_obj.entropy()
cluster_obj.pval()"
- [abstract] Entropy highlights transitions along developmental trajectories: "enable the computation of “label entropies", highlighting transitions along developmental trajectories"
- [discussion] Single-cluster failure mode: "When there is only one cluster in the data, every data point can only receive one type of label."
- [discussion] Trial count scales with cluster number: "The number of Monte Carlo trials varies with the number of clusters in the data, as a larger number of clusters need more Monte Carlo trials"
- [readme] README usage of validation outputs: "cluster_obj.entropy_list # list of entropies of the data point
cluster_obj.pval_list # list of posterior significance values"
