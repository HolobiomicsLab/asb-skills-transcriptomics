---
name: kmeans-clustering-with-true-labels
description: Use when evaluating whether an imputation method improves single-cell
  cluster analysis quality on datasets with known cell-type labels, comparing imputed
  vs raw expression matrices.
when_to_use_negative:
- Datasets without known cell-type labels or a defensible true number of cell types
  k — the protocol requires k from ground truth.
- Tasks aiming to discover cell clusters de novo (e.g. trajectory inference or atlas
  visualization) rather than benchmark imputation quality against known labels.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scISR
  role: Provides the imputed expression matrices whose clustering quality is benchmarked;
    README example runs k-means/ARI on scISR output vs raw data
  repo: https://github.com/duct317/scISR
- name: mclust
  role: adjustedRandIndex computes the ARI between k-means clusters and known cell
    labels
- name: irlba
  role: prcomp_irlba performs approximate PCA (n = 50) on cell profiles before k-means
    clustering
- name: R stats
  role: Supplies the k-means clustering and one-sided Wilcoxon signed-rank test (p
    = 0.007) used to assess PI differences
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-022-06500-4
    title: A novel method for single-cell data imputation using subspace regression
  source_content_hash: sha256:b5d6d1981dfc3877d67ed93cb718e5531a5949fc6e6ecb9e587760ab03bfb69b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scISR
  tools:
  - R stats
  - irlba
  - mclust
  - scISR
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# kmeans-clustering-with-true-labels

## Summary

Cluster scRNA-seq cells with k-means using the true number of cell types k as the number of clusters, then score cluster quality against known labels (ARI) to benchmark imputation methods against raw data.

## When to use

Use when evaluating whether an imputation method improves single-cell cluster analysis quality on datasets with known cell-type labels, comparing imputed vs raw expression matrices.

## When NOT to use

- Datasets without known cell-type labels or a defensible true number of cell types k — the protocol requires k from ground truth.
- Tasks aiming to discover cell clusters de novo (e.g. trajectory inference or atlas visualization) rather than benchmark imputation quality against known labels.

## Inputs

- Raw scRNA-seq expression matrix (genes x cells)
- Imputed expression matrix from an imputation method (scISR, scImpute, SAVER, MAGIC, DrImpute, scScope, DCA, DeepImpute, or BISCUIT)
- Vector of known cell-type labels per cell (e.g. Goolam$label)

## Outputs

- Cluster assignments per cell from k-means
- ARI between k-means clusters and known labels
- Purity Index (PI) per dataset/method for benchmark aggregation (e.g. stability_summary.csv)

## How to apply

Run k-means on the cells of each raw or imputed expression matrix with the number of clusters set to the true number of cell types k (e.g. Goolam, Fan, Yan datasets), using PCA-reduced profiles as in the scISR README (prcomp_irlba to 50 components, nstart=2000, iter.max=2000). Score assignments with the Adjusted Rand Index against known cell labels via mclust::adjustedRandIndex. In the article's benchmark, this k-means-with-true-k protocol underlies the Purity Index (PI) comparison across the 25 real scRNA-seq datasets, where scISR's PI exceeded raw data in 15 of 25 datasets (one-sided Wilcoxon signed-rank test p = 0.007). Set the random seed (set.seed(1)) before clustering so raw-vs-imputed comparisons are reproducible, and filter all-zero genes (rowSums(raw != 0) > 0) before PCA on raw data.

## Related tools

- **scISR** (Provides the imputed expression matrices whose clustering quality is benchmarked; README example runs k-means/ARI on scISR output vs raw data) — https://github.com/duct317/scISR
- **mclust** (adjustedRandIndex computes the ARI between k-means clusters and known cell labels)
- **irlba** (prcomp_irlba performs approximate PCA (n = 50) on cell profiles before k-means clustering)
- **R stats** (Supplies the k-means clustering and one-sided Wilcoxon signed-rank test (p = 0.007) used to assess PI differences)

## Examples

```
cluster_raw <- kmeans(pca_raw, length(unique(label)), nstart=2000, iter.max=2000)$cluster; adjustedRandIndex(cluster_raw, label)
```

## Evaluation signals

- ARI is reported between k-means clusters and known labels for both raw and imputed data; imputed-data ARI should typically exceed raw-data ARI (e.g. scISR exceeded raw in 15/25 datasets).
- Number of clusters passed to kmeans equals length(unique(label)) — the true number of cell types k.
- Seed-controlled reproducibility: set.seed(1) with nstart=2000, iter.max=2000 should yield identical clusters on re-run.
- One-sided Wilcoxon signed-rank test over per-dataset PI values should reproduce p = 0.007 for scISR vs raw across the 25 datasets.
- Genes with all-zero counts are removed before PCA on raw data (rowSums(raw != 0) > 0).

## Limitations

- Requires pre-known cell-type labels and the true k; not applicable to label-free exploratory clustering.
- The 25 real scRNA-seq datasets are not all named with accession IDs in the article, limiting full reproducibility.
- K-means results depend on random initialization; the README fixes set.seed(1) but other seeds may shift ARI values.
- Pipeline follows dataset-specific normalization (e.g. log2 transformation when value range exceeds 100), so raw vs imputed comparisons assume consistent preprocessing.

## Evidence

- [results] k-means with true k and ARI: "we use k-means to cluster the cells using the true number of cell types k as the number of clusters."
- [readme] README clustering recipe: "pca_raw <- irlba::prcomp_irlba(t(raw_filer), n = 50)$x"
- [readme] README k-means call: "cluster_raw <- kmeans(pca_raw, length(unique(label)),
                      nstart = 2000, iter.max = 2000)$cluster"
- [readme] README ARI scoring: "print(paste('ARI of clusters using raw data:', round(adjustedRandIndex(cluster_raw, label),3)))"
- [results] PI benchmark finding: [evidence span withheld — anchor status: not_found]
- [results] Wilcoxon significance: [evidence span withheld — anchor status: not_found]
