---
name: cluster-number-selection-elbow-silhouette
description: Use when clustering PCA-reduced single-cell expression data (e.g., scSPARKL-normalized
  wide-format matrices in Parquet/CSV) and you need to justify k via elbow (WCSS)
  and silhouette criteria before downstream per-cluster DGE t-tests.
when_to_use_negative:
- Input is a raw, high-dimensional expression matrix that has not been normalized
  or PCA-reduced — elbow/silhouette on unnormalized, dropout-heavy scRNA-seq data
  is unreliable; run scSPARKL normalization and PCA first
- You need graph/community-based clustering (e.g., Louvain/Leiden as in Seurat or
  Scanpy) rather than K-means with a globally fixed k
- The dataset has no plausible range of cluster numbers (e.g., a single homogeneous
  population), making elbow/silhouette selection meaningless
edam_operation: http://edamontology.org/operation_3432
tools:
- name: scSPARKL
  role: Provides the normalized wide-format matrix and the Spark-native PCA representation
    on which K-means, elbow, and silhouette computations are run
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Spark
  role: Distributed in-memory engine executing the K-means clustering and quality-summary/analyses-folder
    output over Parquet-backed data
- name: Apache Parquet
  role: Efficient columnar storage format for the input normalized expression matrices
    consumed by the clustering step
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:0251bf3a15a5bfad20d1f8bba4e75c5d3314ad1fb30d5d192fb5ca2962ca6f87
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# cluster-number-selection-elbow-silhouette

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects the number of K-means clusters (k) for single-cell RNA-seq data by computing within-cluster sum of squares (elbow plot) and silhouette scores across candidate k values on PCA-reduced data, as implemented in the Spark-based scSPARKL pipeline. Validated on the Jurkat–293T cell mixture where k=2 recovers human vs mouse separation with ARI ≈ 0.90.

## When to use

Use when clustering PCA-reduced single-cell expression data (e.g., scSPARKL-normalized wide-format matrices in Parquet/CSV) and you need to justify k via elbow (WCSS) and silhouette criteria before downstream per-cluster DGE t-tests.

## When NOT to use

- Input is a raw, high-dimensional expression matrix that has not been normalized or PCA-reduced — elbow/silhouette on unnormalized, dropout-heavy scRNA-seq data is unreliable; run scSPARKL normalization and PCA first
- You need graph/community-based clustering (e.g., Louvain/Leiden as in Seurat or Scanpy) rather than K-means with a globally fixed k
- The dataset has no plausible range of cluster numbers (e.g., a single homogeneous population), making elbow/silhouette selection meaningless

## Inputs

- Wide-format normalized expression matrix (.csv or .parquet) from the scSPARKL pipeline
- PCA-reduced representation of the cells produced by scSPARKL's Spark-native PCA
- Range of candidate k values for K-means

## Outputs

- Elbow plot (within-cluster sum of squares vs k) saved to the auto-generated analyses folder
- Silhouette scores per candidate k
- K-means cluster assignments per cell at the selected k
- Top 10 differentially expressed genes per cluster pair (gene, statistic, p-value) written to CSV

## How to apply

Run K-means over a range of candidate k values on the PCA-reduced representation of the normalized (CPM-like 'Global/Simple' or quantile-normalized) expression matrix. For each k compute the within-cluster sum of squares for the elbow plot and the silhouette score, then select the k at the elbow / with the highest silhouette — on the Jurkat–293T mixture this yields k=2 matching human vs mouse cells. Once k is chosen, treat each cluster as a group and run per-gene two-sample t-tests for every cluster pair on the normalized values, ranking genes by significance and writing the top 10 DE genes (gene, statistic, p-value) per pair to CSV. Save the elbow plot and silhouette scores in the automatically generated analyses folder. The rationale is that PCA (exclusively Spark-based in scSPARKL) reduces dropout-heavy scRNA-seq noise so that K-means distance structure and silhouette values are meaningful.

## Related tools

- **scSPARKL** (Provides the normalized wide-format matrix and the Spark-native PCA representation on which K-means, elbow, and silhouette computations are run) — https://www.github.com/asif7adil/scSPARKL
- **Apache Spark** (Distributed in-memory engine executing the K-means clustering and quality-summary/analyses-folder output over Parquet-backed data)
- **Apache Parquet** (Efficient columnar storage format for the input normalized expression matrices consumed by the clustering step)

## Examples

```
from pyspark.ml.clustering import KMeans; model = KMeans(k=2, seed=1).fit(pca_df); preds = model.transform(pca_df)
```

## Evaluation signals

- On the Jurkat–293T cell mixture, chosen k should equal 2 and clustering should achieve ARI ≈ 0.90 against the expected human vs mouse labels
- Silhouette score at the selected k should be the maximum over the candidate k range
- Elbow plot should show a clear bend in within-cluster sum of squares at the selected k
- Per-cluster-pair DE CSVs should contain exactly 10 rows per pair with non-null gene, t-statistic, and p-value columns
- Elbow plot and silhouette scores should be present in the automatically generated analyses folder

## Limitations

- The scSPARKL repository is not being actively maintained; a new version is under development (per README)
- The article acknowledges no benchmarking of clustering against widely adopted frameworks such as Seurat and Scanpy
- Spark cluster-mode operations (each node with separate memory allocation) are not yet optimized, which limits scaling of this clustering step on large clusters; data melting to tall format is a RAM workaround needing further optimization
- Tested only on Windows 11 standalone Spark; requires Java, Spark 3.0+, and Python 3.9+

## Evidence

- [results] ARI on Jurkat–293T: "scSPARKL achieved an ARI of approximately 0.90, indicating strong agreement with the expected separation between human and mouse cells"
- [results] Clustering distinguishes cell types: "in the Jurkat–293T cell mixture, scSPARKL clearly distinguished between mouse and human cells"
- [readme] PCA is Spark-native: "The dimension reduction is performed using PCA.
The PCA implementation is exclusively spark based."
- [readme] Filtering and quality summaries precede analysis: "The quality summaries are then passed as arguments to the `data_filter()` to filter out the unwanted genes and cells."
