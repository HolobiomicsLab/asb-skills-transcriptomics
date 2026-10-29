---
name: transition-cell-type-identification-along-pseudotime
description: Use when single-cell cluster labels have already been assigned (e.g.,
  via FFC.fit()) and the goal is to detect cells with ambiguous labels — candidate
  novel transition cell types — along a developmental pseudo-time manifold.
when_to_use_negative:
- 'When the data contain only one cluster: every data point can only receive one type
  of label, so the Monte Carlo validation cannot accurately estimate uncertainty'
- When clustering itself has not yet been performed — this skill validates existing
  FFC labels, not raw expression matrices
- 'When applied to very high-dimensional representations: FFC should be performed
  with nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE), and the curse
  of dimensionality degrades clustering'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: forest-fire-clustering (FFC)
  role: Provides the validate(), entropy(), and pval() methods that run the Monte
    Carlo trials and compute entropy_list and pval_list
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: numba
  role: Accelerates the parallelized Monte Carlo simulations of label propagation
- name: numpy
  role: Linear algebra operations for propagating seed labels across the affinity
    graph
- name: scipy
  role: Sparse matrix acceleration for the affinity graph during label propagation
- name: scanpy
  role: Preprocessing (zheng17 recipe, sc.pp.neighbors KNN/affinity matrix) upstream
    of clustering and validation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:518d329f9779e62010f4f0b2a00e302f1124f56ae4ce7913579639ef20d11d3b
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Transition cell type identification along pseudotime

## Summary

Identifies candidate transition cells along developmental pseudo-time by computing point-wise label entropies from Monte Carlo internal validation of Forest Fire Clustering, where high-entropy cells flag boundary/transition states. Posterior exclusion probabilities (p-values) are computed alongside to quantify per-cell label confidence.

## When to use

Use when single-cell cluster labels have already been assigned (e.g., via FFC.fit()) and the goal is to detect cells with ambiguous labels — candidate novel transition cell types — along a developmental pseudo-time manifold.

## When NOT to use

- When the data contain only one cluster: every data point can only receive one type of label, so the Monte Carlo validation cannot accurately estimate uncertainty
- When clustering itself has not yet been performed — this skill validates existing FFC labels, not raw expression matrices
- When applied to very high-dimensional representations: FFC should be performed with nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE), and the curse of dimensionality degrades clustering

## Inputs

- Preprocessed sample-by-feature matrix X (e.g., single-cell expression data preprocessed with the scanpy zheng17 recipe and first 30 principal components)
- Cluster labels from FFC.fit() (cluster_labels) and the affinity matrix from FFC.preprocess()

## Outputs

- cluster_obj.pval_list: per-cell posterior exclusion probability (significance) values in [0,1]
- cluster_obj.entropy_list: per-cell label entropies, high values flagging transition cells along pseudo-time
- Optional side-by-side export (e.g., validation_metrics.csv with one row per cell containing entropy and p-value)

## How to apply

After FFC.fit() (with fire_temp=100) assigns cluster labels, run cluster_obj.validate() to launch repeated Monte Carlo trials: in trial t a random seed vertex r is selected and re-propagates its retained label Sr across the affinity graph until equilibrium. Accumulate per-cell label frequencies across trials to build a non-parametric posterior label distribution for each cell. Then run cluster_obj.pval() to compute the point-wise posterior exclusion probability (stored in cluster_obj.pval_list) and cluster_obj.entropy() to compute point-wise label entropy (stored in cluster_obj.entropy_list). Cells with high label entropy localize to cluster boundaries and transition regions along pseudo-time, flagging them de novo as candidate transition cell types; low p-values indicate confident, non-excluded labels. The number of Monte Carlo trials scales with the number of clusters (more clusters need more trials), and computation is parallelized via numba.

## Related tools

- **forest-fire-clustering (FFC)** (Provides the validate(), entropy(), and pval() methods that run the Monte Carlo trials and compute entropy_list and pval_list) — https://github.com/gersteinlab/forest-fire-clustering
- **numba** (Accelerates the parallelized Monte Carlo simulations of label propagation)
- **numpy** (Linear algebra operations for propagating seed labels across the affinity graph)
- **scipy** (Sparse matrix acceleration for the affinity graph during label propagation)
- **scanpy** (Preprocessing (zheng17 recipe, sc.pp.neighbors KNN/affinity matrix) upstream of clustering and validation)

## Examples

```
from forest_fire_clustering.forest_fire_clustering import FFC; cluster_obj = FFC(); cluster_obj.preprocess(X); cluster_obj.fit(fire_temp=100); cluster_obj.validate(); cluster_obj.entropy(); cluster_obj.pval()
```

## Evaluation signals

- validation_metrics.csv (or equivalent) exists with exactly one row per input cell and both entropy and p-value fields present
- All pval_list values fall within [0, 1]
- High-entropy points localize to cluster boundaries/transition regions along pseudo-time rather than cluster interiors (qualitative check)
- Monte Carlo trial count is sufficient — it must scale with the number of clusters in the data (more clusters require more trials)
- Posterior distributions are non-parametric and constructed empirically from tabulated seed-label frequencies across Monte Carlo trials

## Limitations

- Monte Carlo internal validation cannot accurately estimate uncertainty when there is only one cluster in the data
- The number of Monte Carlo trials varies with the number of clusters (not fixed), hurting reproducibility, and a large number of trials is computationally intensive (though parallelizable)
- Forest Fire Clustering is sensitive to high dimensionality because fire propagation mimics a low-dimensional process; it should be performed with nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE)

## Evidence

- [readme] To validate the results: cluster_obj.validate() cluster_obj.entropy() cluster_obj.pval(): "To validate the results:

``ncluster_obj.validate()
cluster_obj.entropy()
cluster_obj.pval()

cluster_obj.entropy_list # list of entropies of the data point"
- [abstract] Non-parametric posterior probability and label entropies highlight transitions: "calculates a non-parametric posterior probability that each cell is assigned a cell-type label"
- [discussion] Monte Carlo trial seed vertex selection and label propagation: "In the tth Monte Carlo trial, a random seed vertex r is selected, and the seed vertex label Sr,t at trial t takes on the same label Sr"
- [discussion] Single-cluster limitation of Monte Carlo validation: "When there is only one cluster in the data, every data point can only receive one type of label."
- [discussion] Trial count varies with cluster number: "The number of Monte Carlo trials varies with the number of clusters in the data, as a larger number of clusters need more Monte Carlo trials"
- [discussion] Dimensionality reduction requirement: "should be performed with nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE)"
