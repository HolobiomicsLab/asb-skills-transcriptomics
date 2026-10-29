---
name: single-cell-kmeans-clustering
description: Use when you have a wide-format normalized scRNA-seq expression matrix
  with a PCA-reduced representation and need unbiased cell clustering, optimal-k selection,
  and cluster-pair differential expression ranking.
when_to_use_negative:
- Input is raw count data that has not passed scSPARKL's QC/filtering (percent_ercc
  > 10%, percent_mito > 5%, cells expressing < 10 genes, genes expressed in < 3 cells,
  dropout rate > 70%) and normalization
- You expect many non-convex or highly irregular clusters that K-means cannot capture
- You need multi-omics integration (CITE-seq, scATAC-seq, scBS-seq) — such Spark-based
  pipelines are not yet developed per the paper's discussion
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3306
- http://edamontology.org/topic_3170
tools:
- name: scSPARKL
  role: Framework producing the normalized matrix and Spark-based PCA representation
    that K-means runs on, and writing clustering/DGE outputs to the analyses folder
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Spark
  role: Distributed compute engine underlying scSPARKL's PCA and scalable data handling
    for clustering
- name: Apache Parquet
  role: Columnar storage format for input/output expression matrices in the pipeline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:b2585a860fc737e49a9d6d8ec37f93a6e97a84419da2eaa6bcf6c13723d1de6b
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
  edam_topics:
  - http://edamontology.org/topic_3306
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# single-cell-kmeans-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

K-means clustering of PCA-reduced normalized single-cell expression data, with k selected via elbow (within-cluster sum of squares) and silhouette score analysis, followed by per-cluster-pair two-sample t-test DGE ranking. Grounded in scSPARKL's Spark/Parquet pipeline, which achieved ARI ≈ 0.90 on the Jurkat–293T cell mixture.

## When to use

Use when you have a wide-format normalized scRNA-seq expression matrix with a PCA-reduced representation and need unbiased cell clustering, optimal-k selection, and cluster-pair differential expression ranking.

## When NOT to use

- Input is raw count data that has not passed scSPARKL's QC/filtering (percent_ercc > 10%, percent_mito > 5%, cells expressing < 10 genes, genes expressed in < 3 cells, dropout rate > 70%) and normalization
- You expect many non-convex or highly irregular clusters that K-means cannot capture
- You need multi-omics integration (CITE-seq, scATAC-seq, scBS-seq) — such Spark-based pipelines are not yet developed per the paper's discussion

## Inputs

- Wide-format normalized single-cell expression matrix (scSPARKL output, .csv or .parquet)
- PCA-reduced representation of the same cells (Spark-based PCA output)
- Tall-format normalized expression values (for per-gene two-sample t-tests)

## Outputs

- Cluster assignments per cell for the selected k
- Elbow plot of within-cluster sum of squares across candidate k values
- Silhouette scores per candidate k
- Top 10 differentially expressed genes per cluster pair (gene, statistic, p-value) as CSV, saved in the analyses folder

## How to apply

Load the wide-format normalized matrix and its PCA-reduced representation (scSPARKL's PCA is exclusively Spark-based; inputs are .csv or .parquet). Run K-means over a range of candidate k values on the PCA-reduced data; for each k compute within-cluster sum of squares for an elbow plot and the silhouette score, and select the k at the elbow / with the highest silhouette. After clustering, perform a per-gene two-sample t-test on normalized expression values for every pair of clusters, recording test statistics and p-values. Rank genes by t-test significance per cluster pair and write the top 10 differentially expressed genes (gene, statistic, p-value) per pair to CSV, saving the elbow plot and silhouette scores in the automatically generated analyses folder.

## Related tools

- **scSPARKL** (Framework producing the normalized matrix and Spark-based PCA representation that K-means runs on, and writing clustering/DGE outputs to the analyses folder) — https://www.github.com/asif7adil/scSPARKL
- **Apache Spark** (Distributed compute engine underlying scSPARKL's PCA and scalable data handling for clustering)
- **Apache Parquet** (Columnar storage format for input/output expression matrices in the pipeline)

## Examples

```
from sklearn.cluster import KMeans; from sklearn.metrics import silhouette_score; ks=range(2,10); scores=[silhouette_score(pca_data, KMeans(n_clusters=k).fit_predict(pca_data)) for k in ks]
```

## Evaluation signals

- Silhouette score at the selected k should be the highest across the candidate range; the selected k should also sit at the elbow of the within-cluster-sum-of-squares plot
- On labeled mixtures (e.g. Jurkat–293T), Adjusted Rand Index against expected cell identity should be high — scSPARKL reported ARI ≈ 0.90 for the human vs mouse separation
- Output CSVs must contain gene, statistic, and p-value columns with exactly 10 rows per cluster pair
- Elbow plot and silhouette scores must be present in the automatically generated analyses folder
- Clusters should biologically separate expected populations (e.g. mouse vs human cells in the Jurkat–293T mixture)

## Limitations

- Cluster mode operations in Spark are not yet optimized and are deferred to future development
- The repository is not being actively maintained; a new version is under development
- Data melting is used as a workaround for RAM limitations and needs further optimization
- No benchmarking against Seurat and Scanpy was performed
- Current pipeline only accepts .csv and .parquet input files

## Evidence
