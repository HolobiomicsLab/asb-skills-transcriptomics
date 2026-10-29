---
name: parquet-io-for-single-cell-data
description: Use when a scRNA-seq pipeline must load or persist tall/wide count matrices
  (e.g. filtered tall csv/parquet input, normalized_wide.parquet/normalized_tall.parquet
  outputs) on Spark, especially when in-memory tools like Seurat/Scanpy hit RAM limits.
when_to_use_negative:
- Input data is in formats other than .csv/.parquet — the README states 'currently
  the pipeline uses .csv and .parquet as input files' and other formats are not yet
  supported
- Dataset is small enough for in-memory frameworks such as Seurat or Scanpy, where
  Spark setup overhead outweighs scalability gains
- Operations require optimized Spark cluster mode — the discussion notes cluster mode
  operations are not yet optimized and deferred to future development
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scSPARKL
  role: Modular Spark-based framework whose pipeline performs the Parquet reading/writing
    of tall and wide scRNA-seq matrices
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Spark
  role: Distributed in-memory engine whose DataFrame API and Parquet writer load filtered
    inputs and persist normalized outputs
- name: Apache Parquet
  role: Columnar storage format used to store wide and tall normalized single-cell
    matrices efficiently
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:ab595e99e0cfcdbf91d3669395dbc62e578446f673758f9d08db08e6c993e1e4
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
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# parquet-io-for-single-cell-data

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reading and writing single-cell scRNA-seq count matrices as Apache Parquet on Apache Spark within the scSPARKL framework, which pairs Spark's distributed compute with Parquet's efficient columnar storage to enable scalable analysis of hundreds of thousands of cells on commodity hardware.

## When to use

Use when a scRNA-seq pipeline must load or persist tall/wide count matrices (e.g. filtered tall csv/parquet input, normalized_wide.parquet/normalized_tall.parquet outputs) on Spark, especially when in-memory tools like Seurat/Scanpy hit RAM limits.

## When NOT to use

- Input data is in formats other than .csv/.parquet — the README states 'currently the pipeline uses .csv and .parquet as input files' and other formats are not yet supported
- Dataset is small enough for in-memory frameworks such as Seurat or Scanpy, where Spark setup overhead outweighs scalability gains
- Operations require optimized Spark cluster mode — the discussion notes cluster mode operations are not yet optimized and deferred to future development

## Inputs

- Filtered tall-format scRNA-seq count matrix as .csv or .parquet (Spark DataFrame)
- Wide-format count matrix to be melted to tall format
- Normalized wide/tall DataFrames to be persisted

## Outputs

- normalized_wide.parquet (cells x genes matrix in Apache Parquet)
- normalized_tall.parquet (tall-format matrix in Apache Parquet)
- Quality summary tables in an auto-generated analyses folder

## How to apply

Load the filtered tall-format count matrix (.csv or .parquet — currently the only input formats scSPARKL supports) into a Spark DataFrame. Perform pipeline operations (reshaping, filtering, normalization) as distributed DataFrame transformations rather than collecting to driver memory, since scSPARKL is designed to be memory-efficient on commodity hardware. Persist each result — e.g. the normalization module returns both a wide (cells x genes) and a tall formatted output — with the Spark DataFrame Parquet writer to files such as normalized_wide.parquet and normalized_tall.parquet. Tune executor cores and memory allocations for the Spark environment, as the README explicitly recommends for this in-memory engine. The rationale is that Parquet's efficient storage plus Spark's distributed processing lets the full scSPARKL pipeline (QC through visualization) complete large reanalyses in under 10 minutes.

## Related tools

- **scSPARKL** (Modular Spark-based framework whose pipeline performs the Parquet reading/writing of tall and wide scRNA-seq matrices) — https://www.github.com/asif7adil/scSPARKL
- **Apache Spark** (Distributed in-memory engine whose DataFrame API and Parquet writer load filtered inputs and persist normalized outputs)
- **Apache Parquet** (Columnar storage format used to store wide and tall normalized single-cell matrices efficiently)

## Examples

```
df = spark.read.parquet("filtered_tall.parquet"); df.write.parquet("normalized_tall.parquet")
```

## Evaluation signals

- Both output files (normalized_wide.parquet, normalized_tall.parquet) exist and load correctly as Apache Parquet
- Wide and tall outputs are mutually consistent reshapes of the same normalized values (cells x genes vs melted tall format)
- Global-normalized values match CPM scaling to a 10,000-count target followed by log1p with pseudocount 1
- Quantile-normalized cells share identical empirical distributions (sorted value vectors match across cells)
- No driver-memory overflow: all load/write steps execute as distributed Spark DataFrame operations on commodity hardware

## Limitations

- Only .csv and .parquet input formats are currently supported; more formats are in progress
- The repository is not actively maintained; a new version is under development
- Cluster mode operations are not yet optimized — each node's separate memory allocation handling is deferred to future development
- Data melting to tall format for large matrices mitigates RAM limitations but 'further optimization is needed'

## Evidence
