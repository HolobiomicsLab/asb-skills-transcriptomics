---
name: cell-label-entropy-computation
description: Use when FFC cluster labels exist (fire_temp=100) and you need per-cell
  label uncertainty — e.g., to flag candidate transition cells along a pseudotime
  trajectory or to quantify clustering confidence in scRNA-seq data.
when_to_use_negative:
- When the data contains only one cluster — every point can only receive one type
  of label, so Monte Carlo validation cannot accurately estimate uncertainty
- When clustering raw gene counts without nonlinear dimensionality reduction or FFC
  preprocessing — the affinity graph and label propagation require preprocessed features
  and affinities
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: forest-fire-clustering
  role: Provides FFC.validate(), FFC.entropy(), and FFC.pval() that run the Monte
    Carlo trials and populate entropy_list and pval_list
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: numpy
  role: Linear algebra operations for label propagation and accumulation of per-cell
    label frequencies
- name: scipy
  role: Sparse matrix acceleration of the affinity graph and Spearman correlation
    of entropy vs pseudotime (scipy.stats)
- name: numba
  role: Numerical acceleration of the parallelized Monte Carlo validation trials
- name: scanpy
  role: Preprocesses PBMC/MCA count matrices with the 'zheng17' recipe and computes
    diffusion pseudotime rooted at Leiden clusters
- name: Splatter
  role: Generates simulated scRNA-seq differentiation paths with known pseudotime
    for entropy–pseudotime validation
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:0d9d05eee2bfd817b9dceeab89ada5a6a4c926f2f6e33bd3bce9ec07cd2cdef9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gersteinlab/forest-fire-clustering
  tools:
  - Python
  - R
  - Splatter
  - forest-fire-clustering
  - numba
  - numpy
  - scanpy
  - scipy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# cell-label-entropy-computation

## Summary

Computes point-wise label entropies (and posterior exclusion probabilities) for Forest Fire Clustering labels by running parallelized Monte Carlo internal validation, in which seed labels are re-propagated across trials to build an empirical posterior label distribution per cell; high entropy flags transition cells along developmental pseudo-time.

## When to use

Use when FFC cluster labels exist (fire_temp=100) and you need per-cell label uncertainty — e.g., to flag candidate transition cells along a pseudotime trajectory or to quantify clustering confidence in scRNA-seq data.

## When NOT to use

- When the data contains only one cluster — every point can only receive one type of label, so Monte Carlo validation cannot accurately estimate uncertainty
- When clustering raw gene counts without nonlinear dimensionality reduction or FFC preprocessing — the affinity graph and label propagation require preprocessed features and affinities

## Inputs

- Preprocessed sample-by-feature scRNA-seq expression matrix X (e.g., PBMC or MCA counts preprocessed with scanpy 'zheng17' recipe, first 30 principal components)
- Cluster labels from FFC.fit(fire_temp=100) (cluster_obj.cluster_labels)
- KNN-derived affinity graph from FFC preprocess (scipy sparse affinity matrix)
- Optional pseudotime values (Splatter simulated paths or scanpy diffusion pseudotime rooted at Leiden clusters) for correlation analysis

## Outputs

- cluster_obj.entropy_list: point-wise label entropy per cell
- cluster_obj.pval_list: point-wise posterior exclusion probability per cell (values in [0,1])
- Empirical posterior label distribution per cell from tabulated seed labels across Monte Carlo trials
- Entropy–pseudotime Spearman correlation table (e.g., entropy_pseudotime_correlation.csv or validation_metrics.csv with one row per cell)

## How to apply

After FFC().preprocess(X) and FFC().fit(fire_temp=100) on the sample-by-feature matrix, call cluster_obj.validate() to launch repeated Monte Carlo trials: in trial t a random seed vertex r is selected and its label Sr is re-propagated across the affinity graph until equilibrium. Accumulate per-cell label frequencies across trials to construct the non-parametric posterior label distribution for each data point, then call cluster_obj.entropy() (retrieved from cluster_obj.entropy_list) and cluster_obj.pval() (retrieved from cluster_obj.pval_list) for point-wise label entropy and posterior exclusion probability. The number of Monte Carlo trials scales with the number of clusters; computation is parallelized with numba/numpy on the scipy sparse affinity matrix. Interpret high-entropy cells as candidate transition states: in Splatter-simulated differentiation paths and PBMC cell types, entropy correlates negatively (Spearman) with diffusion pseudotime, with progenitors showing high entropy and differentiated cells low entropy.

## Related tools

- **forest-fire-clustering** (Provides FFC.validate(), FFC.entropy(), and FFC.pval() that run the Monte Carlo trials and populate entropy_list and pval_list) — https://github.com/gersteinlab/forest-fire-clustering
- **numpy** (Linear algebra operations for label propagation and accumulation of per-cell label frequencies)
- **scipy** (Sparse matrix acceleration of the affinity graph and Spearman correlation of entropy vs pseudotime (scipy.stats))
- **numba** (Numerical acceleration of the parallelized Monte Carlo validation trials)
- **scanpy** (Preprocesses PBMC/MCA count matrices with the 'zheng17' recipe and computes diffusion pseudotime rooted at Leiden clusters)
- **Splatter** (Generates simulated scRNA-seq differentiation paths with known pseudotime for entropy–pseudotime validation)

## Examples

```
from forest_fire_clustering.forest_fire_clustering import FFC; cluster_obj = FFC(); cluster_obj.preprocess(X); cluster_obj.fit(fire_temp=100); cluster_obj.validate(); cluster_obj.entropy(); print(cluster_obj.entropy_list)
```

## Evaluation signals

- validation_metrics.csv exists with one row per input cell and both entropy and p-value fields present
- All posterior exclusion probabilities in pval_list fall within [0,1]
- Within each cell type or Splatter path, the Spearman correlation between entropy_list and pseudotime is negative
- High-entropy points localize to cluster boundaries/transition regions rather than cluster interiors, and match progenitor cells identified by diffusion pseudotime
- entropy_list length equals the number of input cells in X

## Limitations

- When there is only one cluster in the data, every data point can only receive one type of label, so Monte Carlo validation cannot accurately estimate uncertainty
- The number of Monte Carlo trials varies with the number of clusters in the data (more clusters need more trials), making trial counts non-fixed across datasets
- A large number of Monte Carlo trials can be computationally intensive, though the computation can be efficiently parallelized
- The method is sensitive to high dimensionality (curse of dimensionality) and should be performed with nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE)

## Evidence

- [readme] readme validate/entropy API: "cluster_obj.validate()
cluster_obj.entropy()
cluster_obj.pval()

cluster_obj.entropy_list # list of entropies of the data point"
- [abstract] posterior label entropies highlight transitions: "enable the computation of "label entropies", highlighting transitions along developmental trajectories"
- [discussion] Monte Carlo trial seed re-propagation: "In the tth Monte Carlo trial, a random seed vertex r is selected, and the seed vertex label Sr,t at trial t takes on the same label Sr"
- [discussion] single-cluster limitation: "When there is only one cluster in the data, every data point can only receive one type of label."
- [discussion] nonlinear dimensionality reduction requirement: "Hence, Forest Fire Clustering should be performed with nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE)"
