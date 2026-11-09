---
name: entropy-pseudotime-correlation-analysis
description: 'Use when single-cell clustering results must be checked against a developmental
  trajectory: e.g. testing whether label entropy is high at transition states and
  decreases with differentiation pseudotime.'
when_to_use_negative:
- Data contains only one cluster — Monte Carlo internal validation cannot accurately
  estimate uncertainty when every data point receives one label
- Clustering was performed on raw high-dimensional features without nonlinear dimensionality
  reduction (UMAP/t-SNE/PHATE), since the method is sensitive to the curse of dimensionality
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: forest-fire-clustering
  role: Provides FFC preprocess/fit/validate/entropy pipeline whose entropy_list is
    correlated against pseudotime
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: Splatter
  role: R package used to generate simulated scRNA-seq differentiation paths with
    known pseudotime
- name: scanpy
  role: Preprocesses PBMC counts with the 'zheng17' recipe and supports diffusion
    pseudotime rooted at Leiden clusters
- name: scipy
  role: Spearman correlation between entropy_list and pseudotime
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:e9ffbeb583c1c193daa48b72042d6cf66ea0020a624e861cf8b93c3e74a21203
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
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# entropy-pseudotime-correlation-analysis

## Summary

Validates Forest Fire Clustering's point-wise label entropies as a trajectory read-out by correlating cluster_obj.entropy_list with pseudotime in simulated (Splatter) and PBMC (scanpy diffusion pseudotime) scRNA-seq data.

## When to use

Use when single-cell clustering results must be checked against a developmental trajectory: e.g. testing whether label entropy is high at transition states and decreases with differentiation pseudotime.

## When NOT to use

- Data contains only one cluster — Monte Carlo internal validation cannot accurately estimate uncertainty when every data point receives one label
- Clustering was performed on raw high-dimensional features without nonlinear dimensionality reduction (UMAP/t-SNE/PHATE), since the method is sensitive to the curse of dimensionality

## Inputs

- Splatter-simulated scRNA-seq matrix (cells x genes) with per-cell ground-truth pseudotime coordinates
- PBMC RNA gene expression count matrix preprocessed with scanpy's 'zheng17' recipe
- Diffusion pseudotime values per cell, rooted at Leiden clusters with Procr marker expression

## Outputs

- cluster_obj.entropy_list — per-cell label entropies from FFC Monte Carlo internal validation
- Spearman correlation coefficients between entropy and pseudotime per dataset/cell type
- entropy_pseudotime_correlation.csv with columns: dataset, cell type/path, correlation value

## How to apply

Generate continuous differentiation-path simulations in R with Splatter so each cell carries a known pseudotime coordinate, and preprocess PBMC RNA count matrices in Python with scanpy's 'zheng17' recipe, computing diffusion pseudotime rooted at Leiden clusters (e.g. Procr-positive roots). Run Forest Fire Clustering via FFC().preprocess(X) then FFC().fit(fire_temp=100), execute the Monte Carlo internal validation with cluster_obj.validate(), and retrieve point-wise label uncertainties with cluster_obj.entropy(), collecting cluster_obj.entropy_list. For each Splatter path and each PBMC cell type, compute the Spearman correlation (scipy.stats) between entropy_list and the corresponding pseudotime values. Assemble coefficients into a table (dataset, cell type/path, correlation) saved as entropy_pseudotime_correlation.csv. The method is grounded in the paper's finding that label entropies highlight transitions along developmental trajectories.

## Related tools

- **forest-fire-clustering** (Provides FFC preprocess/fit/validate/entropy pipeline whose entropy_list is correlated against pseudotime) — https://github.com/gersteinlab/forest-fire-clustering
- **Splatter** (R package used to generate simulated scRNA-seq differentiation paths with known pseudotime)
- **scanpy** (Preprocesses PBMC counts with the 'zheng17' recipe and supports diffusion pseudotime rooted at Leiden clusters)
- **scipy** (Spearman correlation between entropy_list and pseudotime)

## Examples

```
from forest_fire_clustering.forest_fire_clustering import FFC; cluster_obj = FFC(); cluster_obj.preprocess(X); cluster_obj.fit(fire_temp=100); cluster_obj.validate(); cluster_obj.entropy(); ent = cluster_obj.entropy_list
```

## Evaluation signals

- Every reported Spearman correlation between label entropy and pseudotime is negative, mirroring the paper's finding (Figs. 4d-f and 5i)
- Progenitor cells identified by diffusion pseudotime overlap with cells highlighted by high Forest Fire label entropy
- entropy_list has exactly one entropy value per cell in the input matrix after cluster_obj.validate() and cluster_obj.entropy() complete
- Output CSV contains the required columns (dataset, cell type/path, correlation value) for every Splatter path and PBMC cell type analyzed

## Limitations

- The number of Monte Carlo trials varies with the number of clusters, and large trial counts are computationally intensive (though parallelizable)
- Forest Fire Clustering is sensitive to high dimensionality and should be performed after nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE)
- The fire temperature parameter c strongly influences the size and number of discovered clusters; fire_temp=100 is the paper's setting

## Evidence

- [abstract] posterior distributions allow evaluation of label confidence and entropy: "enable the computation of “label entropies", highlighting transitions along developmental trajectories"
- [methods] FFC API for validate and entropy: "cluster_obj.validate()"
- [methods] entropy retrieval method: "cluster_obj.entropy()"
- [discussion] diffusion pseudotime rooted at Leiden clusters: "Leiden clusters with Procr marker expression were used as roots for diffusion pseudotime analysis."
- [discussion] PBMC preprocessing recipe: "The RNA gene expression count data for both PBMC datasets were preprocessed using the “zheng17” recipe in the scanpy package"
- [discussion] Splatter simulation tool: "Simulated scRNA-seq data were generated using the R package Splatter."
- [readme] README usage of entropy_list: "cluster_obj.entropy_list # list of entropies of the data point"
