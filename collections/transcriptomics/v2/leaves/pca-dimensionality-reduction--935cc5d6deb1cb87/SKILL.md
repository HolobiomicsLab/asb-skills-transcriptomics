---
name: pca-dimensionality-reduction
description: 'Use when evaluating imputation or preprocessing effects on single-cell
  clustering: after obtaining a gene-by-cell matrix (raw or imputed), reduce it with
  50-component PCA prior to k-means clustering and ARI computation.'
when_to_use_negative:
- When the analysis goal is dimensionality reduction for visualization of already-normalized
  feature tables rather than a controlled ARI comparison between raw and imputed data.
- When the dataset is small enough that full PCA is unnecessary or when a different
  embedding (e.g. for trajectory inference) is the target — this skill is specific
  to the 50-component PCA feeding k-means in the paper's benchmark.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: irlba
  role: Provides prcomp_irlba for truncated PCA with n = 50 components on the transposed
    (cells x genes) matrix
- name: mclust
  role: Provides adjustedRandIndex to score k-means clusters from the PCA scores against
    known cell-type labels
- name: R stats (kmeans)
  role: Performs k-means clustering on the 50-component PCA scores with nstart = 2000,
    iter.max = 2000
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-06500-4
    title: A novel method for single-cell data imputation using subspace regression
  source_content_hash: sha256:98d8c9eb2cf7eb0288d60b417c8c438c2f86cc915bf6e36d63c52644d23d6324
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R stats
  - R stats (kmeans)
  - irlba
  - mclust
  - scISR
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# pca-dimensionality-reduction

## Summary

Reduces high-dimensional scRNA-seq expression matrices (raw or imputed, e.g. the Goolam E-MTAB-3321 CPM data) to 50 principal components before k-means clustering, so that clustering ARI against known cell-type labels can be computed on a denoised low-dimensional representation.

## When to use

Use when evaluating imputation or preprocessing effects on single-cell clustering: after obtaining a gene-by-cell matrix (raw or imputed), reduce it with 50-component PCA prior to k-means clustering and ARI computation.

## When NOT to use

- When the analysis goal is dimensionality reduction for visualization of already-normalized feature tables rather than a controlled ARI comparison between raw and imputed data.
- When the dataset is small enough that full PCA is unnecessary or when a different embedding (e.g. for trajectory inference) is the target — this skill is specific to the 50-component PCA feeding k-means in the paper's benchmark.

## Inputs

- Gene-by-cell scRNA-seq expression matrix (e.g. Goolam E-MTAB-3321 CPM values, optionally log2-transformed when the value range exceeds 100)
- scISR-imputed expression matrix from the same data
- Ground-truth cell-type label vector (Goolam$label)

## Outputs

- 50-component PCA score matrix (prcomp_irlba $x) for each input matrix
- k-means cluster assignments with k equal to the number of true cell types
- Adjusted Rand Index values for raw vs imputed clusterings (e.g. written to ari_results.csv)

## How to apply

Transpose the gene-by-cell matrix so cells are rows, optionally filtering genes with only zero entries (e.g. raw[rowSums(raw != 0) > 0, ]), then run truncated PCA with 50 components using irlba::prcomp_irlba and take the resulting score matrix ($x). Apply PCA separately to the raw and the scISR-imputed matrices so their clusterings are comparable. Cluster the PCA scores with k-means, setting k equal to the true number of cell types (length(unique(label))), using nstart = 2000 and iter.max = 2000, with a fixed seed (set.seed(1)) for reproducibility. Judge success by computing the adjusted Rand Index with mclust::adjustedRandIndex against the known labels — in the paper's Goolam benchmark, PCA of imputed data reaches ARI ≈ 0.641 vs ≈ 0.533 for raw data, so the imputed-data ARI should exceed the raw-data ARI.

## Related tools

- **irlba** (Provides prcomp_irlba for truncated PCA with n = 50 components on the transposed (cells x genes) matrix)
- **mclust** (Provides adjustedRandIndex to score k-means clusters from the PCA scores against known cell-type labels)
- **R stats (kmeans)** (Performs k-means clustering on the 50-component PCA scores with nstart = 2000, iter.max = 2000)

## Examples

```
set.seed(1); pca_raw <- irlba::prcomp_irlba(t(raw_filer), n = 50)$x; cluster_raw <- kmeans(pca_raw, length(unique(label)), nstart = 2000, iter.max = 2000)$cluster; adjustedRandIndex(cluster_raw, label)
```

## Evaluation signals

- PCA must be run with n = 50 components via prcomp_irlba on the transposed matrix; the output $x has as many rows as cells.
- k-means must use k = length(unique(label)) (for Goolam, the true number of embryo cell types), with nstart = 2000 and iter.max = 2000, under a fixed seed (set.seed(1)).
- Raw-data ARI should be approximately 0.533 and scISR-imputed ARI approximately 0.641 on the Goolam dataset; the imputed ARI must exceed the raw ARI.
- Comparison is only valid if PCA is applied identically and separately to raw and imputed matrices.
- ARI is computed with mclust::adjustedRandIndex against the ground-truth labels, rounded to 3 decimals.

## Limitations

- Results depend on random seed and k-means initialization; the README fixes set.seed(1) but ARI values may vary across seeds/implementations.
- The number of PCA components (50) and k-means parameters are tuned for the paper's benchmark datasets (124-cell Goolam, etc.) and may need adjustment elsewhere.
- True k requires known cell-type labels, limiting use to datasets with ground-truth annotations.
- Preprocessing matters: genes with only zero entries are filtered from raw data, and datasets with value range larger than 100 are log2-transformed before this step.

## Evidence

- [results] k-means clustering with true number of cell types k to compute ARI against known labels: "we use k-means to cluster the cells using the true number of cell types k as the number of clusters."
- [readme] PCA 50 components on raw and imputed matrices in README: "pca_raw <- irlba::prcomp_irlba(t(raw_filer), n = 50)$x"
- [readme] k-means parameters and ARI computation in README: "cluster_raw <- kmeans(pca_raw, length(unique(label)), nstart = 2000, iter.max = 2000)$cluster"
- [discussion] expected ARI values from source card validation: [evidence span withheld — anchor status: not_found]
