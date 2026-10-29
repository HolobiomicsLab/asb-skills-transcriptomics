---
name: single-cell-4i-data-preprocessing
description: Use when a preprocessed 4i/protein imaging single-cell dataset (.h5ad,
  e.g. RPE 2759 cells x 241 imaging-derived features) must be loaded and subsampled
  into representative neighborhoods before feature-module discovery.
when_to_use_negative:
- The input is raw image tiles or unsegmented fluorescence stacks rather than a preprocessed
  cell x feature .h5ad matrix
- The goal is supervised feature selection using pre-existing cell-type annotations
  rather than unsupervised trajectory-preserving selection
- Only a cell-similarity graph over the full noisy dataset is needed without neighborhood
  sketching; similarity-based methods can fail when noisy features outweigh informative
  ones
tools:
- name: DELVE
  role: Unsupervised feature selection applied to the preprocessed 4i matrix to identify
    dynamically-expressed feature modules and the 13-feature seed
  repo: https://github.com/jranek/delve
- name: sketchKH
  role: Kernel Herding sketching of the k-NN-averaged 4i feature matrix into m=1000
    prototypical cellular neighborhoods
  repo: https://github.com/CompCy-lab/SketchKH
- name: anndata
  role: Reads and holds the preprocessed .h5ad 4i data object (cells x features) used
    as skill input
- name: scikit-learn
  role: KMeans++ clustering of the transposed delta_mean into 5 feature modules across
    10 random initializations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:b50f1c0ceb2e6861db00b6dbce53fa7576d0fbb2c9bfad4c1c7d03cd63b54d61
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jranek/delve
  tools:
  - DELVE
  - Python
  - anndata
  - scikit-learn
  - sketchKH
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# single-cell-4i-data-preprocessing

## Summary

Load, validate, and distribution-aware subsample preprocessed iterative immunofluorescence (4i) single-cell datasets (.h5ad, cells x imaging-derived features) so that downstream DELVE module discovery operates on representative cellular neighborhoods. This skill grounds feature selection in the RPE 4i cell cycle benchmark of the DELVE paper.

## When to use

Use when a preprocessed 4i/protein imaging single-cell dataset (.h5ad, e.g. RPE 2759 cells x 241 imaging-derived features) must be loaded and subsampled into representative neighborhoods before feature-module discovery.

## When NOT to use

- The input is raw image tiles or unsegmented fluorescence stacks rather than a preprocessed cell x feature .h5ad matrix
- The goal is supervised feature selection using pre-existing cell-type annotations rather than unsupervised trajectory-preserving selection
- Only a cell-similarity graph over the full noisy dataset is needed without neighborhood sketching; similarity-based methods can fail when noisy features outweigh informative ones

## Inputs

- Preprocessed single-cell 4i annotated data object (.h5ad, e.g. adata_RPE.h5ad with 2759 cells x 241 imaging-derived features)
- adata.obs key defining sample-sets (e.g. FCS_File) for per-sample Kernel Herding sketching
- Sketching hyperparameters: num_subsamples (e.g. m=1000), gamma, frequency_seed, n_jobs

## Outputs

- kh_indices: indices of the subsampled representative cells per sample-set
- adata_subsample: downsampled annotated data object (num_subsamples x features)
- k-NN neighbor-averaged feature matrix Z (cells x features)
- modules.csv: feature-module assignments and variance-based permutation p-values; 13 dynamic seed features expected for the RPE 4i dataset

## How to apply

Read the preprocessed annotated data object with anndata (e.g. adata_RPE.h5ad, 2759 cells x 241 imaging-derived features) from the Zenodo release (zenodo.org/records/10534873). Construct a weighted k-nearest-neighbor between-cell affinity graph with k=10 using a Gaussian kernel whose bandwidth is the distance to the 3rd nearest neighbor, then compute the k-NN neighbor-averaged feature matrix Z. Subsample m=1000 prototypical cellular neighborhoods from Z with Kernel Herding (sketchKH's sketch function), which preserves cell-type frequencies and spectral properties of the full dataset. Run DELVE Step 1 on the subsample: compute delta_mean (average pairwise change in expression across the 1000 neighborhoods), cluster the transposed delta_mean with KMeans++ into n_clusters=5 modules over n_random_state=10 initializations, and flag dynamic modules by a variance-based permutation test against randomly drawn feature sets of equal size; the dynamic seed is the intersection of dynamically-expressed features across the 10 runs.

## Related tools

- **DELVE** (Unsupervised feature selection applied to the preprocessed 4i matrix to identify dynamically-expressed feature modules and the 13-feature seed) — https://github.com/jranek/delve
- **sketchKH** (Kernel Herding sketching of the k-NN-averaged 4i feature matrix into m=1000 prototypical cellular neighborhoods) — https://github.com/CompCy-lab/SketchKH
- **anndata** (Reads and holds the preprocessed .h5ad 4i data object (cells x features) used as skill input)
- **scikit-learn** (KMeans++ clustering of the transposed delta_mean into 5 feature modules across 10 random initializations)

## Examples

```
import anndata; from sketchKH import *; adata = anndata.read_h5ad('data/adata_RPE.h5ad'); kh_indices, adata_subsample = sketch(adata, sample_set_key='FCS_File', gamma=1, num_subsamples=1000, frequency_seed=0, n_jobs=-1)
```

## Evaluation signals

- The subsampled adata_subsample has exactly num_subsamples rows (1000 for the RPE benchmark) and preserves cell-type frequencies of the original sample
- delta_mean has dimensions num_subsamples x features (1000 x 241 for the RPE 4i dataset)
- The variance-based permutation test labels modules dynamic only when their observed average feature variance exceeds randomly drawn feature sets of equal size
- The recovered dynamic seed contains 13 features out of 241 total, matching the paper's reported RPE result
- modules.csv is produced with feature-module assignments and permutation p-values identical in schema across the 10 random initializations

## Limitations

- Feature selection can greatly bias interpretation of the underlying cellular trajectory; careful consideration is needed before applying it
- The sketch function assumes multiple profiled sample-sets keyed in adata.obs (e.g. FCS_File); single-sample layouts must be adapted
- Delta and clustering results are stochastic; 10 random KMeans++ initializations are used to reduce but not eliminate clustering variability

## Evidence

- [methods] 1. Load the preprocessed RPE 4i annotated data object (2759 cells x 241 imaging-derived features): [evidence span withheld — anchor status: not_found]
- [readme] README sketch invocation: [evidence span withheld — anchor status: external_doc]
- [readme] README data access: "You can download all of the preprocessed single-cell datasets (`.h5ad` files) from the [Zenodo](https://zenodo.org/records/10534873) repository."
- [abstract] Abstract scope: "Using simulations, single-cell RNA sequencing, and iterative immunofluorescence imaging data in the context of cell cycle and cellular differentiation"
- [discussion] Discussion rationale: "performing feature selection with DELVE prior to performing trajectory inference improved the identification and resolution of gene programs"
