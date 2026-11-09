---
name: wide-to-tall-dataframe-reshaping
description: Use when a wide-format scRNA-seq count matrix (cells x genes, .csv) must
  be converted into a Spark-optimized tall (cell, gene, count) format before downstream
  filtering or normalization.
when_to_use_negative:
- Input is already a tall cell-gene-count dataframe or an existing .parquet file produced
  by a prior melt
- Downstream step is not Spark-based (e.g., a Seurat or Scanpy in-memory pipeline),
  where wide in-memory matrices are expected
- Dataset fits comfortably in RAM and does not need Spark/Parquet optimization
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scSPARKL
  role: Provides the pipeline environment and melt_Spark() routine that sanitizes,
    melts, and writes the count matrix
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Spark
  role: Distributed in-memory engine on which the wide dataframe is loaded and melted
- name: Apache Parquet
  role: Columnar storage format used to persist the melted tall dataframe as a compressed
    file
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:893c5d2c738a074db6e45c046642d725284b5ebe3a45d150426a8d0b75eccf57
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# wide-to-tall-dataframe-reshaping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stage-1 preprocessing skill from scSPARKL that reshapes a wide-format scRNA-seq count matrix (.csv) into a tall cell-gene-count Spark dataframe via melt_Spark() and persists it as compressed Apache Parquet, reducing the columnar load so analysis can run on commodity hardware with as little as 4 GB RAM.

## When to use

Use when a wide-format scRNA-seq count matrix (cells x genes, .csv) must be converted into a Spark-optimized tall (cell, gene, count) format before downstream filtering or normalization.

## When NOT to use

- Input is already a tall cell-gene-count dataframe or an existing .parquet file produced by a prior melt
- Downstream step is not Spark-based (e.g., a Seurat or Scanpy in-memory pipeline), where wide in-memory matrices are expected
- Dataset fits comfortably in RAM and does not need Spark/Parquet optimization

## Inputs

- Raw wide-format scRNA-seq count matrix (.csv) with one column per gene
- Apache Spark dataframe (post column-name sanitization)

## Outputs

- Tall melted Spark dataframe with cell identifier, gene identifier, and count value columns
- Compressed Apache Parquet file with rows equal to cells x genes

## How to apply

Load the raw wide-format count matrix (.csv) as an Apache Spark dataframe inside the scSPARKL environment, since the pipeline currently accepts only .csv and .parquet inputs. Sanitize column names by replacing periods, commas, and spaces with underscores so the schema is Spark-compatible. Apply melt_Spark() to transform the wide dataframe to tall format, yielding one row per cell-gene observation with cell identifier, gene identifier, and count value columns. Write the melted tall dataframe to disk as a compressed Apache Parquet file; the melt reduces the columnar load and mitigates RAM limitations on low-memory machines. Validate that the Parquet file exists, parses as Parquet, and its row count equals cells x genes from the input matrix.

## Related tools

- **scSPARKL** (Provides the pipeline environment and melt_Spark() routine that sanitizes, melts, and writes the count matrix) — https://www.github.com/asif7adil/scSPARKL
- **Apache Spark** (Distributed in-memory engine on which the wide dataframe is loaded and melted)
- **Apache Parquet** (Columnar storage format used to persist the melted tall dataframe as a compressed file)

## Examples

```
melted_df = melt_Spark(spark_df); melted_df.write.parquet("tall_counts.parquet")
```

## Evaluation signals

- Parquet output file exists and parses correctly as Parquet
- Row count of the melted dataframe equals cells x genes from the input wide matrix
- Output schema contains exactly the expected cell identifier, gene identifier, and count value columns
- Column names contain no periods, commas, or spaces (all replaced with underscores)
- Tall dataframe loads and supports downstream quality-summary generation and data_filter() calls without columnar-load errors

## Limitations

- The data-melting workaround for RAM limitation still needs further optimization per the paper's discussion
- Pipeline currently accepts only .csv and .parquet input formats; more formats are not yet supported
- Repository is not actively maintained; a new version is under development
- Cluster-mode Spark operations are not yet optimized, so melting is primarily intended for standalone/low-memory setups

## Evidence
