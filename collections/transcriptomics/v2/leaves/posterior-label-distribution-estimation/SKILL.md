---
name: posterior-label-distribution-estimation
description: Use when FFC cluster labels (cluster_obj.cluster_labels after fit(fire_temp=100))
  need point-wise confidence quantification, e.g., posterior exclusion probabilities
  and label entropies to flag transition cells along developmental pseudo-time.
when_to_use_negative:
- Data contains only one cluster — every data point can only receive one type of label,
  so Monte Carlo validation cannot accurately estimate uncertainty.
- Clustering was performed without prior nonlinear dimensionality reduction (e.g.,
  UMAP, t-SNE, PHATE) — FFC is sensitive to high dimensionality.
- Input is raw gene expression counts not yet preprocessed (normalized/standardized
  with a recipe such as scanpy's zheng17) — run preprocess and fit first.
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: forest-fire-clustering (FFC)
  role: Provides validate(), pval(), and entropy() methods that run Monte Carlo re-propagation
    trials and emit pval_list and entropy_list
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: numpy
  role: Linear algebra operations underlying label propagation and Monte Carlo accumulation
- name: scipy
  role: Sparse matrix acceleration of affinity-graph label propagation
- name: numba
  role: Numerical acceleration / parallelization of the computationally intensive
    Monte Carlo trials
- name: scanpy
  role: sc.pp.neighbors KNN search and adaptive kernel used to build the affinity
    matrix A that re-propagation traverses
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:31b365134b890f1057f8b3de982d52bacb032154b7ac772822b9f2c724bb5a4a
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
  - scipy
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# posterior-label-distribution-estimation

## Summary

Internally validates Forest Fire Clustering (FFC) results by re-propagating retained seed labels across parallelized Monte Carlo trials to build a non-parametric posterior label distribution per cell, from which point-wise posterior exclusion probabilities (pval) and label entropies are derived.

## When to use

Use when FFC cluster labels (cluster_obj.cluster_labels after fit(fire_temp=100)) need point-wise confidence quantification, e.g., posterior exclusion probabilities and label entropies to flag transition cells along developmental pseudo-time.

## When NOT to use

- Data contains only one cluster — every data point can only receive one type of label, so Monte Carlo validation cannot accurately estimate uncertainty.
- Clustering was performed without prior nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE) — FFC is sensitive to high dimensionality.
- Input is raw gene expression counts not yet preprocessed (normalized/standardized with a recipe such as scanpy's zheng17) — run preprocess and fit first.

## Inputs

- Sample-by-feature matrix X (e.g., PCA-reduced scRNA-seq matrix, first 30 principal components after scanpy zheng17 preprocessing)
- Cluster labels cluster_obj.cluster_labels from FFC.fit(fire_temp=100)
- KNN-derived affinity matrix A produced by FFC.preprocess(X)

## Outputs

- cluster_obj.pval_list — point-wise posterior exclusion probability per cell (values in [0,1])
- cluster_obj.entropy_list — point-wise label entropy per cell
- validation_metrics.csv with entropy_list and pval_list side-by-side, one row per input cell

## How to apply

After FFC.fit(fire_temp=100) on the preprocessed sample-by-feature matrix, call cluster_obj.validate() to run repeated Monte Carlo trials: at trial t a random seed vertex r is selected and re-propagates its assigned label Sr across the KNN-derived affinity matrix until equilibrium. Accumulate per-cell label frequencies across trials to construct the non-parametric posterior label distribution for each data point; the number of trials varies with the number of clusters (more clusters need more trials) and is parallelized (numpy/scipy/numba). Then run cluster_obj.pval() to compute point-wise posterior exclusion probabilities into cluster_obj.pval_list, and cluster_obj.entropy() to compute point-wise label entropies into cluster_obj.entropy_list, where high entropy flags candidate transition cells along developmental pseudo-time. Write entropy_list and pval_list side-by-side (one row per cell) to validation_metrics.csv and check both fields lie in [0,1] for p-values.

## Related tools

- **forest-fire-clustering (FFC)** (Provides validate(), pval(), and entropy() methods that run Monte Carlo re-propagation trials and emit pval_list and entropy_list) — https://github.com/gersteinlab/forest-fire-clustering
- **numpy** (Linear algebra operations underlying label propagation and Monte Carlo accumulation)
- **scipy** (Sparse matrix acceleration of affinity-graph label propagation)
- **numba** (Numerical acceleration / parallelization of the computationally intensive Monte Carlo trials)
- **scanpy** (sc.pp.neighbors KNN search and adaptive kernel used to build the affinity matrix A that re-propagation traverses)

## Examples

```
from forest_fire_clustering.forest_fire_clustering import FFC; cluster_obj = FFC(); cluster_obj.preprocess(X); cluster_obj.fit(fire_temp=100); cluster_obj.validate(); cluster_obj.pval(); cluster_obj.entropy()
```

## Evaluation signals

- validation_metrics.csv exists with exactly one row per input cell and both entropy and p-value fields present.
- All pval_list values fall within [0,1] since they are posterior exclusion probabilities per cell.
- entropy_list values are non-negative and bounded by log(number of cluster labels); high-entropy points should localize to cluster boundaries/transition regions rather than cluster interiors.
- Posterior label distribution frequencies, accumulated across Monte Carlo trials, sum to 1 (or the trial count) per cell.
- High pval/entropy cells should be enriched near developmental pseudo-time transition zones consistent with label-entropy semantics described in the article.

## Limitations

- When there is only one cluster in the data, every data point can only receive one type of label, so Monte Carlo validation cannot accurately estimate uncertainty.
- The number of Monte Carlo trials varies with the number of clusters in the data (more clusters need more trials) and is not fixed, affecting reproducibility.
- A large number of Monte Carlo trials can be computationally intensive, though it can be efficiently parallelized (e.g., with numba).

## Evidence

- [readme] README validate workflow: "cluster_obj.validate()
cluster_obj.entropy()
cluster_obj.pval()

cluster_obj.entropy_list # list of entropies of the data point"
- [discussion] Monte Carlo trial mechanism: "In the tth Monte Carlo trial, a random seed vertex r is selected, and the seed vertex label Sr,t at trial t takes on the same label Sr"
- [abstract] Posterior distribution purpose: "calculates a non-parametric posterior probability that each cell is assigned a cell-type label"
- [abstract] Entropy highlights transitions: "enable the computation of “label entropies", highlighting transitions along developmental trajectories"
- [discussion] Single-cluster limitation: "When there is only one cluster in the data, every data point can only receive one type of label."
