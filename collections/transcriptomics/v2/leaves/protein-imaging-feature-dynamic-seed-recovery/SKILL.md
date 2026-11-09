---
name: protein-imaging-feature-dynamic-seed-recovery
description: Use when Step 1 of DELVE must be reproduced on a preprocessed .h5ad single-cell
  object (e.g., 2759 cells x 241 imaging-derived features) to identify temporally
  co-expressed dynamic feature modules and the dynamic seed.
when_to_use_negative:
- The goal is benchmarking DELVE against twelve alternative feature selection methods
  — use the delve_benchmark package (fs class with run_delve_fs, laplacian_score_fs,
  etc.) instead of reconstructing Step 1 manually
- Supervised feature selection is acceptable because manual annotations (e.g., cell
  types) are available; DELVE targets unsupervised settings where such labels are
  unavailable
- The input is raw or unnormalized data — this skill presumes a preprocessed .h5ad
  object such as those distributed on the Zenodo repository
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: DELVE
  role: Unsupervised feature selection method whose Step 1 identifies dynamically
    co-expressed feature modules and the dynamic seed from single-cell data
  repo: https://github.com/jranek/delve
- name: anndata
  role: Reads the preprocessed .h5ad annotated data object holding cells x imaging-derived
    features
- name: sketchKH
  role: Kernel Herding sketching that subsamples representative cellular neighborhoods
    while preserving cell-type frequencies and spectral properties
  repo: https://github.com/CompCy-lab/SketchKH
- name: scikit-learn
  role: Provides KMeans++ used to cluster the transposed delta_mean into n_clusters=5
    feature modules across 10 random initializations
- name: delve_benchmark
  role: Alternative entry point exposing seed_features (dynamic seed features from
    Step 1 of the DELVE algorithm) for feature selection and evaluation
  repo: https://github.com/jranek/delve_benchmark
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:d62f0010db262a7c81d613d9e0620035e2e926b3832d11f77c69b956845a9a0a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jranek/delve
  tools:
  - DELVE
  - Python
  - anndata
  - delve_benchmark
  - scikit-learn
  - sketchKH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8947368421052632
n_claims: 19
---

# protein-imaging-feature-dynamic-seed-recovery

## Summary

Reconstructs Step 1 of DELVE (dynamic seed selection) on imaging-derived single-cell features: it builds a k-NN neighborhood graph, sketches prototypical neighborhoods with Kernel Herding, clusters features by their average pairwise change in expression, and applies a variance-based permutation test to recover the dynamically-expressed seed module.

## When to use

Use when Step 1 of DELVE must be reproduced on a preprocessed .h5ad single-cell object (e.g., 2759 cells x 241 imaging-derived features) to identify temporally co-expressed dynamic feature modules and the dynamic seed.

## When NOT to use

- The goal is benchmarking DELVE against twelve alternative feature selection methods — use the delve_benchmark package (fs class with run_delve_fs, laplacian_score_fs, etc.) instead of reconstructing Step 1 manually
- Supervised feature selection is acceptable because manual annotations (e.g., cell types) are available; DELVE targets unsupervised settings where such labels are unavailable
- The input is raw or unnormalized data — this skill presumes a preprocessed .h5ad object such as those distributed on the Zenodo repository

## Inputs

- Preprocessed annotated data object (.h5ad) containing single-cell profiles, dimensions = cells x features (e.g., RPE 4i dataset: 2759 cells x 241 imaging-derived features)
- k-NN graph parameters: k=10 neighbors, Gaussian kernel bandwidth = distance to 3rd nearest neighbor
- Sketch size m=1000 (num_subsamples) of representative cellular neighborhoods
- KMeans++ settings: n_clusters=5 feature modules, n_random_state=10 clustering initializations

## Outputs

