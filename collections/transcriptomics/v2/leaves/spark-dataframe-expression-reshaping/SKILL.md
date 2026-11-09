---
name: spark-dataframe-expression-reshaping
description: Use when scRNA-seq count matrices (.csv or .parquet) are too large for
  in-memory tools like Seurat or Scanpy, and downstream Spark-based filtering, normalization,
  or PCA requires melted tall-format or reshaped wide-format DataFrames.
when_to_use_negative:
- Input data is already in the target layout (e.g., already melted tall data when
  a wide matrix is requested without transformation).
- The dataset is small enough for in-memory tools such as Seurat or Scanpy, where
  a Spark melt adds unnecessary overhead.
- The input format is not .csv or .parquet, since the pipeline currently only supports
  those input file types.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scSPARKL
  role: Modular Spark-based framework providing the data melting and reshape operations
    on single-cell count data
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Spark
  role: Distributed in-memory execution engine used to melt and reshape expression
    DataFrames in parallel
- name: Apache Parquet
  role: Columnar storage format used to persist the reshaped wide and tall normalized
    outputs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:7576b7ccde5db6c5d7f5de989a765191027b8a35e1a747cd050bc12a137a4be0
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# spark-dataframe-expression-reshaping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reshape scRNA-seq count data between tall (melted) and wide (cells x genes) formats inside Apache Spark using Spark DataFrames, as implemented in the scSPARKL pipeline so that downstream filtering, normalization, and PCA can run distributed rather than in memory.

## When to use

Use when scRNA-seq count matrices (.csv or .parquet) are too large for in-memory tools like Seurat or Scanpy, and downstream Spark-based filtering, normalization, or PCA requires melted tall-format or reshaped wide-format DataFrames.

## When NOT to use

- Input data is already in the target layout (e.g., already melted tall data when a wide matrix is requested without transformation).
- The dataset is small enough for in-memory tools such as Seurat or Scanpy, where a Spark melt adds unnecessary overhead.
- The input format is not .csv or .parquet, since the pipeline currently only supports those input file types.

## Inputs

- Raw scRNA-seq count matrix as .csv file
- Raw scRNA-seq count matrix as .parquet file
- Spark session with tuned executor/driver memory and core allocations

## Outputs

- Melted tall-format Spark DataFrame (cell, gene, expression)
- Wide-format (cells x genes) normalized matrix, e.g. normalized_wide.parquet
- Tall-format normalized matrix, e.g. normalized_tall.parquet
- Cell and gene quality summaries written to an analyses folder

## How to apply

Load the raw count matrix (.csv or .parquet are the currently supported inputs) into a Spark DataFrame, clean it, and melt it to tall format (cell, gene, count) as the first 'Data Melting' step of the scSPARKL pipeline. Run quality-summary generation and `data_filter()` on the tall data, then perform normalization (Global/CPM-like with log1p, or Quantile Normalization), which 'takes tall formated data and returns one wide formatted and a tall formated normalized data'. Write reshaped results with the Apache Parquet writer (e.g., normalized_wide.parquet and normalized_tall.parquet). The rationale is that Spark's distributed, in-memory engine plus Parquet storage lets the reshape run on hundreds of thousands of cells on commodity hardware, where Seurat and Scanpy are constrained by available RAM. Tune Spark executor cores and driver/executor memory allocations before melting, since reshaping large matrices is memory-intensive and the README warns to configure executors and drivers carefully.

## Related tools

- **scSPARKL** (Modular Spark-based framework providing the data melting and reshape operations on single-cell count data) — https://www.github.com/asif7adil/scSPARKL
- **Apache Spark** (Distributed in-memory execution engine used to melt and reshape expression DataFrames in parallel)
- **Apache Parquet** (Columnar storage format used to persist the reshaped wide and tall normalized outputs)

## Examples

```
df = spark.read.parquet('counts.parquet'); tall_df = df.selectExpr("stack(n_genes, " + ",".join([f"'{g}', {g}" for g in genes]) + ") as (gene, count)"); tall_df.write.parquet('tall_counts.parquet')
```

## Evaluation signals

- Both normalized_wide.parquet and normalized_tall.parquet exist and load correctly as Apache Parquet files.
- The wide and tall outputs describe the same values: the tall DataFrame melted from the wide matrix reproduces the original counts cell-by-cell and gene-by-gene.
- Global-normalized values match CPM-style scaling (library size scaled to 10,000 with log1p pseudocount 1).
- Quality summaries were produced in the automatically generated analyses folder before data_filter() was invoked.
- The full reshaping pipeline on large datasets (e.g., PBMC n = 10000) completes in minutes (~8 min reported), indicating Spark parallelism is effective.

## Limitations

- The pipeline currently accepts only .csv and .parquet inputs; more formats are still being added.
- Data melting is a workaround for Spark RAM limitations and 'further optimization is needed' per the discussion.
- Cluster mode operations are not yet optimized; each node has separate memory allocation and optimization is deferred to future development.
- The repository is not actively maintained; a new version is under development.

## Evidence
