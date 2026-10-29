---
name: parquet-columnar-storage-write
description: Use when a wide-format scRNA-seq count matrix (.csv) has been melted
  to tall format in Spark and needs a columnar, compressed on-disk representation
  for subsequent scSPARKL pipeline stages.
when_to_use_negative:
- The data are already stored as .parquet and simply need to be loaded for downstream
  analysis — Stage-1 reshaping is unnecessary
- The workflow requires input formats other than .csv/.parquet — the README notes
  the pipeline currently only accepts these two formats
- A wide (cell x gene) normalized matrix must be retained for tools that expect wide
  format — melting is irreversible reshaping, and only the normalization step returns
  both wide and tall outputs
edam_operation: http://edamontology.org/operation_1812
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scSPARKL
  role: Provides the Stage-1 preprocessing pipeline including the melt_Spark() reshape
    and the Parquet write of the tall dataframe
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Spark
  role: Distributed in-memory engine that loads the .csv count matrix, holds the melted
    dataframe, and writes it out as Parquet
- name: Apache Parquet
  role: Columnar storage format used to persist the melted tall dataframe in compressed
    form for efficient later reads
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:df3153068bb29b34640e19c62fd2534ccb01afa140e84bbe7e655ef8f12edfa2
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
  edam_operation: http://edamontology.org/operation_1812
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# parquet-columnar-storage-write

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Writes large scRNA-seq dataframes (e.g., melted tall cell-gene-count tables from scSPARKL Stage-1) as compressed Apache Parquet files so downstream Spark stages avoid the memory limits of in-memory tools like Seurat or Scanpy.

## When to use

Use when a wide-format scRNA-seq count matrix (.csv) has been melted to tall format in Spark and needs a columnar, compressed on-disk representation for subsequent scSPARKL pipeline stages.

## When NOT to use

- The data are already stored as .parquet and simply need to be loaded for downstream analysis — Stage-1 reshaping is unnecessary
- The workflow requires input formats other than .csv/.parquet — the README notes the pipeline currently only accepts these two formats
- A wide (cell x gene) normalized matrix must be retained for tools that expect wide format — melting is irreversible reshaping, and only the normalization step returns both wide and tall outputs

## Inputs

- Wide-format scRNA-seq count matrix as .csv
- Melted tall Spark dataframe with cell identifier, gene identifier, and count value columns

## Outputs

- Compressed Apache Parquet file of the tall (melted) dataframe
- On-disk columnar store readable as input (.parquet) by later scSPARKL stages

## How to apply

Within the scSPARKL environment, first load the raw wide-format count matrix (.csv) as a Spark dataframe and sanitize column names by replacing periods, commas, and spaces with underscores so the schema is Spark-compatible. Melt the wide dataframe to tall format with melt_Spark(), yielding one row per cell-gene observation (cell identifier, gene identifier, count value). Write the melted dataframe to disk as a compressed Apache Parquet file; this columnar layout is what lets the pipeline run on commodity hardware with as little as 4 GB RAM. Pair Spark's compute with Parquet's efficient storage, since the paper presents this combination as the first attempt tailored for downstream single-cell analysis.

## Related tools

- **scSPARKL** (Provides the Stage-1 preprocessing pipeline including the melt_Spark() reshape and the Parquet write of the tall dataframe) — https://www.github.com/asif7adil/scSPARKL
- **Apache Spark** (Distributed in-memory engine that loads the .csv count matrix, holds the melted dataframe, and writes it out as Parquet)
- **Apache Parquet** (Columnar storage format used to persist the melted tall dataframe in compressed form for efficient later reads)

## Examples

```
df = spark.read.csv('counts.csv', header=True, inferSchema=True); tall = melt_Spark(df); tall.write.parquet('melted_counts.parquet')
```

## Evaluation signals

- The Parquet output file exists at the expected path and parses correctly as Parquet
- Row count of the Parquet file equals cells x genes from the input wide matrix
- The Parquet schema contains the expected cell identifier, gene identifier, and count value columns
- Column names contain no periods, commas, or spaces (all replaced with underscores), confirming a Spark-compatible schema
- The file can be reloaded as a Spark dataframe by subsequent pipeline stages (README: pipeline accepts .csv and .parquet inputs)

## Limitations

- Data melting is a workaround for RAM limitations and the paper states 'further optimization is needed'
- The repository is not actively maintained; a new version is under development (README notice)
- Only .csv and .parquet input formats are currently supported; more formats are planned but not available
- Standalone/local mode is the tested configuration; Spark cluster mode operations are not yet optimized and deferred to future development

## Evidence

- [other] the melt/reshape step reduces the columnar load and increases efficiency: [evidence span withheld — anchor status: not_found]
