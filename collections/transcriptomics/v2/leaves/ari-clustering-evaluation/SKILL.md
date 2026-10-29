---
name: ari-clustering-evaluation
description: Use when comparing imputation or preprocessing methods on scRNA-seq data
  with known cell-type labels, so that quality is measured as improvement in k-means
  clustering accuracy (ARI) rather than direct value error.
when_to_use_negative:
- 'Cell-type labels are unknown or unreliable: k cannot be set to the true number
  of cell types and ARI against ground truth cannot be computed.'
- The analysis goal is per-gene accuracy of recovered expression values rather than
  downstream clustering quality — masked-value recovery (randomly zeroing 30% of values
  in 40% of genes) is the paper's separate evaluation design.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scISR
  role: Imputation method under evaluation; its output matrix is clustered with k-means
    and scored by ARI against true labels
  repo: https://github.com/duct317/scISR
- name: mclust
  role: Provides adjustedRandIndex used to score k-means clusters against known cell-type
    labels
- name: irlba
  role: prcomp_irlba computes the 50-component PCA of the (raw or imputed) matrix
    prior to k-means clustering
- name: scImpute
  role: Competing imputation method whose output is clustered and scored by ARI for
    baseline comparison
- name: SAVER
  role: Competing imputation method evaluated via the same k-means/ARI protocol
- name: MAGIC
  role: Competing imputation method evaluated via the same k-means/ARI protocol
- name: DrImpute
  role: Competing imputation method evaluated via the same k-means/ARI protocol
- name: scScope
  role: Competing imputation method evaluated via the same k-means/ARI protocol
provenance:
  source_task_ids:
  - task_008
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-06500-4
    title: A novel method for single-cell data imputation using subspace regression
  source_content_hash: sha256:994960eee06c5de9212940971a91c06d643d34da40a3900665ee40f2e4830f2f
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
  - SAVER
  - Splatter
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
yes_rate: 0.5333333333333333
n_claims: 15
---

# ari-clustering-evaluation

## Summary

Evaluates single-cell imputation quality by k-means clustering of imputed (and raw) matrices using the true number of cell types k, then scoring the clustering against known labels with the Adjusted Rand Index (ARI). Used in the scISR paper as the primary metric across 25 real scRNA-seq datasets and 116 Splatter-simulated datasets.

## When to use

Use when comparing imputation or preprocessing methods on scRNA-seq data with known cell-type labels, so that quality is measured as improvement in k-means clustering accuracy (ARI) rather than direct value error.

## When NOT to use

- Cell-type labels are unknown or unreliable: k cannot be set to the true number of cell types and ARI against ground truth cannot be computed.
- The analysis goal is per-gene accuracy of recovered expression values rather than downstream clustering quality — masked-value recovery (randomly zeroing 30% of values in 40% of genes) is the paper's separate evaluation design.

## Inputs

- Preprocessed scRNA-seq expression matrix (raw and/or imputed), genes x cells (e.g., Goolam 'Goolam$data' from GEO accession E-MTAB-3321)
- Vector of known cell-type labels per cell (e.g., 'Goolam$label'), used only to set k and score the clustering
- True number of cell types k (length of unique(label))

## Outputs

- ARI value per (method, dataset, scenario) comparing k-means clusters to true labels
- Comparison tables such as runtime_ari_comparison.csv and simulation_ari_comparison.csv with ARI (and runtime) per imputation method
- Raw-data baseline ARI against which imputed-data improvements are judged

## How to apply

After imputation, project each matrix to 50 principal components (e.g., with irlba::prcomp_irlba on the transposed matrix) and run k-means with the true number of cell types k as the number of clusters. In the paper's evaluation, preprocessing first removes genes expressed in fewer than two cells and genes with less than one percent non-zero-valued entries, and applies a log2 (base 2) transformation when the value range exceeds 100. Compute ARI (e.g., with mclust::adjustedRandIndex) between the k-means clusters and the known cell-type labels for each imputed matrix, plus the raw matrix as a baseline. Aggregate ARI values per method per scenario (e.g., into runtime_ari_comparison.csv or simulation_ari_comparison.csv) and judge a method by whether its ARI exceeds the raw-data baseline and competing methods; scISR is expected to remain highest and stable across dropout rates, normalizations, and quantification schemes. A representative example is Goolam (E-MTAB-3321, 124 cells), where imputed-data ARI should improve on the raw-data ARI.

## Related tools

- **scISR** (Imputation method under evaluation; its output matrix is clustered with k-means and scored by ARI against true labels) — https://github.com/duct317/scISR
- **mclust** (Provides adjustedRandIndex used to score k-means clusters against known cell-type labels)
- **irlba** (prcomp_irlba computes the 50-component PCA of the (raw or imputed) matrix prior to k-means clustering)
- **scImpute** (Competing imputation method whose output is clustered and scored by ARI for baseline comparison)
- **SAVER** (Competing imputation method evaluated via the same k-means/ARI protocol)
- **MAGIC** (Competing imputation method evaluated via the same k-means/ARI protocol)
- **DrImpute** (Competing imputation method evaluated via the same k-means/ARI protocol)
- **scScope** (Competing imputation method evaluated via the same k-means/ARI protocol)

## Examples

```
cluster_imputed <- kmeans(prcomp_irlba(t(imputed), n = 50)$x, length(unique(label)), nstart = 2000, iter.max = 2000)$cluster; round(adjustedRandIndex(cluster_imputed, label), 3)
```

## Evaluation signals

- ARI of imputed data exceeds the raw-data baseline ARI (e.g., Goolam raw baseline 0.68 per the paper's dataset table).
- Setting nstart = 2000 and iter.max = 2000 in kmeans (as in the README) yields stable clusters across random seeds.
- Mean ARI per method is highest and stable for scISR across all dropout-rate, normalization, and quantification scenarios in the 116-dataset simulation table.
- Each method produces exactly one ARI record per dataset/scenario in the output CSV (e.g., runtime_ari_comparison.csv), with runtime or an OOM flag.
- k used in kmeans equals the number of unique true labels (e.g., length(unique(label)) for Goolam).

## Limitations

- ARI depends on clustering success, not on how faithfully true zeros were preserved; methods that alter all zero-valued entries, including true zeros, may still score well or introduce false signals.
- Evaluation on masked data only partially specifies simulation parameters, limiting reproducibility of the 116 simulated datasets.
- For very large datasets (e.g., Darrah, 162,490 cells), some competing imputers fail with out-of-memory errors, so ARI comparisons can be incomplete.

## Evidence

- [results] k-means with true k: "we use k-means to cluster the cells using the true number of cell types k as the number of clusters."
- [readme] README ARI assessment: "print(paste('ARI of clusters using imputed data:', round(adjustedRandIndex(cluster_imputed, label),3)))"
- [abstract] robustness finding: "scISR consistently improves the quality of cluster analysis regardless of dropout rates, normalization techniques, and quantification schemes"
- [results] preprocessing filters: "(i) genes expressed in less than two cells; and (ii) genes that have less than one percent of non-zero-valued entries."
