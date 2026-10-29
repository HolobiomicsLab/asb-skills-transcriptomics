---
name: knn-affinity-graph-construction
description: Use when analyzing preprocessed single-cell data (.h5ad, cells x features)
  and you need a cell-similarity or trajectory graph — e.g., for neighborhood-averaged
  feature matrices, dynamic module clustering, or Laplacian Score feature ranking
  as in DELVE.
when_to_use_negative:
- The input is already a cell-similarity graph or adjacency matrix rather than a cells
  x features expression matrix
- The dataset has not been preprocessed/normalized (e.g., not CPM normalized, log+1,
  HVG-filtered), since graph edges on raw noisy features can distort the trajectory
- The number of noisy/irrelevant features outweighs informative ones and no bottom-up
  module selection precedes graph use — the article notes similarity-based methods
  relying on the full-data graph can fail in that regime
edam_operation: http://edamontology.org/operation_2492
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: DELVE
  role: Implements the k=10 weighted k-NN affinity graph with Gaussian kernel (bandwidth
    = distance to 3rd nearest neighbor) in both Step 1 module identification and Step
    2 Laplacian Score ranking
  repo: https://github.com/jranek/delve
- name: Python
  role: Language in which the open-source DELVE package and graph construction are
    implemented
  repo: https://github.com/jranek/delve
- name: anndata
  role: Reads the preprocessed .h5ad annotated data object that supplies the cells
    x features matrix for graph construction
- name: scikit-learn
  role: Provides nearest-neighbor computation and KMeans++ clustering on the graph-derived
    delta_mean
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:b34a0d568bcba7588226472161935cca21ba788add590ae6eb44d88e2339568e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jranek/delve
  tools:
  - DELVE
  - EnrichR
  - Python
  - ScVelo
  - Scanpy
  - anndata
  - gseapy
  - scikit-learn
  - sketchKH
  - statsmodels
  edam_operation: http://edamontology.org/operation_2492
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# knn-affinity-graph-construction

## Summary

Construct a weighted k-nearest-neighbor between-cell affinity graph with Gaussian edge weights, where the kernel bandwidth sigma_i is set to the distance to the 3rd nearest neighbor, as used in both steps of the DELVE feature selection pipeline (equation 6) before forming the unnormalized graph Laplacian L = D - W_tilde.

## When to use

Use when analyzing preprocessed single-cell data (.h5ad, cells x features) and you need a cell-similarity or trajectory graph — e.g., for neighborhood-averaged feature matrices, dynamic module clustering, or Laplacian Score feature ranking as in DELVE.

## When NOT to use

- The input is already a cell-similarity graph or adjacency matrix rather than a cells x features expression matrix
- The dataset has not been preprocessed/normalized (e.g., not CPM normalized, log+1, HVG-filtered), since graph edges on raw noisy features can distort the trajectory
- The number of noisy/irrelevant features outweighs informative ones and no bottom-up module selection precedes graph use — the article notes similarity-based methods relying on the full-data graph can fail in that regime

## Inputs

- Preprocessed single-cell annotated data object (.h5ad), dimensions cells x features (e.g., 2759 cells x 241 imaging-derived features)
- Feature expression matrix (dense or sparse array), optionally PCA-reduced (n_pcs=50)
- Graph parameters: number of neighbors k=10; Gaussian bandwidth defined as distance to the 3rd nearest neighbor

## Outputs

- Weighted k-NN affinity graph W_tilde (cells x cells) with Gaussian edge weights
- k-NN neighbor-averaged feature matrix Z used for delta_mean computation
- Unnormalized graph Laplacian L = D - W_tilde (with diagonal degree matrix D) for Laplacian Score ranking

## How to apply

Start from a preprocessed annotated data object (e.g., the RPE 4i dataset with 2759 cells x 241 imaging-derived features, or PCA-reduced scRNA-seq with n_pcs=50). Compute pairwise cell distances and connect each cell to its k=10 nearest neighbors, weighting each edge by a Gaussian kernel whose bandwidth sigma_i equals the distance to the cell's 3rd nearest neighbor (equation 6). This same construction is reused twice in DELVE: once on all features to compute the k-NN neighbor-averaged matrix Z for delta_mean and Kernel Herding sketching (num_subsamples=1000) in Step 1, and again on the 13 dynamic seed features in Step 2. From the resulting weight matrix W_tilde, form the diagonal degree matrix D and the unnormalized graph Laplacian L = D - W_tilde, which is then used to compute Laplacian Scores (equation 9, f_tilde^T L f_tilde / f_tilde^T D f_tilde) that rank features by expression smoothness across neighboring cells.

## Related tools

- **DELVE** (Implements the k=10 weighted k-NN affinity graph with Gaussian kernel (bandwidth = distance to 3rd nearest neighbor) in both Step 1 module identification and Step 2 Laplacian Score ranking) — https://github.com/jranek/delve
- **Python** (Language in which the open-source DELVE package and graph construction are implemented) — https://github.com/jranek/delve
- **anndata** (Reads the preprocessed .h5ad annotated data object that supplies the cells x features matrix for graph construction)
- **scikit-learn** (Provides nearest-neighbor computation and KMeans++ clustering on the graph-derived delta_mean)

## Examples

```
from delve_fs import *; modules, seed_features = run_delve_fs(adata, k=10, num_ccs=5, num_subsamples=1000, n_random_states=10)
```

## Evaluation signals

- The weight matrix W_tilde is symmetric with exactly k=10 nonzero neighbors per cell (row sums of the adjacency structure equal 10)
- Each Gaussian edge weight uses bandwidth sigma_i equal to the distance to the cell's 3rd nearest neighbor, so no external bandwidth tuning is needed
- The degree matrix D has diagonal entries equal to row sums of W_tilde, and L = D - W_tilde has zero row sums
- When reused in Step 2 on the RPE 4i dataset, Laplacian Scores computed on this graph recover phase-specific features at precision@30 of about 0.800
- Cell-type frequencies and spectral properties of the full dataset are preserved when the graph supports Kernel Herding sketching with num_subsamples=1000

## Limitations

- Similarity-based methods that rely on the cell-similarity graph from the full dataset can fail to identify relevant features when noisy features outweigh informative ones
- Noisy or confounding features (dropout, low signal-to-noise, cell cycle, irrelevant imaging measurements) can distort or mask the intended trajectory encoded in the graph
- DELVE leverages all dynamic modules equally when constructing the approximate cellular trajectory graph; excluding unwanted dynamic modules is possible but not implemented

## Evidence

- [abstract] DELVE abstract method claim: "we present DELVE, an unsupervised feature selection method for identifying a representative subset of molecular features which robustly recapitulate cellular trajectories"
- [intro] Graph Laplacian in method context: "uniform manifold approximation and projection (UMAP)59, eigenvectors of the graph Laplacian matrix60"
- [abstract] Bottom-up module rationale: "DELVE uses a bottom-up approach to mitigate the effects of confounding sources of variation, and instead models cell states from dynamic gene or protein modules based on core regulatory complexes."
- [intro] Similarity-graph failure mode: "they rely on the cell-similarity graph from the full dataset and can fail to identify relevant features when the number of noisy features outweighs the number of informative ones"
- [readme] DELVE package description in README: "DELVE is an unsupervised feature selection method for identifying a representative subset of dynamically-expressed molecular features that recapitulate cellular trajectories from single-cell data"
