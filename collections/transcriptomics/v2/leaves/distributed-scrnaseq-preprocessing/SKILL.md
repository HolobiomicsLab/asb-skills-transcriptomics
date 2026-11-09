---
name: distributed-scrnaseq-preprocessing
description: Use when raw scRNA-seq counts arrive as a wide-format .csv too large
  for in-memory tools (Seurat/Scanpy constrained by RAM) and downstream Spark-based
  QC, filtering, normalization, PCA or clustering is planned.
when_to_use_negative:
- Input is already in tall format or an existing .parquet artifact from a prior scSPARKL
  melt — re-melting would duplicate work
- Downstream step (normalization, gene selection, PCA) that consumes the melted Parquet,
  not a raw wide .csv
- Dataset is small enough for in-memory tools like Seurat or Scanpy and no Spark environment
  is needed
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scSPARKL
  role: provides the data cleaning and melt_Spark() routine that reshapes the wide
    count matrix to tall format within Spark
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Spark
  role: execution environment (standalone or distributed) that loads the wide .csv
    as a DataFrame and performs the melt out-of-core
- name: Apache Parquet
  role: columnar storage format used to persist the melted tall dataframe as a compressed
    artifact
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:64a3c9c7feddd98b2c8d01b8ef1f9f0d479a14298cbb334f57d2d70633ba3fe3
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# distributed-scrnaseq-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stage-1 preprocessing of scSPARKL: load a wide-format scRNA-seq count matrix (.csv) into Apache Spark, sanitize the schema, melt it to tall (one row per cell-gene observation), and persist it as a compressed Apache Parquet file for all subsequent distributed analysis steps.

## When to use

Use when raw scRNA-seq counts arrive as a wide-format .csv too large for in-memory tools (Seurat/Scanpy constrained by RAM) and downstream Spark-based QC, filtering, normalization, PCA or clustering is planned.

## When NOT to use

- Input is already in tall format or an existing .parquet artifact from a prior scSPARKL melt — re-melting would duplicate work
- Downstream step (normalization, gene selection, PCA) that consumes the melted Parquet, not a raw wide .csv
- Dataset is small enough for in-memory tools like Seurat or Scanpy and no Spark environment is needed

## Inputs

- wide-format scRNA-seq count matrix as .csv (cells x genes)

## Outputs

- melted tall-format DataFrame (cell identifier, gene identifier, count value per row)
- compressed Apache Parquet file of the tall-format data on disk

## How to apply

Load the raw wide-format count matrix (.csv) as a Spark DataFrame inside the scSPARKL environment (Spark standalone or distributed depending on data load). Sanitize column names by replacing periods, commas, and spaces with underscores so the schema is Spark-compatible. Apply scSPARKL's melt_Spark() to reshape the wide matrix to tall format — one row per cell-gene observation with cell identifier, gene identifier, and count columns — which reduces the columnar load and makes the pipeline feasible on machines with as little as 4 GB RAM. Write the melted tall DataFrame to disk as a compressed Apache Parquet artifact; this Parquet file is the canonical interchange format consumed by later stages (quality summaries, data_filter(), normalization, MLlib PCA) and by the Python-side UMAP/t-SNE steps.

## Related tools

- **scSPARKL** (provides the data cleaning and melt_Spark() routine that reshapes the wide count matrix to tall format within Spark) — https://www.github.com/asif7adil/scSPARKL
- **Apache Spark** (execution environment (standalone or distributed) that loads the wide .csv as a DataFrame and performs the melt out-of-core)
- **Apache Parquet** (columnar storage format used to persist the melted tall dataframe as a compressed artifact)

## Examples

```
tall_df = melt_Spark(spark.read.csv('counts.csv', header=True, inferSchema=True)); tall_df.write.parquet('melted.parquet')
```

## Evaluation signals

- Parquet output file exists and parses correctly as Parquet
- Row count of the tall Parquet equals cells x genes of the input wide matrix
- Schema contains exactly the expected cell identifier, gene identifier, and count columns
- Column names contain no periods, commas, or spaces (all replaced with underscores) so the schema is Spark-compatible
- Generated Parquet is readable by subsequent scSPARKL stages (quality summaries, data_filter(), normalization, PCA)

## Limitations

- Only .csv and .parquet input formats are currently supported by the pipeline; other formats are not yet accepted
- The data melting workaround for RAM limitation still needs further optimization per the paper's discussion
- Repository is not being actively maintained (a new version is under development), and the current implementation has been tested only on Windows 11

## Evidence
