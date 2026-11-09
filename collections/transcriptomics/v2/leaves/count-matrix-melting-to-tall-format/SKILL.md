---
name: count-matrix-melting-to-tall-format
description: Use when the input is a wide gene-by-cell count matrix (CSV or Parquet)
  that must be reshaped to tall format before running the scSPARKL pipeline stages
  of QC summaries, filtering, and normalization.
when_to_use_negative:
- Input is already in tall/long format from a prior scSPARKL melt — re-melting is
  unnecessary and wasteful.
- Input is in formats other than .csv or .parquet (e.g. .h5ad or 10x mtx), which the
  current scSPARKL implementation does not accept.
- The analysis target is single-cell multi-omics data (CITE-seq, scATAC-seq, scBS-seq)
  — those Spark-based pipelines are not yet developed.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scSPARKL
  role: Provides the data melting routine that cleans and reshapes the wide count
    matrix to tall format as the first pipeline stage.
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Spark
  role: Distributed execution engine on which the melting/reshaping runs; requires
    tuned executor core and memory configuration.
- name: Apache Parquet
  role: Efficient columnar input/output format supported by the melting stage.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:13279a51cb0996417790b326f40a014c8b4f3e6244b9833bab99bd5a3e837e41
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
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# count-matrix-melting-to-tall-format

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Clean and melt a wide scRNA-seq count matrix (.csv or .parquet) into tall (long) format using scSPARKL on Apache Spark, producing the tall-formatted representation required by all subsequent scSPARKL stages (quality summaries, data_filter(), normalization).

## When to use

Use when the input is a wide gene-by-cell count matrix (CSV or Parquet) that must be reshaped to tall format before running the scSPARKL pipeline stages of QC summaries, filtering, and normalization.

## When NOT to use

- Input is already in tall/long format from a prior scSPARKL melt — re-melting is unnecessary and wasteful.
- Input is in formats other than .csv or .parquet (e.g. .h5ad or 10x mtx), which the current scSPARKL implementation does not accept.
- The analysis target is single-cell multi-omics data (CITE-seq, scATAC-seq, scBS-seq) — those Spark-based pipelines are not yet developed.

## Inputs

- Raw scRNA-seq count matrix in wide gene-by-cell layout as .csv
- Raw scRNA-seq count matrix in wide gene-by-cell layout as .parquet

## Outputs

- Tall (long) formatted count data where each row is a cell-gene observation
- Auto-generated analyses folder for subsequent cell and gene quality summaries

## How to apply

Load the raw count matrix into the Apache Spark environment (standalone or distributed depending on data size), accepting `.csv` or `.parquet` inputs. As the first pipeline stage, clean the matrix and melt it from wide gene-by-cell layout into tall (long) format, where each row is a cell-gene observation; scSPARKL's downstream routines (cell/gene quality summaries, data_filter(), and normalization) are all written against this tall representation. Tune Spark executor cores and driver/executor memory before melting, since the melt is an in-memory operation that mitigates RAM limitations through data reshaping. Normalization later consumes the tall data and returns both a wide and a tall normalized output, so keep the tall schema consistent with column names needed for QC summaries (e.g. percent_ercc, percent_mito, genes expressed per cell).

## Related tools

- **scSPARKL** (Provides the data melting routine that cleans and reshapes the wide count matrix to tall format as the first pipeline stage.) — https://www.github.com/asif7adil/scSPARKL
- **Apache Spark** (Distributed execution engine on which the melting/reshaping runs; requires tuned executor core and memory configuration.)
- **Apache Parquet** (Efficient columnar input/output format supported by the melting stage.)

## Examples

```
spark-submit scSPARKL.py --task melt --input raw_counts.csv  # or run the Jupyter notebook shipped with the repo
```

## Evaluation signals

- Output data is in tall (long) format with one row per cell-gene observation rather than the original wide gene-by-cell layout.
- The analyses folder is automatically generated after melting, ready to receive cell and gene quality summaries.
- The tall schema retains the columns needed by data_filter() (e.g. percent_ercc, percent_mito, gene counts per cell) so the subsequent filtering stage runs without schema errors.
- Downstream normalization accepts the tall data and returns both wide and tall normalized outputs, confirming the melt preserved cell-gene count pairs.
- Melting completes within the configured Spark executor/driver memory allocation without spilling or failure.

## Limitations

- Data melting is a workaround for Spark's RAM limitation and the README/paper note that further optimization is needed.
- Only `.csv` and `.parquet` input formats are currently supported; more formats are under development.
- Tested on Microsoft Windows 11; Spark cluster mode operations are not yet optimized since each node has separate memory allocation.

## Evidence

- [readme] Data Melting: "Input data is first cleaned and melted to tall format."
- [discussion] data melting rationale: "This challenge is currently mitigated through data melting techniques, but further optimization is needed"
