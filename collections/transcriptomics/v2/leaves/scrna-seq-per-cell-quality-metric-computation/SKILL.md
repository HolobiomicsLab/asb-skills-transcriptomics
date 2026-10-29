---
name: scrna-seq-per-cell-quality-metric-computation
description: Use when tall-format (melted) scRNA-seq expression data has been loaded
  as a Spark DataFrame and per-cell/per-gene quality summaries are needed before applying
  data_filter() thresholds.
when_to_use_negative:
- Input has not yet been melted to tall format — scSPARKL requires data melting as
  the preceding step
- The goal is already-filtered data; this skill only derives quality summaries — filtering
  itself is delegated to data_filter()
- You need a quality-control implementation in Seurat, Scanpy, or scater rather than
  a Spark-native one — scSPARKL targets in-memory-limitation scenarios where those
  pipelines are constrained by RAM
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scSPARKL
  role: Framework whose Stage-2 quality control module generates the cell and gene
    quality summaries and passes them to data_filter()
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Spark
  role: Distributed compute engine on which the groupBy/aggregate QC computations
    over the tall-format DataFrame are executed
- name: Apache Parquet
  role: Storage format for the two QC summary files written to the analyses folder
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:2653f027dbeb7a4b4eab3f082787b569e7d863ca71fece4870354ec841091356
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# scrna-seq-per-cell-quality-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute Spark-distributed per-cell (Θ1) and per-gene (Θ2) quality summary matrices from melted tall-format scRNA-seq data in scSPARKL, producing the QC metrics (e.g., percent ERCC, percent mitochondrial, dropout rate) that downstream cell/gene filtering thresholds depend on.

## When to use

Use when tall-format (melted) scRNA-seq expression data has been loaded as a Spark DataFrame and per-cell/per-gene quality summaries are needed before applying data_filter() thresholds.

## When NOT to use

- Input has not yet been melted to tall format — scSPARKL requires data melting as the preceding step
- The goal is already-filtered data; this skill only derives quality summaries — filtering itself is delegated to data_filter()
- You need a quality-control implementation in Seurat, Scanpy, or scater rather than a Spark-native one — scSPARKL targets in-memory-limitation scenarios where those pipelines are constrained by RAM

## Inputs

- Cleaned, melted tall-format scRNA-seq expression matrix as CSV or Parquet, loaded as an Apache Spark distributed DataFrame

## Outputs

- Θ1 per-cell quality summary DataFrame (total expression, number of genes expressed, average expression, percent ERCC, percent mitochondrial) written as Parquet into the analyses folder
- Θ2 per-gene quality summary DataFrame (cells expressing gene > 0, total expression, dropout count, dropout percentage, mean expression) written as Parquet into the analyses folder
- log1p-transformed expression-count columns in both summary tables

## How to apply

Load the cleaned, melted tall-format expression data (CSV or Parquet) as a Spark distributed DataFrame in scSPARKL. Group by cell and aggregate the Θ1 per-cell QC metrics — total expression, number of genes expressed, average expression, percentage of ERCC reads, and percentage of mitochondrial reads — following scater/SCANPY/Seurat conventions. Group by gene and aggregate the Θ2 per-gene QC metrics — number of cells expressing the gene, total expression, dropout count, dropout percentage, and mean expression. Apply log1p transformation to the relevant expression-count columns of both summary tables. Write the two quality DataFrames as separate Parquet files into the automatically generated analyses folder so that data_filter() can consume them; its default thresholds include percent_ercc > 10%, percent_mito > 5%, cells expressing < 10 genes, genes expressed in < 3 cells, and dropout rate > 70%.

## Related tools

- **scSPARKL** (Framework whose Stage-2 quality control module generates the cell and gene quality summaries and passes them to data_filter()) — https://www.github.com/asif7adil/scSPARKL
- **Apache Spark** (Distributed compute engine on which the groupBy/aggregate QC computations over the tall-format DataFrame are executed)
- **Apache Parquet** (Storage format for the two QC summary files written to the analyses folder)

## Evaluation signals

- Both Parquet QC files exist in the automatically generated analyses folder after the step completes
- Θ1 file contains percent_ercc and percent_mito columns, the metrics consumed by downstream thresholds (percent_ercc > 10%, percent_mito > 5%)
- Θ2 file contains cells-expressing-gene, dropout count/dropout percentage, total and mean expression columns consumed by data_filter()
- Relevant expression-count columns in both summaries are log1p-transformed
- Summary counts agree with the melted input: number of genes expressed > 0 per cell and number of cells expressing each gene > 0 match the tall-format data

## Limitations

- scSPARKL cluster mode operations are not yet optimized; the framework was tested on standalone Spark (Microsoft Windows 11), where memory tuning of executors and drivers is critical
- Currently only .csv and .parquet inputs are supported by the pipeline
- The data melting workaround for large-dataset RAM limitations needs further optimization
- The repository is not being actively maintained; a new version is under development

## Evidence
