---
name: scrnaseq-count-matrix-melting
description: Use when a wide-format scRNA-seq count matrix (.csv) with one column
  per gene must be reshaped into a tall cell-gene-value table before Parquet serialization
  and downstream scSPARKL analysis on low-memory hardware.
when_to_use_negative:
- Input is already a tall/melted cell-gene-value table or already stored as .parquet
  — the pipeline accepts .csv and .parquet inputs and melting wide data again would
  be redundant
- Working entirely within an in-memory tool like Seurat or Scanpy where the dataset
  comfortably fits in RAM — melting is specific to the Spark-based scSPARKL pipeline
- Input files are formats other than .csv/.parquet (e.g. .h5 or 10x matrix files),
  which the current scSPARKL implementation does not ingest
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scSPARKL
  role: Provides the melt/reshape routine that cleans and melts the wide count matrix
    to tall format as the first pipeline task
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Spark
  role: Distributed in-memory engine on which the wide dataframe is loaded and melted
    without loading the full matrix into driver RAM
- name: Apache Parquet
  role: Columnar storage format in which the melted tall dataframe is written for
    efficient downstream Spark reads
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:388d387403b3809f1943c8eeeebbab253d6f96451c0d9f821d00c470a3275b79
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# scrnaseq-count-matrix-melting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A data-reshaping skill that melts a wide-format scRNA-seq count matrix (.csv, one column per gene) into a tall cell-gene-value dataframe inside Apache Spark via scSPARKL, so the data can be stored as Parquet and analyzed on commodity hardware with as little as 4 GB of RAM.

## When to use

Use when a wide-format scRNA-seq count matrix (.csv) with one column per gene must be reshaped into a tall cell-gene-value table before Parquet serialization and downstream scSPARKL analysis on low-memory hardware.

## When NOT to use

- Input is already a tall/melted cell-gene-value table or already stored as .parquet — the pipeline accepts .csv and .parquet inputs and melting wide data again would be redundant
- Working entirely within an in-memory tool like Seurat or Scanpy where the dataset comfortably fits in RAM — melting is specific to the Spark-based scSPARKL pipeline
- Input files are formats other than .csv/.parquet (e.g. .h5 or 10x matrix files), which the current scSPARKL implementation does not ingest

## Inputs

- Raw wide-format scRNA-seq count matrix as .csv (rows = cells, columns = genes)

## Outputs

- Tall (long) Spark dataframe with one row per cell-gene observation (cell identifier, gene identifier, count value)
- Compressed Apache Parquet file containing the melted tall dataframe
- Row count equal to cells x genes from the input matrix

## How to apply

Load the raw wide-format count matrix (.csv) as an Apache Spark dataframe within the scSPARKL environment, then sanitize column names (replacing periods, commas, and spaces with underscores) so the schema is Spark-compatible. Apply the melt operation (melt_Spark()) to convert the wide table into tall format, yielding one row per cell-gene observation with cell identifier, gene identifier, and count value columns. Write the melted tall dataframe to disk as a compressed Apache Parquet file, which becomes the input for the later scSPARKL steps (quality summaries, data_filter(), normalization, PCA). The rationale is that the melt/reshape step reduces the columnar load, enabling execution on machines with as little as 4 GB of memory, and that Parquet combines Spark's compute power with efficient columnar storage. Note that in this implementation data melting is a mitigation for RAM limitations rather than a fully optimized solution, so expect further optimization in later framework versions.

## Related tools

- **scSPARKL** (Provides the melt/reshape routine that cleans and melts the wide count matrix to tall format as the first pipeline task) — https://www.github.com/asif7adil/scSPARKL
- **Apache Spark** (Distributed in-memory engine on which the wide dataframe is loaded and melted without loading the full matrix into driver RAM)
- **Apache Parquet** (Columnar storage format in which the melted tall dataframe is written for efficient downstream Spark reads)

## Examples

```
melted = melt_Spark(spark.read.csv('raw_counts.csv', header=True, inferSchema=True)); melted.write.parquet('counts_tall.parquet')
```

## Evaluation signals

- The Parquet output file exists and parses correctly as Parquet
- Row count of the melted dataframe equals (number of cells) x (number of genes) of the input wide matrix
- Output contains exactly the expected columns: cell identifier, gene identifier, and count value
- Dataframe column names contain no periods, commas, or spaces (all replaced with underscores) so the schema loads cleanly in Spark
- Downstream steps (quality summaries, data_filter(), normalization) can read the tall Parquet without schema errors

## Limitations

- The data melting workaround for RAM limitation needs further optimization according to the authors' discussion
- Current pipeline only accepts .csv and .parquet as input file formats; other formats are not yet supported
- Repository is not actively maintained; a new version is under development, and Spark cluster mode operations are not yet optimized (single-node/standalone focus)

## Evidence