- modules.csv with feature-module assignments and permutation test p-values
- Dynamic seed module: the intersection of dynamically-expressed features across the 10 clustering initializations (expected 13 of 241 features for the RPE dataset)
- delta_mean matrix (num_subsamples x features) of average pairwise change in expression across prototypical neighborhoods

## How to apply

Load the preprocessed annotated data object (adata_RPE.h5ad, 2759 cells x 241 imaging-derived features) with anndata, then construct a weighted k-nearest-neighbor between-cell affinity graph with k=10 using a Gaussian kernel whose bandwidth is the distance to the 3rd nearest neighbor and compute the k-NN neighbor-averaged feature matrix Z. Subsample m=1000 representative cellular neighborhoods from Z using Kernel Herding sketching (sketchKH), which preserves cell-type frequencies and spectral properties of the dataset. Compute delta_mean, the average pairwise change in expression across the 1000 prototypical neighborhoods (num_subsamples x features), cluster the transposed delta_mean with KMeans++ into n_clusters=5 feature modules across n_random_state=10 initializations, and for each module run a variance-based permutation test comparing the module's average sample variance against randomly drawn feature sets of equal size. The dynamic seed is defined as the intersection of dynamically-expressed features across the 10 initializations; save feature-module assignments and permutation p-values as modules.csv.

## Related tools

- **DELVE** (Unsupervised feature selection method whose Step 1 identifies dynamically co-expressed feature modules and the dynamic seed from single-cell data) — https://github.com/jranek/delve
- **anndata** (Reads the preprocessed .h5ad annotated data object holding cells x imaging-derived features)
- **sketchKH** (Kernel Herding sketching that subsamples representative cellular neighborhoods while preserving cell-type frequencies and spectral properties) — https://github.com/CompCy-lab/SketchKH
- **scikit-learn** (Provides KMeans++ used to cluster the transposed delta_mean into n_clusters=5 feature modules across 10 random initializations)
- **delve_benchmark** (Alternative entry point exposing seed_features (dynamic seed features from Step 1 of the DELVE algorithm) for feature selection and evaluation) — https://github.com/jranek/delve_benchmark

## Examples

```
pip install delve-fs
```

## Evaluation signals

- The recovered dynamic seed contains 13 seed features out of 241 total imaging-derived features, matching the paper's reported RPE result
- modules.csv exists and contains feature-module assignments plus permutation p-values for the 5 modules
- delta_mean has dimensions num_subsamples x features (1000 x 241) computed from the Kernel Herding sketch of Z
- The dynamic module's average sample variance exceeds that of randomly drawn feature sets of equal size in the permutation test
- The Kernel Herding sketch preserves cell-type frequencies and spectral properties of the original dataset

## Limitations

- Feature selection can greatly bias interpretation of the underlying cellular trajectory; careful consideration should be made when performing feature selection for trajectory analysis
- DELVE can be improved by using a set of previously established regulators
- All dynamic modules were leveraged equally for constructing the approximate cellular trajectory; excluding unwanted dynamic modules is possible but not implemented

## Evidence

- [abstract] DELVE uses a bottom-up approach to mitigate the effects of confounding sources of variation, and instead models cell states from dynamic gene or protein modules based on core regulatory complexes: "DELVE uses a bottom-up approach to mitigate the effects of confounding sources of variation, and instead models cell states from dynamic gene or protein modules based on core regulatory complexes"
- [results] DELVE identifies dynamic modules of features that are temporally co-expressed by clustering features according to their average pairwise change in expression across prototypical cellular neighborhoods.: [evidence span withheld — anchor status: not_found]
- [readme] DELVE is an unsupervised feature selection method for identifying a representative subset of dynamically-expressed molecular features that recapitulate cellular trajectories from single-cell data: "DELVE is an unsupervised feature selection method for identifying a representative subset of dynamically-expressed molecular features that recapitulate cellular trajectories from single-cell data"
- [readme] seed_features - Dynamic seed features from Step 1 of the DELVE algorithm: [evidence span withheld — anchor status: external_doc]
