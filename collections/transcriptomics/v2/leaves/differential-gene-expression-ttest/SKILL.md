---
name: differential-gene-expression-ttest
description: Use when you have Spark-normalized scRNA-seq expression (tall/wide format)
  plus K-means cluster assignments, and need per-gene two-sample t-test DE rankings
  between every pair of clusters.
when_to_use_negative:
- Input is raw unnormalized count data — normalization (Global/CPM-like or Quantile)
  must be run first in scSPARKL
- Clusters have not yet been defined — the t-test is per cluster pair, so it presupposes
  completed K-means clustering on PCA-reduced data
- Comparison of DE methods against Seurat or Scanpy benchmarks is required — the paper
  explicitly notes no benchmarking against these frameworks was performed
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scSPARKL
  role: Framework providing normalization (tall/wide output), Spark-based PCA, K-means
    clustering, and the analyses folder where DE t-test results are written
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Spark
  role: Distributed engine executing the per-gene t-test computation and the upstream
    clustering/dimensionality-reduction steps
- name: Apache Parquet
  role: Efficient storage format for the input/output normalized expression matrices
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:1d96f455b872a8133abe59d249e523eb3a3cdb232d3bb4e882f8b8a49cf736b2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/asif7adil/scSparkl
  tools:
  - Apache Parquet
  - Apache Spark
  - scSPARKL
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# differential-gene-expression-ttest

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Ranks differentially expressed genes between K-means clusters by running a per-gene two-sample t-test on normalized scRNA-seq expression values within the scSPARKL Spark pipeline, writing the top 10 genes per cluster pair.

## When to use

Use when you have Spark-normalized scRNA-seq expression (tall/wide format) plus K-means cluster assignments, and need per-gene two-sample t-test DE rankings between every pair of clusters.

## When NOT to use

- Input is raw unnormalized count data — normalization (Global/CPM-like or Quantile) must be run first in scSPARKL
- Clusters have not yet been defined — the t-test is per cluster pair, so it presupposes completed K-means clustering on PCA-reduced data
- Comparison of DE methods against Seurat or Scanpy benchmarks is required — the paper explicitly notes no benchmarking against these frameworks was performed

## Inputs

- Wide-format normalized expression matrix (.csv or .parquet)
- Tall-format normalized expression data
- PCA-reduced representation of cells (Spark-based PCA)
- K-means cluster assignments per cell (k chosen via elbow plot / silhouette score)

## Outputs

- CSV of top 10 differentially expressed genes per cluster pair, with columns (gene, statistic, p-value)
- T-test statistics and p-values recorded for every gene per cluster pair
- Elbow plot and silhouette scores saved in the auto-generated analyses folder

## How to apply

First normalize the melted tall-format data with scSPARKL (Global/Simple CPM-like normalization or Quantile Normalization), which returns one wide-format and one tall-format normalized matrix. Cluster cells with Spark-based K-means on the PCA-reduced representation, selecting k via the elbow plot of within-cluster sum of squares and the silhouette score. For every pair of clusters, run a two-sample t-test on each gene's normalized expression values, recording the test statistic and p-value. Rank genes by t-test significance within each cluster pair and write the top 10 DE genes (gene, statistic, p-value) to CSV, saving results in the analyses folder that scSPARKL auto-generates. On the Jurkat–293T benchmark this workflow correctly separated human from mouse cells, with clustering ARI of approximately 0.90.

## Related tools

- **scSPARKL** (Framework providing normalization (tall/wide output), Spark-based PCA, K-means clustering, and the analyses folder where DE t-test results are written) — https://www.github.com/asif7adil/scSPARKL
- **Apache Spark** (Distributed engine executing the per-gene t-test computation and the upstream clustering/dimensionality-reduction steps)
- **Apache Parquet** (Efficient storage format for the input/output normalized expression matrices)

## Examples

```
from scipy.stats import ttest_ind; t, p = ttest_ind(expr[cluster_a_idx], expr[cluster_b_idx], axis=1)  # per-gene two-sample t-test on normalized expression
```

## Evaluation signals

- Every cluster pair yields exactly 10 ranked DE genes with non-empty (gene, statistic, p-value) columns in the output CSV
- Genes are ordered by t-test significance (p-value) within each cluster pair
- Elbow plot and silhouette scores exist in the auto-generated analyses folder and support the chosen k
- Upstream clustering quality: on the Jurkat–293T cell mixture, ARI should be approximately 0.90 vs the expected human/mouse separation
- Input matrix is in one of the two normalized forms scSPARKL emits: wide-format or tall-format normalized data (Global/CPM-like or Quantile Normalization)

## Limitations

- Repository is not actively maintained; a new version is under development (per README notice)
- Current implementation has been tested only on Microsoft Windows 11; the pipeline accepts only .csv and .parquet inputs
- No benchmarking against Seurat or Scanpy was performed, and Spark cluster-mode operations are not yet optimized

## Evidence

- [other] Per-pair two-sample t-test defined on the Jurkat-293T benchmark: [evidence span withheld — anchor status: not_found]
- [other] Ranking and CSV output specification: [evidence span withheld — anchor status: not_found]
