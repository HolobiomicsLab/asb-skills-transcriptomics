---
name: silhouette-cluster-quality-assessment
description: Use when comparing two clusterings of the same scRNA-seq dataset (e.g.,
  5% vs 10% cell subsamples processed by SCC or SSCC) and you need a numeric measure
  of clustering consistency and per-cell cluster quality.
when_to_use_negative:
- You need an external ground-truth-based accuracy metric (e.g., ARI against known
  cell types) rather than internal consistency between two subsampling schemes
- The dataset is small enough that full-data clustering is feasible and subsampling
  bias is not a concern, making the consistency comparison moot
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SSCC
  role: Spearman subsampling-clustering-classification framework whose clusterings
    are evaluated; uses Spearman-correlation-based feature engineering and random
    projection to improve cross-subsampling consistency
  repo: https://github.com/Japrin/sscClust
- name: R
  role: Implementation language of the sscClust framework used to run the pipelines
    and compute silhouette/NMI metrics
  repo: https://github.com/Japrin/sscClust
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2018.10.003
    title: 'SSCC: A Novel Computational Framework for Rapid and Accurate Clustering
      Large-Scale Single Cell RNA-Seq Data'
  source_content_hash: sha256:0ede9d8e9c92a5ad60a3f72366d2c716a7fc0bfbb91b07898e38587f633ef0e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Japrin/sscClust
  tools:
  - R
  - SSCC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# silhouette-cluster-quality-assessment

## Summary

Quantify clustering quality and cross-scheme consistency for single-cell RNA-seq clusterings using silhouette statistics, comparing clusterings produced under different subsampling schemes (e.g., 5% vs 10% of cells) in the SSCC vs SCC framework of sscClust.

## When to use

Use when comparing two clusterings of the same scRNA-seq dataset (e.g., 5% vs 10% cell subsamples processed by SCC or SSCC) and you need a numeric measure of clustering consistency and per-cell cluster quality.

## When NOT to use

- You need an external ground-truth-based accuracy metric (e.g., ARI against known cell types) rather than internal consistency between two subsampling schemes
- The dataset is small enough that full-data clustering is feasible and subsampling bias is not a concern, making the consistency comparison moot

## Inputs

- 49,300-cell Macosko retina scRNA-seq expression matrix preprocessed for clustering with sscClust
- Cluster label vectors for cells from each condition (SCC 5%, SCC 10%, SSCC 5%, SSCC 10%)
- Per-cell silhouette values from each clustering

## Outputs

- Average silhouette difference between the two subsampling schemes (expected ~0.29 for SCC, ~0.01 for SSCC)
- NMI between the 5% and 10% subsampling-scheme clusterings (expected ~0.60 for SCC, ~0.69 for SSCC)
- Pearson correlation of per-cell silhouette values between schemes (expected ~0.47 for SCC, ~0.58 for SSCC)
- Metrics table (scc_sscc_macosko_benchmark.csv)

## How to apply

Run the subsampling-clustering-classification pipeline (sscClust R package) at two subsampling ratios (5% and 10%) of the 49,300-cell Macosko retina expression matrix, once with plain SCC (k-means clustering on the subsample, then k-NN classification of remaining cells) and once with full SSCC, which adds Spearman-correlation-based feature engineering and random projection. For each resulting labeling, compute per-cell silhouette values, then report the difference of average silhouettes between the two schemes, the NMI between the two labelings, and the Pearson correlation of per-cell silhouette values between schemes. Interpret lower silhouette difference and higher NMI/correlation as better cross-scheme consistency: in the source benchmark, SSCC reduced average silhouette difference from ~0.29 (SCC) to ~0.01 and raised NMI from 0.60 to 0.69 (per-cell silhouette Pearson correlation 0.47 → 0.58). Write all metrics to a CSV (e.g., scc_sscc_macosko_benchmark.csv) for direct SCC-vs-SSCC comparison.

## Related tools

- **SSCC** (Spearman subsampling-clustering-classification framework whose clusterings are evaluated; uses Spearman-correlation-based feature engineering and random projection to improve cross-subsampling consistency) — https://github.com/Japrin/sscClust
- **R** (Implementation language of the sscClust framework used to run the pipelines and compute silhouette/NMI metrics) — https://github.com/Japrin/sscClust

## Examples

```
data("sce.Pollen"); sce.sub <- ssc.run(sce.Pollen, subsampling=T, sub.frac = 0.8, k.batch=11)
```

## Evaluation signals

- Average silhouette difference between the 5% and 10% schemes drops to ~0.01 under SSCC (vs ~0.29 under SCC)
- NMI between the two subsampling schemes reaches ~0.69 under SSCC (vs ~0.60 under SCC)
- Pearson correlation of per-cell silhouette values between schemes rises to ~0.58 under SSCC (vs ~0.47 under SCC)
- All three metrics are present and finite in the output CSV (scc_sscc_macosko_benchmark.csv) for both SCC and SSCC conditions

## Limitations

- The SCC strategy suffers from biases introduced by subsampling, which can decrease clustering accuracy and robustness even while reducing computational complexity from O(n^2) to O(n)
- Metrics are consistency/quality measures between schemes, not validation against true cell-type labels; a good silhouette does not guarantee biologically correct clusters
- Reported benchmark values (0.01 vs 0.29 silhouette difference, NMI 0.60 vs 0.69, correlation 0.47 vs 0.58) are specific to the Macosko retina dataset with k-means and k-NN and may not transfer to other datasets or base clusterers

## Evidence

- [results] switching from SCC to SSCC made the average silhouette difference between the two subsampling schemes negligible (0.01 vs 0.29) and raised the NMI between the two schemes from 0.60 to 0.69: [evidence span withheld — anchor status: not_found]
- [discussion] SSCC improves consistency: [evidence span withheld — anchor status: not_found]
- [discussion] SCC subsampling bias: "the SCC strategy suffers from biases introduced by subsampling which may greatly decrease the clustering accuracy and robustness, although it can reduce the computational complexity from O(n2) to O(n)"
- [abstract] SSCC framework definition: "Spearman subsampling-clustering-classification (SSCC), a new clustering framework based on random projection and feature construction"
- [readme] sscClust README subsampling clustering: "clustering with subsampling and classfification which make it feasible to process thousands cells' data"
