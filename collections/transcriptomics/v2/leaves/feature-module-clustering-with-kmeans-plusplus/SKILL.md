---
name: feature-module-clustering-with-kmeans-plusplus
description: Use when you have a preprocessed cells x features single-cell matrix
  (.h5ad) and need to identify dynamically co-expressed feature modules that seed
  trajectory-preserving feature selection.
when_to_use_negative:
- The goal is ranking individual features by variance or differential expression rather
  than discovering dynamically co-expressed feature modules — the paper notes variance-based
  selection can be overwhelmed by noisy features
- The input data lack the neighborhood structure needed for delta_mean computation
  (e.g., too few cells or no meaningful cell-state continuum), since clustering operates
  on expression changes across neighborhoods, not raw profiles
- Supervised feature selection is appropriate (known labels/annotations exist), which
  the paper notes has stronger power but requires manual annotations often unavailable
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: DELVE
  role: Package implementing the dynamic seed selection pipeline (Step 1) in which
    feature-module clustering is embedded; installed as delve-fs via pip
  repo: https://github.com/jranek/delve
- name: scikit-learn
  role: Provides KMeans++ clustering of the transposed delta_mean feature matrix into
    n_clusters=5 modules
- name: sketchKH
  role: Kernel Herding sketching that produces the m=1000 prototypical cellular neighborhoods
    used to compute delta_mean prior to clustering
  repo: https://github.com/CompCy-lab/SketchKH
- name: anndata
  role: Loads the preprocessed .h5ad single-cell object (cells x features) that supplies
    the input expression matrix
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:dc69b2bb32e90e81fabcd30b70b3be32f1af592387b008afa323842a23d80408
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# feature-module-clustering-with-kmeans-plusplus

## Summary

Clusters features (genes or imaging-derived measurements) into co-expressed modules by applying KMeans++ to the transposed delta_mean matrix — the average pairwise change in feature expression across prototypical cellular neighborhoods obtained via Kernel Herding sketching — as Step 1 of the DELVE dynamic seed selection pipeline.

## When to use

Use when you have a preprocessed cells x features single-cell matrix (.h5ad) and need to identify dynamically co-expressed feature modules that seed trajectory-preserving feature selection.

## When NOT to use

- The goal is ranking individual features by variance or differential expression rather than discovering dynamically co-expressed feature modules — the paper notes variance-based selection can be overwhelmed by noisy features
- The input data lack the neighborhood structure needed for delta_mean computation (e.g., too few cells or no meaningful cell-state continuum), since clustering operates on expression changes across neighborhoods, not raw profiles
- Supervised feature selection is appropriate (known labels/annotations exist), which the paper notes has stronger power but requires manual annotations often unavailable

## Inputs

- Preprocessed annotated single-cell data object (adata_RPE.h5ad), dimensions = 2759 cells x 241 imaging-derived features
- delta_mean matrix (num_subsamples=1000 x features) of average pairwise change in expression across prototypical neighborhoods
- Hyperparameters: n_clusters=5, n_random_state=10

## Outputs

- Feature-module assignments for all features (modules.csv)
- Permutation test p-values labeling modules as dynamic or not
- Dynamic seed module: intersection of dynamically-expressed features across the 10 clustering initializations (13 features for the RPE dataset)

## How to apply

First compute delta_mean, the num_subsamples x features matrix of average pairwise change in expression across m=1000 prototypical cellular neighborhoods subsampled with Kernel Herding (sketchKH) from the k-NN neighbor-averaged feature matrix Z (k=10, Gaussian kernel with bandwidth equal to the distance to the 3rd nearest neighbor). Then cluster the transposed delta_mean with KMeans++ into n_clusters=5 feature modules, repeating across n_random_state=10 random initializations to reduce stochasticity. For each module, run a variance-based permutation test comparing the average sample variance of the module's features to randomly drawn feature sets of equal size; the dynamic seed is the intersection of dynamically-expressed features across the 10 initializations. On the RPE 4i dataset (2759 cells x 241 imaging-derived features) this recovers a 13-feature dynamic seed module, matching the paper's reported result.

## Related tools

- **DELVE** (Package implementing the dynamic seed selection pipeline (Step 1) in which feature-module clustering is embedded; installed as delve-fs via pip) — https://github.com/jranek/delve
- **scikit-learn** (Provides KMeans++ clustering of the transposed delta_mean feature matrix into n_clusters=5 modules)
- **sketchKH** (Kernel Herding sketching that produces the m=1000 prototypical cellular neighborhoods used to compute delta_mean prior to clustering) — https://github.com/CompCy-lab/SketchKH
- **anndata** (Loads the preprocessed .h5ad single-cell object (cells x features) that supplies the input expression matrix)

## Examples

```
from sklearn.cluster import KMeans; modules = KMeans(n_clusters=5, init='k-means++', n_init=10, random_state=s).fit_predict(delta_mean.T)
```

## Evaluation signals

- The recovered dynamic seed module contains 13 seed features (p=13 of 241 total imaging-derived features) for the RPE 4i cell cycle dataset, matching the paper's reported result
- modules.csv is produced containing feature-module assignments and permutation p-values for all 241 features
- Clustering is stable across n_random_state=10 KMeans++ initializations; the seed is defined as the intersection of dynamic features across all 10 runs
- Permutation test labels modules as dynamic only when the observed average sample variance exceeds variance from randomly drawn feature sets of equal size

## Limitations

- Feature selection can greatly bias interpretation of the underlying cellular trajectory; careful consideration is needed when performing feature selection for trajectory analysis
- DELVE could be improved by incorporating a set of previously established regulators
- Clustering results depend on stochastic KMeans++ initializations; the pipeline mitigates this by repeating across 10 random states and intersecting dynamic features

## Evidence

- [methods] Step 1 identifies dynamic modules: [evidence span withheld — anchor status: not_found]
- [intro] DELVE bottom-up dynamic modules: "DELVE uses a bottom-up approach to mitigate the effect of unwanted sources of variation confounding feature selection and trajectory inference, and instead models cell states from dynamic feature"
- [readme] README package description: "DELVE is an unsupervised feature selection method for identifying a representative subset of dynamically-expressed molecular features that recapitulate cellular trajectories from single-cell data"
- [discussion] Benchmark conclusion: "similarity-based feature selection methods (e.g., DELVE, Laplacian score, and Hotspot) achieved the highest recovery of differentially expressed genes within a cell type or along a cellular lineage"
