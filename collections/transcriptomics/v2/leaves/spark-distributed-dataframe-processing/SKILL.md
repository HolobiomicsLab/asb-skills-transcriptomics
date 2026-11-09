---
name: spark-distributed-dataframe-processing
description: Use when scRNA-seq count data (tens of thousands to millions of cells)
  exceeds in-memory tool limits and the analysis must run on Spark with .csv or .parquet
  inputs.
when_to_use_negative:
- Input is a small dataset that comfortably fits in RAM — in-memory tools like Seurat
  or Scanpy are the standard choice, and scSPARKL was not benchmarked against them
- Input formats other than .csv or .parquet — the README states the pipeline currently
  accepts only these two formats
- Multi-omics data (CITE-seq, scATAC-seq, scBS-seq) — Spark-based pipelines for these
  are not yet developed
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scSPARKL
  role: 'Framework providing the Spark-based distributed pipeline: data melting, QC
    summaries, data_filter(), normalization, HVG selection, PCA, and clustering'
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Spark
  role: Distributed in-memory engine whose DataFrames hold the count matrix and run
    the parallel QC/filter/normalization routines
- name: Apache Parquet
  role: Efficient columnar storage format for the input matrix and the QC summary
    outputs in the analyses folder
provenance:
  source_task_ids:
  - task_001
  - task_007
  - task_003
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:274bcdc63c96443008e0c9dda8d7252ce4214cb0d23b5592a6cf92575443b5b2
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
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# spark-distributed-dataframe-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Distributed processing of scRNA-seq count matrices as Spark DataFrames (CSV/Parquet) using scSPARKL on Apache Spark, enabling reshaping, QC aggregation, filtering, and normalization of large single-cell datasets without loading them into RAM.

## When to use

Use when scRNA-seq count data (tens of thousands to millions of cells) exceeds in-memory tool limits and the analysis must run on Spark with .csv or .parquet inputs.

## When NOT to use

- Input is a small dataset that comfortably fits in RAM — in-memory tools like Seurat or Scanpy are the standard choice, and scSPARKL was not benchmarked against them
- Input formats other than .csv or .parquet — the README states the pipeline currently accepts only these two formats
- Multi-omics data (CITE-seq, scATAC-seq, scBS-seq) — Spark-based pipelines for these are not yet developed

## Inputs

- Raw single-cell count matrix in .csv or .parquet format (wide format, e.g. the 3,388-cell Jurkat-293T mixture or 68K-PBMC)
- Cleaned, melted tall-format expression data as a Spark DataFrame
- Spark executor/driver memory and core configuration

## Outputs

- Melted tall-format cleaned count DataFrame
- Cell (Theta1) and gene (Theta2) quality summary Parquet files in the auto-generated analyses folder
- Filtered wide and tall normalized expression DataFrames
- PCA-reduced cell representation and per-cell cluster assignments (cluster_assignments.csv)

## How to apply

Load the raw count matrix (CSV or Parquet) into a Spark distributed DataFrame and clean/melt it from wide to tall format, since the downstream scSPARKL stages operate on tall data. Compute per-cell (total expression, genes expressed, average expression, percent ERCC, percent mitochondrial) and per-gene (cells expressing, total expression, dropout count/percentage, mean expression) quality summaries via groupBy aggregations, apply log1p to count columns, and write the summaries as Parquet files into the auto-generated analyses folder. Pass the quality summaries to data_filter(), whose defaults (percent_ercc > 10%, percent_mito > 5%, cells expressing < 10 genes, genes expressed in < 3 cells, dropout rate > 70%) can be overridden in the filter package. Normalize with Global/Simple (CPM-like) or Quantile normalization to produce wide and tall outputs, then proceed to HVG selection, Spark-native PCA, and clustering.

## Related tools

- **scSPARKL** (Framework providing the Spark-based distributed pipeline: data melting, QC summaries, data_filter(), normalization, HVG selection, PCA, and clustering) — https://www.github.com/asif7adil/scSPARKL
- **Apache Spark** (Distributed in-memory engine whose DataFrames hold the count matrix and run the parallel QC/filter/normalization routines)
- **Apache Parquet** (Efficient columnar storage format for the input matrix and the QC summary outputs in the analyses folder)

## Examples

```
spark-submit scSPARKL.py --input jurkat_293t_counts.csv --format csv; # then data_filter(cell_quality_df, gene_quality_df) in the filter package
```

## Evaluation signals

- Both QC Parquet files exist in the analyses folder and contain required columns including percent ERCC and percent mitochondrial
- Filtered data respects thresholds: cells expressing < 10 genes, genes expressed in < 3 cells, dropout rate > 70%, percent_ercc > 10%, percent_mito > 5%
- Normalization returns both a wide and a tall normalized output from tall-format input
- On the Jurkat-293T benchmark, downstream clustering of CD3D/XIST-separated human vs mouse cells yields an ARI of approximately 0.90
- Full pipeline (QC, filtering, normalization, dimension reduction, clustering, visualization) completes in under 10 minutes on benchmark datasets

## Limitations

- Only tested on Windows 11; distributed/cluster mode is not yet optimized because each node has separate memory allocation
- Wide-to-tall data melting is a workaround for Spark's RAM limitation and needs further optimization
- No changelog; repository is not actively maintained, and a new version is under development
- No benchmarking against Seurat and Scanpy, and no GPU-accelerated configuration is supported in the current version

## Evidence
