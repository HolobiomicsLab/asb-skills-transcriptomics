---
name: trajectory-graph-feature-evaluation
description: Use when you have dynamic seed features from DELVE Step 1 and need to
  rank all features by how well their expression varies smoothly along the underlying
  cellular trajectory graph rather than by variance.
when_to_use_negative:
- When you lack a set of trusted dynamic seed features — building the affinity graph
  on the full noisy feature set is exactly the failure mode DELVE Step 2 is designed
  to avoid
- When the goal is supervised identification of biologically relevant features using
  manual cell-type annotations; DELVE is unsupervised and does not exploit labels
- When you need to exclude unwanted dynamic modules from the trajectory graph — the
  paper notes all dynamic modules are currently leveraged equally and this extension
  is not implemented
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DELVE
  role: Python package implementing the kNN affinity graph construction and Laplacian
    Score feature ranking of Step 2 (pip install delve-fs)
  repo: https://github.com/jranek/delve
- name: Python
  role: Runtime environment (>= 3.9) with anndata, numpy, scipy, scanpy, umap-learn
    dependencies for executing the workflow
  repo: https://github.com/jranek/delve
- name: delve_benchmark
  role: Provides laplacian_score_fs and seed_features functions to reproduce Step
    2 within the benchmarking framework
  repo: https://github.com/jranek/delve_benchmark
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:2adeaed789676f81b3d67198de469a96f11e62e3bec9dd428a8368994a0ce680
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jranek/delve
  tools:
  - DELVE
  - Python
  - delve_benchmark
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# trajectory-graph-feature-evaluation

## Summary

Step 2 of DELVE: construct an approximate cellular trajectory graph from dynamic seed feature modules (weighted kNN affinity graph, k=10, Gaussian weights with sigma_i set to the distance to the 3rd nearest neighbor) and rank all measured features by Laplacian Score to retain globally smooth features.

## When to use

Use when you have dynamic seed features from DELVE Step 1 and need to rank all features by how well their expression varies smoothly along the underlying cellular trajectory graph rather than by variance.

## When NOT to use

- When you lack a set of trusted dynamic seed features — building the affinity graph on the full noisy feature set is exactly the failure mode DELVE Step 2 is designed to avoid
- When the goal is supervised identification of biologically relevant features using manual cell-type annotations; DELVE is unsupervised and does not exploit labels
- When you need to exclude unwanted dynamic modules from the trajectory graph — the paper notes all dynamic modules are currently leveraged equally and this extension is not implemented

## Inputs

- Preprocessed single-cell .h5ad object (e.g., adata_RPE.h5ad, 2759 cells x 241 features) from the Zenodo deposit
- List of dynamic seed features from DELVE Step 1 (e.g., 13 dynamic seed features for the RPE 4i dataset)

## Outputs

- ranked_features.csv: feature table with Laplacian Scores for all measured features, sorted ascending
- Ranked feature list (e.g., top-30 features) for downstream trajectory-preserving feature selection

## How to apply

Load the preprocessed .h5ad dataset (e.g., adata_RPE.h5ad, 2759 cells x 241 imaging-derived features) with anndata and subset the expression matrix to the 13 dynamic seed features identified in Step 1. Build a weighted k-nearest-neighbor affinity graph (k=10) on the seed features with Gaussian edge weights, setting each kernel bandwidth sigma_i to the distance to the cell's 3rd nearest neighbor (equation 6), and form the unnormalized graph Laplacian L = D - W_tilde with diagonal degree matrix D. For each of the 241 measured features, mean-center by D and compute the Laplacian Score Lf = f_tilde^T L f_tilde / f_tilde^T D f_tilde (equation 9). Sort features in ascending Laplacian Score: low scores indicate expression that is similar amongst neighboring cells on the trajectory graph, while high scores indicate rapidly oscillating, noisy features that should be excluded. This step is why DELVE succeeds where variance-based or full-dataset similarity-graph methods fail: the graph is built only from trusted dynamic seed modules, shielding the trajectory estimate from noisy features.

## Related tools

- **DELVE** (Python package implementing the kNN affinity graph construction and Laplacian Score feature ranking of Step 2 (pip install delve-fs)) — https://github.com/jranek/delve
- **Python** (Runtime environment (>= 3.9) with anndata, numpy, scipy, scanpy, umap-learn dependencies for executing the workflow) — https://github.com/jranek/delve
- **delve_benchmark** (Provides laplacian_score_fs and seed_features functions to reproduce Step 2 within the benchmarking framework) — https://github.com/jranek/delve_benchmark

## Examples

```
import anndata; adata = anndata.read_h5ad('adata_RPE.h5ad'); from delve_benchmark.tl import laplacian_score_fs; ranked = laplacian_score_fs(X=adata[:, seed_features].X, feature_names=seed_features)
```

## Evaluation signals

- Top-30 ranked features recover phase-specific features at precision@30 of about 0.800, the median value reported for DELVE on the RPE dataset
- ranked_features.csv contains Laplacian Scores for all 241 measured features, sorted in ascending order
- Retained (low-score) features show expression similar amongst neighboring cells on the seed-feature trajectory graph; excluded features are rapidly oscillating/noisy
- Affinity graph parameters match specification: k=10 neighbors, Gaussian weights, sigma_i = distance to the 3rd nearest neighbor

## Limitations

- Feature selection can greatly bias the interpretation of the underlying cellular trajectory; careful consideration is needed when performing feature selection for trajectory analysis
- All dynamic modules are leveraged equally when constructing the approximate cellular trajectory; excluding unwanted dynamic modules is possible in principle but not implemented
- Unsupervised similarity-based feature selection can fail to identify relevant features when the number of noisy features outweighs the number of informative ones — hence the seed-module graph construction

## Evidence

- [other] In step 2, DELVE builds an affinity graph from the dynamic seed modules and ranks all features by Laplacian Score: [evidence span withheld — anchor status: not_found]
- [abstract] unsupervised feature selection method for identifying a representative subset: "we present DELVE, an unsupervised feature selection method for identifying a representative subset of molecular features which robustly recapitulate cellular trajectories"
- [readme] DELVE is available as an open-source python package: "DELVE is available as an open-source python package: https://github.com/jranek/delve"
- [discussion] similarity-based feature selection methods achieved the highest recovery: "similarity-based feature selection methods (e.g., DELVE, Laplacian score, and Hotspot) achieved the highest recovery of differentially expressed genes within a cell type or along a cellular lineage"
- [discussion] performing feature selection with DELVE prior to trajectory inference: "performing feature selection with DELVE prior to performing trajectory inference improved the identification and resolution of gene programs"
