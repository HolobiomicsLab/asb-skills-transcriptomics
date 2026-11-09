---
name: monte-carlo-clustering-validation
description: Use when a single-cell clustering result needs per-cell confidence or
  uncertainty estimates, e.g. to flag transition states along developmental pseudotime
  or to internally validate cluster labels without external ground truth.
when_to_use_negative:
- Data contains only a single cluster — every point can receive only one label type,
  so Monte Carlo validation cannot accurately estimate uncertainty
- Input is a raw count matrix that has not yet been clustered (no fitted FFC object
  with cluster_labels exists to validate)
- The representation is high-dimensional — validation inherits the clustering's sensitivity
  to high dimensionality, so nonlinear dimensionality reduction (UMAP, t-SNE, PHATE)
  should precede it
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: forest-fire-clustering
  role: Provides the FFC class whose validate(), entropy(), and pval() methods implement
    the Monte Carlo internal validation and point-wise uncertainty outputs
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: scanpy
  role: Preprocesses PBMC RNA count matrices with the zheng17 recipe and computes
    diffusion pseudotime rooted at Leiden clusters for correlating with entropy_list
- name: scipy
  role: Sparse matrix acceleration and Spearman correlation between label entropies
    and pseudotime
- name: numba
  role: Numerical acceleration/parallelization of the Monte Carlo trials
- name: Splatter
  role: Generates simulated scRNA-seq differentiation paths with known pseudotime
    used as ground truth for the entropy-vs-pseudotime check
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:7a82cb10cccad64ca58967c82369a3371ffd46565627fd0d638a4cd0f29fc7cd
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# monte-carlo-clustering-validation

## Summary

Runs Forest Fire Clustering's Monte Carlo internal validation (cluster_obj.validate()) to build posterior label distributions over re-propagated seed labels, then extracts point-wise label entropies (cluster_obj.entropy(), entropy_list) and posterior significance values (cluster_obj.pval(), pval_list) for each cell.

## When to use

Use when a single-cell clustering result needs per-cell confidence or uncertainty estimates, e.g. to flag transition states along developmental pseudotime or to internally validate cluster labels without external ground truth.

## When NOT to use

- Data contains only a single cluster — every point can receive only one label type, so Monte Carlo validation cannot accurately estimate uncertainty
- Input is a raw count matrix that has not yet been clustered (no fitted FFC object with cluster_labels exists to validate)
- The representation is high-dimensional — validation inherits the clustering's sensitivity to high dimensionality, so nonlinear dimensionality reduction (UMAP, t-SNE, PHATE) should precede it

## Inputs

- Fitted forest_fire_clustering.FFC object (after preprocess(X) and fit(fire_temp=100)) on a [sample x feature] expression matrix (e.g., scanpy-zheng17-preprocessed PBMC counts or Splatter-simulated scRNA-seq)
- Optional per-cell reference coordinates such as diffusion pseudotime or Splatter pseudotime values for downstream entropy-vs-pseudotime correlation

## Outputs

- cluster_obj.entropy_list: point-wise label entropies, one per cell
- cluster_obj.pval_list: point-wise posterior significance values
- Optional correlation table (e.g., entropy_pseudotime_correlation.csv with columns dataset, cell type/path, Spearman correlation)

## How to apply

After fitting FFC (cluster_obj.preprocess(X); cluster_obj.fit(fire_temp=100)), run cluster_obj.validate() to execute parallelized Monte Carlo trials in which a random seed vertex r is selected per trial and its label Sr is re-propagated until equilibrium, yielding a posterior probability that each cell holds each cluster label. Then call cluster_obj.entropy() to compute point-wise label entropies, retrieved as cluster_obj.entropy_list, and optionally cluster_obj.pval() for posterior significance values in cluster_obj.pval_list. Rationale: high entropy marks cells receiving mixed labels across trials, so entropy peaks highlight transition states along trajectories — in the source analysis, Spearman correlation of entropy_list with Splatter pseudotime and scanpy diffusion pseudotime (zheng17 preprocessing, Leiden roots at Procr+ clusters) was negative for every path and PBMC cell type. The number of Monte Carlo trials scales with the number of clusters since more clusters require more trials; computation is parallelized with numba/scipy.

## Related tools

- **forest-fire-clustering** (Provides the FFC class whose validate(), entropy(), and pval() methods implement the Monte Carlo internal validation and point-wise uncertainty outputs) — https://github.com/gersteinlab/forest-fire-clustering
- **scanpy** (Preprocesses PBMC RNA count matrices with the zheng17 recipe and computes diffusion pseudotime rooted at Leiden clusters for correlating with entropy_list)
- **scipy** (Sparse matrix acceleration and Spearman correlation between label entropies and pseudotime)
- **numba** (Numerical acceleration/parallelization of the Monte Carlo trials)
- **Splatter** (Generates simulated scRNA-seq differentiation paths with known pseudotime used as ground truth for the entropy-vs-pseudotime check)

## Examples

```
from forest_fire_clustering.forest_fire_clustering import FFC; c = FFC(); c.preprocess(X); c.fit(fire_temp=100); c.validate(); c.entropy(); c.pval(); E = c.entropy_list
```

## Evaluation signals

- entropy_list and pval_list each contain exactly one value per input cell (same length as X rows) after validate() completes
- All Spearman correlations between entropy_list and pseudotime are negative for each Splatter path and each PBMC cell type, mirroring Figs. 4d-f and 5i
- Cells with highest entropy coincide with progenitor/transition cells identified independently by diffusion pseudotime (Procr-rooted Leiden clusters)
- Pval_list values are valid posterior significance probabilities (bounded in [0,1]) across trials

## Limitations

- The number of Monte Carlo trials varies with the number of clusters (more clusters need more trials) and is not fixed, complicating runtime reproducibility
- A large number of Monte Carlo trials can be computationally intensive, though it can be efficiently parallelized
- When there is only one cluster in the data, every data point receives one label type, so the Monte Carlo validation cannot accurately estimate uncertainty
- The fire temperature c (set to 100 here) strongly influences the size and number of discovered clusters, affecting downstream entropy profiles

## Evidence

- [discussion] Monte Carlo internal validation via re-propagation: "In the tth Monte Carlo trial, a random seed vertex r is selected, and the seed vertex label Sr,t at trial t takes on the same label Sr"
- [abstract] Entropy highlights transitions along trajectories: "enable the computation of “label entropies", highlighting transitions along developmental trajectories"
- [methods] API for validation outputs: "cluster_obj.validate()"
- [readme] Output attributes: "cluster_obj.entropy_list # list of entropies of the data point"
- [discussion] Trial-count variability caveat: "The number of Monte Carlo trials varies with the number of clusters in the data, as a larger number of clusters need more Monte Carlo trials"
