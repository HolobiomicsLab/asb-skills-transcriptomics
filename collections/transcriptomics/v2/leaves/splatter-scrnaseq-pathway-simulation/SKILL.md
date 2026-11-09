---
name: splatter-scrnaseq-pathway-simulation
description: Use when you need simulated single-cell RNA-seq data with continuous
  differentiation structure and known pseudotime to benchmark clustering, pseudotime,
  or label-uncertainty methods against a ground truth.
when_to_use_negative:
- Data contain only a single cluster — Monte Carlo internal validation cannot accurately
  estimate uncertainty when every data point can only receive one label
- The goal is clustering quality benchmarking (purity/ARI/silhouette) rather than
  trajectory/transition analysis — use the paper's benchmark workflow instead
- You need expression values along pre-defined biological pathways rather than a continuous
  differentiation path — Splatter path simulations model pseudotime progression, not
  pathway activity
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Splatter
  role: R package used to generate simulated scRNA-seq differentiation-path data with
    known pseudotime
- name: scanpy
  role: Preprocessing (zheng17 recipe), neighbor graph, and diffusion pseudotime rooted
    at Leiden clusters for experimental data
- name: scipy
  role: Spearman correlation between label entropy and pseudotime
- name: forest-fire-clustering
  role: FFC().preprocess(X), FFC().fit(fire_temp=100), validate() and entropy_list
    supply the point-wise label entropies being validated
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: numba
  role: Numerical acceleration of the clustering and Monte Carlo validation used downstream
    of the simulations
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:1a97b399dc7c327c407e380e6cf83ded6670e1930c91aa76ece260b5de52607d
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# splatter-scrnaseq-pathway-simulation

## Summary

Generate simulated scRNA-seq differentiation-path datasets with the R package Splatter so each cell carries a known pseudotime coordinate, enabling validation of trajectory-aware methods (e.g., Forest Fire Clustering label entropy) against ground-truth developmental progression.

## When to use

Use when you need simulated single-cell RNA-seq data with continuous differentiation structure and known pseudotime to benchmark clustering, pseudotime, or label-uncertainty methods against a ground truth.

## When NOT to use

- Data contain only a single cluster — Monte Carlo internal validation cannot accurately estimate uncertainty when every data point can only receive one label
- The goal is clustering quality benchmarking (purity/ARI/silhouette) rather than trajectory/transition analysis — use the paper's benchmark workflow instead
- You need expression values along pre-defined biological pathways rather than a continuous differentiation path — Splatter path simulations model pseudotime progression, not pathway activity

## Inputs

- Splatter simulation parameters for a continuous differentiation path (R)
- Raw scRNA-seq gene expression count matrix for experimental counterpart data (e.g., PBMC counts)
- Known per-cell pseudotime coordinates for simulated cells

## Outputs

- Simulated scRNA-seq count matrix with ground-truth pseudotime per cell
- Per-cell Forest Fire label entropies (cluster_obj.entropy_list)
- Correlation table (dataset, cell type/path, Spearman correlation) e.g. entropy_pseudotime_correlation.csv

## How to apply

Use the R package Splatter (as in the article: 'Simulated scRNA-seq data were generated using the R package Splatter') to generate continuous differentiation-path scRNA-seq simulations in which each cell carries a known pseudotime coordinate. Preprocess experimental comparisons with scanpy's 'zheng17' recipe (e.g., PBMC RNA gene expression count matrices) and compute diffusion pseudotime rooted at Leiden clusters as a biological counterpart to the simulated paths. For the skill's downstream use in this paper, run Forest Fire Clustering on both datasets (FFC().preprocess(X) with the first 30 principal components, then FFC().fit(fire_temp=100)), perform the Monte Carlo internal validation via cluster_obj.validate(), and retrieve point-wise label uncertainties from cluster_obj.entropy_list. Compute Spearman correlations between entropy_list and pseudotime per path/cell type with scipy.stats, and assemble the coefficients into a table (e.g., entropy_pseudotime_correlation.csv). Rationale: the paper found entropy highlights transitions along developmental trajectories, so correct application should reproduce an inverse entropy-pseudotime relationship.

## Related tools

- **Splatter** (R package used to generate simulated scRNA-seq differentiation-path data with known pseudotime)
- **scanpy** (Preprocessing (zheng17 recipe), neighbor graph, and diffusion pseudotime rooted at Leiden clusters for experimental data)
- **scipy** (Spearman correlation between label entropy and pseudotime)
- **forest-fire-clustering** (FFC().preprocess(X), FFC().fit(fire_temp=100), validate() and entropy_list supply the point-wise label entropies being validated) — https://github.com/gersteinlab/forest-fire-clustering
- **numba** (Numerical acceleration of the clustering and Monte Carlo validation used downstream of the simulations)

## Examples

```
from forest_fire_clustering.forest_fire_clustering import FFC; cluster_obj = FFC(); cluster_obj.preprocess(X); cluster_obj.fit(fire_temp=100); cluster_obj.validate(); ent = cluster_obj.entropy_list
```

## Evaluation signals

- Every reported Spearman correlation between label entropy and pseudotime should be negative (paper's validation criterion, Figs. 4d-f and 5i)
- Progenitor cells identified by diffusion pseudotime should match the cells highlighted by Forest Fire label entropy
- Per-cell entropy values are retrievable as cluster_obj.entropy_list after running cluster_obj.validate()
- Correlation table has the expected schema: columns for dataset, cell type/path, and correlation value
- Simulated cells must each carry a defined pseudotime coordinate from the Splatter path before correlation

## Limitations

- Forest Fire Clustering is limited by the curse of dimensionality; distances become more uniform in high dimensions, so a nonlinear dimensionality reduction (UMAP, t-SNE, PHATE) should precede clustering
- The fire temperature parameter c strongly influences the size and number of discovered clusters (resolution-like sensitivity)
- The number of Monte Carlo trials varies with the number of clusters and is not fixed, affecting reproducibility of entropy estimates
- For datasets over 1 million points, the distance matrix must be approximated via KNN

## Evidence

- [discussion] Simulated scRNA-seq data were generated using the R package Splatter.: "Simulated scRNA-seq data were generated using the R package Splatter."
- [abstract] label entropies highlight transitions along trajectories: "enable the computation of “label entropies", highlighting transitions along developmental trajectories"
- [readme] FFC usage from README: "cluster_obj.preprocess(X)
cluster_obj.fit(fire_temp=100)"
- [readme] entropy retrieval from README: "cluster_obj.entropy_list # list of entropies of the data point"
- [discussion] scanpy preprocessing of PBMC data: "The RNA gene expression count data for both PBMC datasets were preprocessed using the “zheng17” recipe in the scanpy package"
- [discussion] diffusion pseudotime rooting: "Leiden clusters with Procr marker expression were used as roots for diffusion pseudotime analysis."
