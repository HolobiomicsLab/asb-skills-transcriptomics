---
name: k-means-cell-clustering
description: Use when you need to quantify whether an imputation method (e.g. scISR
  vs MAGIC, scImpute, SAVER, scScope, scGNN or raw data) improved cell-type clustering
  of an scRNA-seq dataset with known cell-type labels, measured by ARI.
when_to_use_negative:
- The number of true cell types k is unknown — the benchmark protocol requires k to
  be set to the true number of cell types, so an unsupervised k must be estimated
  by other means
- The goal is to evaluate imputation itself on dropout-free or labeled-truth-free
  data — k-means/ARI validation needs ground-truth cell labels and is meaningless
  for purely exploratory clustering
- Evaluating raw (unimputed) data as the final answer — the article shows raw-data
  ARI is systematically lower (e.g. Goolam 0.533 raw vs 0.641 scISR), so clustering
  raw data without an imputed comparison understates the achievable quality
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scISR
  role: Supplies the imputed expression matrix (hyper-geometric dropout testing +
    subspace regression) whose clusters are compared against the raw-data baseline
  repo: https://github.com/duct317/scISR
- name: irlba
  role: Computes the truncated PCA (n = 50 components) of the transposed raw/imputed
    matrix that feeds k-means
- name: mclust
  role: Provides adjustedRandIndex used to score k-means clusters against known cell
    labels
- name: R stats (kmeans)
  role: Performs the k-means clustering with nstart = 2000, iter.max = 2000 on the
    PCA embedding
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_008
  source_papers:
  - doi: 10.1038/s41598-022-06500-4
    title: A novel method for single-cell data imputation using subspace regression
  source_content_hash: sha256:25d7c83257866463dd948f2800ecbc5e1817e6ce30aaac3870ba275a2ab838b9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scISR
  tools:
  - BISCUIT
  - DCA
  - DeepImpute
  - DrImpute
  - MAGIC
  - R stats
  - R stats (kmeans)
  - SAVER
  - irlba
  - mclust
  - scISR
  - scImpute
  - scScope
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# k-means-cell-clustering

## Summary

Clusters single-cell RNA-seq expression matrices with k-means (k set to the true number of cell types) on 50-component PCA embeddings to benchmark imputation quality via Adjusted Rand Index against known cell labels.

## When to use

Use when you need to quantify whether an imputation method (e.g. scISR vs MAGIC, scImpute, SAVER, scScope, scGNN or raw data) improved cell-type clustering of an scRNA-seq dataset with known cell-type labels, measured by ARI.

## When NOT to use

- The number of true cell types k is unknown — the benchmark protocol requires k to be set to the true number of cell types, so an unsupervised k must be estimated by other means
- The goal is to evaluate imputation itself on dropout-free or labeled-truth-free data — k-means/ARI validation needs ground-truth cell labels and is meaningless for purely exploratory clustering
- Evaluating raw (unimputed) data as the final answer — the article shows raw-data ARI is systematically lower (e.g. Goolam 0.533 raw vs 0.641 scISR), so clustering raw data without an imputed comparison understates the achievable quality

## Inputs

- scRNA-seq expression matrix (genes x cells) as CPM/RPM/TPM/RPKM/FPKM/UMI values, raw or imputed
- Vector of known cell-type labels for the cells (e.g. Goolam$label from the scISR package)
- Number of true cell types k (derived from the label vector via length(unique(label)))

## Outputs

- Cell cluster assignments from k-means on the 50-component PCA embedding
- Adjusted Rand Index (ARI) of each clustering against the ground-truth labels
- Per-dataset ARI comparison records (e.g. ari_results.csv or runtime_ari_comparison.csv rows for raw vs each imputation method)

## How to apply

First preprocess the expression matrix: remove genes expressed in fewer than two cells and genes with less than one percent non-zero entries, and log2-transform (base 2) datasets whose value range exceeds 100. Project cells into 50 principal components using irlba::prcomp_irlba on the transposed matrix, separately for the raw and imputed versions. Run k-means on the PCA coordinates with the number of clusters k equal to the true number of cell types, using nstart = 2000 and iter.max = 2000 for stable optima. Fix the random seed (e.g. set.seed(1)) so the stochastic clustering is reproducible. Compute the Adjusted Rand Index of the clusters against the known labels with mclust::adjustedRandIndex and compare raw vs imputed ARI values; the imputed ARI should exceed the raw ARI.

## Related tools

- **scISR** (Supplies the imputed expression matrix (hyper-geometric dropout testing + subspace regression) whose clusters are compared against the raw-data baseline) — https://github.com/duct317/scISR
- **irlba** (Computes the truncated PCA (n = 50 components) of the transposed raw/imputed matrix that feeds k-means)
- **mclust** (Provides adjustedRandIndex used to score k-means clusters against known cell labels)
- **R stats (kmeans)** (Performs the k-means clustering with nstart = 2000, iter.max = 2000 on the PCA embedding)

## Examples

```
set.seed(1); pca <- irlba::prcomp_irlba(t(imputed), n = 50)$x; cl <- kmeans(pca, length(unique(label)), nstart = 2000, iter.max = 2000)$cluster; mclust::adjustedRandIndex(cl, label)
```

## Evaluation signals

- ARI of the scISR-imputed clustering exceeds the raw-data ARI (Goolam benchmark: ~0.641 vs ~0.533)
- Per-method mean ARIs across the 25 real datasets match the reported ordering: scISR 0.571 > raw 0.504 > MAGIC 0.461 > SAVER 0.423 > scGNN 0.279 > scImpute 0.286? (scImpute 0.286) > scScope 0.165
- k was set to length(unique(label)) and identical preprocessing (gene filters, log2 for range > 100, 50 PCA components) was applied to raw and imputed matrices before clustering
- Determinism check: with set.seed(1), nstart = 2000, iter.max = 2000, re-running yields the same ARI to 3 decimals
- Aggregated one-sided Wilcoxon signed-rank tests reproduce significance (p ~ 9.8e-6 vs competing methods, 3.2e-5 vs raw)

## Limitations

- k-means requires the true number of cell types k in advance; it cannot be applied when labels or class count are unknown
- Results depend on the preprocessing choices (gene expressed in < 2 cells removed, < 1% non-zero entries removed, log2 transform only when value range exceeds 100); deviating from these may not reproduce reported ARIs
- No software versions for the scISR package or R stats were stated, and key robustness evidence is deferred to supplementary figures, limiting exact reproducibility

## Evidence

- [results] we use k-means to cluster the cells using the true number of cell types k as the number of clusters.: "we use k-means to cluster the cells using the true number of cell types k as the number of clusters."
- [readme] pca_imputed <- irlba::prcomp_irlba(t(imputed), n = 50)$x: "pca_imputed <- irlba::prcomp_irlba(t(imputed), n = 50)$x"
- [readme] print(paste('ARI of clusters using imputed data:', round(adjustedRandIndex(cluster_imputed, label),3))): "print(paste('ARI of clusters using imputed data:', round(adjustedRandIndex(cluster_imputed, label),3)))"
- [results] genes expressed in less than two cells; and (ii) genes that have less than one percent of non-zero-valued entries.: "genes expressed in less than two cells; and (ii) genes that have less than one percent of non-zero-valued entries."
