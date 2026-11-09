---
name: adjusted-rand-index-cluster-evaluation
description: Use when you have scRNA-seq expression data with ground-truth cell-type
  labels and need a chance-corrected metric to compare clustering quality across imputation
  methods or against raw data.
when_to_use_negative:
- The dataset has no ground-truth cell-type labels — ARI cannot be computed and unsupervised
  validation is required instead
- You need biological insight into cluster composition (marker genes, novel cell types)
  — ARI only measures agreement with predefined labels, not discovery
- Evaluating clustering where the number of clusters is unknown — the protocol requires
  k set to the true number of cell types
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: mclust
  role: Provides adjustedRandIndex used to score k-means clusters against known cell
    labels
- name: irlba
  role: Computes truncated PCA (50 components) of raw and imputed matrices prior to
    k-means clustering
- name: scISR
  role: Generates the imputed data whose clustering ARI is compared against raw-data
    ARI and competing imputation methods
  repo: https://github.com/duct317/scISR
- name: R stats
  role: Supplies the kmeans implementation (nstart = 2000, iter.max = 2000) used for
    clustering
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-06500-4
    title: A novel method for single-cell data imputation using subspace regression
  source_content_hash: sha256:a742217b6396d4c7d12aed0e121a9358478e1514320750fc9aaf7005bce4c143
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scISR
  tools:
  - MAGIC
  - R stats
  - SAVER
  - irlba
  - mclust
  - scISR
  - scImpute
  - scScope
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# adjusted-rand-index-cluster-evaluation

## Summary

Evaluates cell-type clustering quality by computing the Adjusted Rand Index (ARI) between k-means clusters (with k set to the true number of cell types) and known cell labels, used here to quantify whether scRNA-seq imputation (e.g. scISR) improves clustering over raw data.

## When to use

Use when you have scRNA-seq expression data with ground-truth cell-type labels and need a chance-corrected metric to compare clustering quality across imputation methods or against raw data.

## When NOT to use

- The dataset has no ground-truth cell-type labels — ARI cannot be computed and unsupervised validation is required instead
- You need biological insight into cluster composition (marker genes, novel cell types) — ARI only measures agreement with predefined labels, not discovery
- Evaluating clustering where the number of clusters is unknown — the protocol requires k set to the true number of cell types

## Inputs

- Preprocessed scRNA-seq expression matrix (cells x genes; CPM/RPKM/TPM/FPKM variants, log2-transformed if value range exceeds 100)
- Vector of known cell-type labels (e.g. Goolam E-MTAB-3321 labels)
- True number of cell types k (length(unique(label)))

## Outputs

- ARI value per clustering (raw and each imputed variant), e.g. raw 0.533 vs scISR 0.641 on Goolam
- ari_results.csv with raw and imputed ARI values; optionally ari_comparison_table.csv with per-dataset ARIs, mean ARIs per method, and Wilcoxon p-values

## How to apply

First embed cells with PCA (in the paper, 50 components computed with irlba::prcomp_irlba on the cell-by-gene matrix), then run k-means with the number of clusters set to the true number of cell types (nstart = 2000, iter.max = 2000, set.seed(1) for reproducibility). Compute the Adjusted Rand Index between the k-means cluster assignment and the known labels using mclust::adjustedRandIndex. Apply this identically to each data variant being compared (raw vs. imputed by scISR, MAGIC, scImpute, SAVER, scScope, scGNN) so ARIs are directly comparable. ARI corrects for chance agreement, so values near 0 indicate random clustering and higher values (e.g. scISR 0.641 vs raw 0.533 on Goolam) indicate label-consistent clusters. Write the per-method ARIs to a results file (e.g. ari_results.csv) and, across multiple datasets, aggregate into mean ARIs per method with one-sided Wilcoxon signed-rank tests.

## Related tools

- **mclust** (Provides adjustedRandIndex used to score k-means clusters against known cell labels)
- **irlba** (Computes truncated PCA (50 components) of raw and imputed matrices prior to k-means clustering)
- **scISR** (Generates the imputed data whose clustering ARI is compared against raw-data ARI and competing imputation methods) — https://github.com/duct317/scISR
- **R stats** (Supplies the kmeans implementation (nstart = 2000, iter.max = 2000) used for clustering)

## Examples

```
library(mclust); library(irlba); set.seed(1); pca <- irlba::prcomp_irlba(t(imputed), n=50)$x; cl <- kmeans(pca, length(unique(label)), nstart=2000, iter.max=2000)$cluster; adjustedRandIndex(cl, label)
```

## Evaluation signals

- On Goolam (124 cells), ARI should be ~0.641 for scISR-imputed vs ~0.533 for raw data, with scISR exceeding raw (validation criterion)
- Identical PCA (50 components), k-means (true k, seed 1, nstart 2000) and ARI settings applied to every data variant so results are comparable
- ARI is chance-corrected: a value near 0 (e.g. scScope at 0.1) signals clustering no better than random; raw-data ARI (~0.504 mean over 25 datasets) is the baseline to beat
- ari_results.csv (or ari_comparison_table.csv) is produced containing raw and imputed ARI values per dataset
- Across the 25-dataset benchmark, mean ARIs should reproduce reported values (scISR 0.571 vs raw 0.504) with significant one-sided Wilcoxon p-values (3.2e-5 vs raw, 9.8e-6 vs competing methods)

## Limitations

- Requires the true number of cell types k in advance; the benchmark deliberately uses k = true number of cell types
- ARI depends on clustering stochasticity — k-means results vary without a fixed seed (README sets set.seed(1), nstart = 2000)
- ARI measures agreement with known labels only; the article notes imputation methods can over-smoothen and remove meaningful cell-to-cell stochasticity, which ARI may not capture
- Evaluation depends on preprocessing choices (log2 transform when value range exceeds 100; removing genes expressed in fewer than two cells or with less than one percent non-zero entries), which affect ARI comparability

## Evidence

- [results] we use k-means to cluster the cells using the true number of cell types k as the number of clusters.: "we use k-means to cluster the cells using the true number of cell types k as the number of clusters."
- [results] For datasets with the range of values larger than 100, we rescale the data using log transformation (base 2).: "For datasets with the range of values larger than 100, we rescale the data using log transformation (base 2)."
- [readme] pca_raw <- irlba::prcomp_irlba(t(raw_filer), n = 50)$x: "pca_raw <- irlba::prcomp_irlba(t(raw_filer), n = 50)$x"
- [readme] adjustedRandIndex(cluster_raw, label): "print(paste('ARI of clusters using raw data:', round(adjustedRandIndex(cluster_raw, label),3)))"
- [intro] tending to over-smoothen and remove the cell-to-cell stochasticity that represents meaningful biological variations in gene expression.: "tending to over-smoothen and remove the cell-to-cell stochasticity that represents meaningful biological variations in gene expression."
