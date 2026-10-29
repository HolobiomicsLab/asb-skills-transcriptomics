---
name: graph-signal-smoothness-analysis
description: Use when you have single-cell data (.h5ad, e.g. scRNA-seq or protein
  iterative immunofluorescence) plus a small set of dynamic seed features from Step
  1 of DELVE, and you need to rank all measured features by their smoothness over
  the cell trajectory graph.
when_to_use_negative:
- You lack dynamic seed features from Step 1 of DELVE and would have to build the
  trajectory graph from the full noisy feature set, the failure mode DELVE's bottom-up
  approach is designed to avoid
- Your data have no meaningful cellular trajectory or gradual feature progression
  (e.g. static, well-separated cell types with no cell-type transitions)
- You have expensive/laborious manual cell-type annotations available and a supervised
  feature selection approach is appropriate
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: DELVE
  role: 'Implements the Step-2 pipeline: kNN affinity graph on dynamic seed modules
    (k=10, Gaussian weights, sigma_i = 3rd-neighbor distance), graph Laplacian L =
    D - W_tilde, and Laplacian Score ranking of all features'
  repo: https://github.com/jranek/delve
- name: Python
  role: Open-source language of the delve-fs package; the workflow uses anndata to
    load the Zenodo .h5ad deposit and numpy/scipy for the Laplacian Score computation
  repo: https://github.com/jranek/delve
- name: delve_benchmark
  role: Provides run_delve_fs and laplacian_score_fs functions plus the fs class (select()/evaluate_select())
    to run and evaluate this feature selection against 12 competing methods
  repo: https://github.com/jranek/delve_benchmark
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:3cfa379d166bf94febb927fd197bd7bc7637da0c926ee90373004a26af65c0b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jranek/delve
  tools:
  - DELVE
  - Python
  - delve_benchmark
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# graph-signal-smoothness-analysis

## Summary

Ranks features by Laplacian Score on an approximate cellular trajectory graph built from dynamic seed modules, so that globally smooth features are retained and rapidly oscillating, noisy features are excluded (Step 2 of DELVE, applied to the RPE 4i dataset of 2759 cells x 241 imaging-derived features).

## When to use

Use when you have single-cell data (.h5ad, e.g. scRNA-seq or protein iterative immunofluorescence) plus a small set of dynamic seed features from Step 1 of DELVE, and you need to rank all measured features by their smoothness over the cell trajectory graph.

## When NOT to use

- You lack dynamic seed features from Step 1 of DELVE and would have to build the trajectory graph from the full noisy feature set, the failure mode DELVE's bottom-up approach is designed to avoid
- Your data have no meaningful cellular trajectory or gradual feature progression (e.g. static, well-separated cell types with no cell-type transitions)
- You have expensive/laborious manual cell-type annotations available and a supervised feature selection approach is appropriate

## Inputs

- Preprocessed single-cell AnnData object (.h5ad), e.g. adata_RPE.h5ad (2759 cells x 241 imaging-derived features) from the Zenodo deposit
- List of dynamic seed features (13 features) from DELVE Step 1 dynamic module output

## Outputs

- ranked_features.csv: table of all features (e.g. 241) sorted in ascending order of Laplacian Score, including scores for every feature
- Set of top-ranked globally smooth features for downstream trajectory inference

## How to apply

Load the preprocessed .h5ad object (e.g. adata_RPE.h5ad, 2759 cells x 241 imaging-derived features) with anndata and subset the expression matrix to the 13 dynamic seed features produced by the Step-1 dynamic module output. Build a weighted k-nearest-neighbor affinity graph on the seed features with k = 10 and Gaussian edge weights, setting the kernel bandwidth sigma_i to the distance to the 3rd nearest neighbor (equation 6), then form the unnormalized graph Laplacian L = D - W_tilde with diagonal degree matrix D. For each of the 241 measured features, mean-center by D and compute the Laplacian Score Lf = f_tilde^T L f_tilde / f_tilde^T D f_tilde (equation 9). Sort features in ascending order of Laplacian Score: low scores correspond to features whose expression is similar among neighboring cells on the approximate trajectory graph, while high scores flag noisy, rapidly oscillating features to exclude. Write the full ranked feature table, including scores for all features, to ranked_features.csv.

## Related tools

- **DELVE** (Implements the Step-2 pipeline: kNN affinity graph on dynamic seed modules (k=10, Gaussian weights, sigma_i = 3rd-neighbor distance), graph Laplacian L = D - W_tilde, and Laplacian Score ranking of all features) — https://github.com/jranek/delve
- **Python** (Open-source language of the delve-fs package; the workflow uses anndata to load the Zenodo .h5ad deposit and numpy/scipy for the Laplacian Score computation) — https://github.com/jranek/delve
- **delve_benchmark** (Provides run_delve_fs and laplacian_score_fs functions plus the fs class (select()/evaluate_select()) to run and evaluate this feature selection against 12 competing methods) — https://github.com/jranek/delve_benchmark

## Examples

```
import anndata; adata = anndata.read_h5ad('adata_RPE.h5ad'); seed_features = [...13 Step-1 dynamic seed features...]; # pip install delve-fs, then rank all 241 features by Laplacian Score on the k=10 seed-feature graph and save ranked_features.csv
```

## Evaluation signals

- Top-30 ranked features recover phase-specific features at precision@30 of about 0.800, the median value reported for DELVE on the RPE dataset
- ranked_features.csv contains Laplacian Scores for all 241 features sorted in ascending order
- Features with low Laplacian Score show expression that is similar amongst neighboring cells on the seed-feature trajectory graph; top features should not be rapidly oscillating or noisy
- Affinity graph parameters reproduce the specification: k = 10 neighbors, Gaussian edge weights with bandwidth sigma_i equal to the distance to the 3rd nearest neighbor
- In the paper's benchmark, similarity-based feature selection methods (DELVE, Laplacian score, Hotspot) achieved the highest recovery of differentially expressed genes and highest cell-type classification accuracy on simulated scRNA-seq data

## Limitations

- Feature selection can greatly bias interpretation of the underlying cellular trajectory; careful consideration should be made when performing feature selection for trajectory analysis
- All dynamic modules were leveraged equally for constructing the approximate cellular trajectory; extension to exclude unwanted dynamic modules is possible but not implemented
- Unsupervised similarity-based methods can fail to identify relevant features when the number of noisy features outweighs the number of informative ones

## Evidence

- [methods] In step 2, DELVE builds an affinity graph from the dynamic seed modules and ranks all features by Laplacian Score, retaining globally smooth features: [evidence span withheld — anchor status: not_found]
- [methods] the top-30 ranked features should recover phase-specific features at precision@30 of about 0.800: [evidence span withheld — anchor status: not_found]
- [discussion] similarity-based feature selection methods (e.g., DELVE, Laplacian score, and Hotspot) achieved the highest recovery of differentially expressed genes: "similarity-based feature selection methods (e.g., DELVE, Laplacian score, and Hotspot) achieved the highest recovery of differentially expressed genes within a cell type or along a cellular lineage"
- [discussion] feature selection can greatly bias the interpretation of the underlying cellular trajectory: "feature selection can greatly bias the interpretation of the underlying cellular trajectory45, thus careful consideration should be made when performing feature selection for trajectory analysis"
- [readme] DELVE is an unsupervised feature selection method for identifying a representative subset of dynamically-expressed molecular features that recapitulate cellular trajectories: "DELVE is an unsupervised feature selection method for identifying a representative subset of dynamically-expressed molecular features that recapitulate cellular trajectories from single-cell data"
