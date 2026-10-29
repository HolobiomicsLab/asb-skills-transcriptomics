---
name: variance-based-permutation-testing-for-dynamic-expression
description: Use when applying DELVE Step 1 to preprocessed single-cell data (e.g.
  .h5ad of RPE 4i imaging features) to find the subset of dynamically-expressed features
  that recapitulate cellular trajectories before trajectory inference.
when_to_use_negative:
- When the goal is to rank features by variance alone (e.g. HVG or variance_score
  baselines) rather than identify dynamic, temporally co-expressed modules tied to
  a cell trajectory graph
- When features are already a curated seed/regulator list — the article notes DELVE
  could be improved by using a set of previously established regulators, which bypasses
  Step 1's unsupervised discovery
- When the data are not preprocessed single-cell profiles with meaningful neighborhoods
  (noisy/confounding features like dropout or irrelevant imaging measurements can
  distort the neighborhood graph the test depends on)
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: DELVE
  role: Implements Step 1 seed-module discovery and the variance-based permutation
    test (open-source python package delve-fs)
  repo: https://github.com/jranek/delve
- name: sketchKH
  role: Kernel Herding sketching to subsample m=1000 representative cellular neighborhoods
    from the k-NN-averaged feature matrix Z
  repo: https://github.com/CompCy-lab/SketchKH
- name: anndata
  role: Reads and stores the preprocessed .h5ad single-cell data object (cells x features)
- name: scikit-learn
  role: Provides KMeans++ used to cluster the transposed delta_mean into n_clusters=5
    feature modules across 10 random initializations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:1b6fc0465642645759041ea705039aabb85c7ace5e621520dab15912603f0740
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
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# variance-based-permutation-testing-for-dynamic-expression

## Summary

Step 1 of the DELVE algorithm: identify dynamically co-expressed feature modules by clustering features according to their average pairwise change in expression across prototypical cellular neighborhoods, then label modules as dynamic via a variance-based permutation test to recover the dynamic seed.

## When to use

Use when applying DELVE Step 1 to preprocessed single-cell data (e.g. .h5ad of RPE 4i imaging features) to find the subset of dynamically-expressed features that recapitulate cellular trajectories before trajectory inference.

## When NOT to use

- When the goal is to rank features by variance alone (e.g. HVG or variance_score baselines) rather than identify dynamic, temporally co-expressed modules tied to a cell trajectory graph
- When features are already a curated seed/regulator list — the article notes DELVE could be improved by using a set of previously established regulators, which bypasses Step 1's unsupervised discovery
- When the data are not preprocessed single-cell profiles with meaningful neighborhoods (noisy/confounding features like dropout or irrelevant imaging measurements can distort the neighborhood graph the test depends on)

## Inputs

- Preprocessed annotated single-cell data object (.h5ad), dimensions = cells x features (e.g. adata_RPE.h5ad: 2759 cells x 241 imaging-derived features)
- DELVE Step 1 hyperparameters: k=10 neighbors, Gaussian bandwidth = distance to 3rd nearest neighbor, num_subsamples m=1000, n_clusters=5, n_random_state=10

## Outputs

- modules.csv containing feature-module assignments and permutation p-values
- Dynamic seed feature set (intersection of dynamically-expressed features across the 10 KMeans++ initializations; 13 of 241 features for the RPE dataset)
- delta_mean matrix (num_subsamples x features) of average pairwise change in expression across prototypical neighborhoods

## How to apply

Load the preprocessed annotated data object (e.g. adata_RPE.h5ad, 2759 cells x 241 imaging-derived features) with anndata and build a weighted k-nearest neighbor between-cell affinity graph with k=10 using a Gaussian kernel whose bandwidth is the distance to the 3rd nearest neighbor, then compute the k-NN neighbor-averaged feature matrix Z. Subsample m=1000 representative cellular neighborhoods from Z with Kernel Herding sketching (sketchKH), which preserves cell-type frequencies and spectral properties of the original dataset. Compute delta_mean, the num_subsamples x features matrix of average pairwise change in expression across the 1000 prototypical neighborhoods, and cluster its transpose with KMeans++ into n_clusters=5 feature modules, repeating across n_random_state=10 clustering initializations to reduce stochasticity. For each module, run a variance-based permutation test comparing the average sample variance of the module's features against randomly drawn feature sets of equal size across random permutations; a module is labeled dynamic if its observed variance exceeds the random assignment. Define the dynamic seed as the intersection of dynamically-expressed features across the 10 initializations, and save feature-module assignments and permutation p-values as modules.csv.

## Related tools

- **DELVE** (Implements Step 1 seed-module discovery and the variance-based permutation test (open-source python package delve-fs)) — https://github.com/jranek/delve
- **sketchKH** (Kernel Herding sketching to subsample m=1000 representative cellular neighborhoods from the k-NN-averaged feature matrix Z) — https://github.com/CompCy-lab/SketchKH
- **anndata** (Reads and stores the preprocessed .h5ad single-cell data object (cells x features))
- **scikit-learn** (Provides KMeans++ used to cluster the transposed delta_mean into n_clusters=5 feature modules across 10 random initializations)

## Examples

```
from delve_benchmark.tl import seed_features; seed_feats = seed_features(adata, k=10, num_subsamples=1000, n_clusters=5, n_random_state=10)
```

## Evaluation signals

- Recovered dynamic seed contains 13 seed features out of 241 total imaging-derived features, matching the paper's reported result for the RPE 4i dataset
- modules.csv exists with columns for feature-module assignments and permutation p-values for all 241 features
- Dynamic modules show average sample variance exceeding that of randomly drawn feature sets of equal size in the permutation test
- Consistency of the seed intersection across the 10 KMeans++ random initializations (low stochasticity in recovered seed features)
- Intermediate delta_mean has the expected dimensions (num_subsamples=1000 x num_features=241) and sketching preserves cell-type frequencies and spectral properties of the original dataset

## Limitations

- The dynamic seed definition depends on n_random_state=10 clustering initializations; KMeans++ stochasticity can affect which features fall in dynamic modules
- Feature selection can greatly bias interpretation of the underlying cellular trajectory, so careful consideration is needed when performing feature selection for trajectory analysis
- DELVE could be improved by incorporating a set of previously established regulators rather than relying solely on unsupervised dynamic module discovery

## Evidence

- [methods] DELVE Step 1 variance-based permutation test: [evidence span withheld — anchor status: not_found]
- [readme] DELVE method overview: "DELVE is an unsupervised feature selection method for identifying a representative subset of dynamically-expressed molecular features"
- [abstract] Rationale for dynamic module modeling: "DELVE uses a bottom-up approach to mitigate the effects of confounding sources of variation, and instead models cell states from dynamic gene or protein modules based on core regulatory complexes"
- [discussion] Downstream trajectory construction caveat: "we leveraged all dynamic modules equally for constructing the approximate cellular trajectory; however, we note that is possible to extend this framework to exclude unwanted dynamic modules"
