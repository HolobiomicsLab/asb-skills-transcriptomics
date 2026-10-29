---
name: spark-dataframe-column-sanitization
description: Use when loading a raw wide-format scRNA-seq count matrix (.csv) into
  Apache Spark and column headers contain periods, commas, or spaces that would break
  Spark schema compatibility.
when_to_use_negative:
- Input is already a tall-format or Parquet file that has previously passed through
  scSPARKL Stage-1 preprocessing (pipeline accepts .csv and .parquet inputs post-preprocessing)
- Column headers are already Spark-compatible (no periods, commas, or spaces) and
  would be left unchanged by the replacement
- The analysis pipeline is an in-memory tool such as Seurat or Scanpy that does not
  rely on Spark schema resolution
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scSPARKL
  role: Framework whose Stage-1 preprocessing performs the column sanitization, melt,
    and Parquet write
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Spark
  role: Distributed analytics engine in which the DataFrame is loaded and its column
    names are sanitized
- name: Apache Parquet
  role: Storage format that the sanitized schema feeds into after the wide-to-tall
    melt
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:9b6c2ee98accff400cda6120a941cfa7f8a7d07fc903c398d36e9ed60f7b4700
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# spark-dataframe-column-sanitization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Sanitizes wide-format scRNA-seq count matrix column names (replacing periods, commas, and spaces with underscores) so the Spark DataFrame schema is Spark-compatible before Stage-1 preprocessing in scSPARKL. Clean column names are a prerequisite for the wide-to-tall melt and Parquet write that make scRNA-seq data efficient on commodity hardware.

## When to use

Use when loading a raw wide-format scRNA-seq count matrix (.csv) into Apache Spark and column headers contain periods, commas, or spaces that would break Spark schema compatibility.

## When NOT to use

- Input is already a tall-format or Parquet file that has previously passed through scSPARKL Stage-1 preprocessing (pipeline accepts .csv and .parquet inputs post-preprocessing)
- Column headers are already Spark-compatible (no periods, commas, or spaces) and would be left unchanged by the replacement
- The analysis pipeline is an in-memory tool such as Seurat or Scanpy that does not rely on Spark schema resolution

## Inputs

- Raw wide-format scRNA-seq count matrix as .csv (cells x genes, e.g. 10x Genomics output)
- Apache Spark DataFrame with unsanitized column headers containing periods, commas, or spaces

## Outputs

- Apache Spark DataFrame with Spark-compatible column names (periods, commas, and spaces replaced by underscores)
- Schema-validated DataFrame ready for melt_Spark() and Parquet serialization

## How to apply

Load the raw wide-format scRNA-seq count matrix (.csv) as an Apache Spark DataFrame within the scSPARKL environment. Iterate over the DataFrame's column identifiers and replace periods, commas, and spaces with underscores so the schema is Spark-compatible; typically cell barcodes (e.g. from 10x Genomics data) arrive with such characters in headers. Perform this sanitization before calling melt_Spark(), because the melt to tall format (one row per cell-gene observation with cell identifier, gene identifier, and count value) and the downstream compressed Apache Parquet write assume a clean schema. The rationale is that the melt/reshape step reduces columnar load and increases efficiency, enabling analysis on low-memory machines (as little as 4 GB), which only works if Spark can parse and persist the schema. Validate that every sanitized header matches the pattern of underscores-only names and that no original punctuation characters remain before proceeding.

## Related tools

- **scSPARKL** (Framework whose Stage-1 preprocessing performs the column sanitization, melt, and Parquet write) — https://www.github.com/asif7adil/scSPARKL
- **Apache Spark** (Distributed analytics engine in which the DataFrame is loaded and its column names are sanitized)
- **Apache Parquet** (Storage format that the sanitized schema feeds into after the wide-to-tall melt)

## Examples

```
df = spark.read.csv('raw_counts.csv', header=True, inferSchema=True); df = df.withColumnsRenamed({c: c.replace('.', '_').replace(',', '_').replace(' ', '_') for c in df.columns})
```

## Evaluation signals

- No column header in the sanitized DataFrame contains a period, comma, or space character
- The set of sanitized column names is in one-to-one correspondence with the original .csv headers (no renames collide after punctuation replacement)
- After the subsequent melt, the tall Parquet file's row count equals cells x genes from the input matrix
- The melted output contains exactly the expected cell identifier, gene identifier, and count value columns
- The Parquet output exists and parses correctly as Apache Parquet

## Limitations

- The repository is not being actively maintained; a new version is under development
- The pipeline currently only supports .csv and .parquet as input formats, so other formats require conversion before sanitization
- Data melting to circumvent RAM limitations still needs further optimization, per the article's discussion
- Tested on Microsoft Windows 11 with Apache Spark 3.0+ and Python 3.9+; other environments are untested

## Evidence

- [other] Sanitize the dataframe column names by replacing periods, commas, and spaces with underscores so the schema is Spark-compatible.: [evidence span withheld — anchor status: not_found]
- [other] Melt the wide dataframe to tall format using melt_Spark(), yielding one row per cell-gene observation with cell identifier, gene identifier, and count value columns.: [evidence span withheld — anchor status: not_found]
