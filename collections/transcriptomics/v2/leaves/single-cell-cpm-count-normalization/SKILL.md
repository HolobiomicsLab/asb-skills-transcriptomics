---
name: single-cell-cpm-count-normalization
description: Use when raw scRNA-seq count data (.csv or .parquet) has already been
  quality-filtered in a scSPARKL/Spark pipeline and cell counts must be made comparable
  across cells prior to gene selection, PCA, and clustering.
when_to_use_negative:
- Input is not raw counts but already-normalized or transformed expression data (CPM-like
  or quantile normalization would be applied twice).
- Input files are in formats other than .csv or .parquet — the current scSPARKL pipeline
  does not support them.
- The dataset is small enough for in-memory tools such as Seurat or Scanpy, which
  are the standard alternatives and avoid Spark setup overhead.
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scSPARKL
  role: Framework providing the two normalization methods (Global/Simple CPM-like
    and Quantile Normalization) executed on tall-formatted filtered data
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Spark
  role: Distributed in-memory execution engine on which the parallel normalization
    routines run
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:e873d7360978f4a3e56e6c8755dd36b7068307894da39222a31cc8908635106d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/asif7adil/scSparkl
  tools:
  - Apache Spark
  - scSPARKL
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-cell-cpm-count-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Normalize filtered single-cell RNA-seq counts using scSPARKL's Spark-based normalization stage, which offers a CPM-like Global/Simple normalization or Quantile Normalization on tall-formatted data. This step makes expression values comparable across cells before HVG selection, PCA, and clustering.

## When to use

Use when raw scRNA-seq count data (.csv or .parquet) has already been quality-filtered in a scSPARKL/Spark pipeline and cell counts must be made comparable across cells prior to gene selection, PCA, and clustering.

## When NOT to use

- Input is not raw counts but already-normalized or transformed expression data (CPM-like or quantile normalization would be applied twice).
- Input files are in formats other than .csv or .parquet — the current scSPARKL pipeline does not support them.
- The dataset is small enough for in-memory tools such as Seurat or Scanpy, which are the standard alternatives and avoid Spark setup overhead.

## Inputs

- Tall (long) formatted filtered count data from the scSPARKL melting and data_filter() stages
- Raw count matrix in .csv or .parquet format loaded into Apache Spark (upstream)

## Outputs

- Wide-formatted normalized expression matrix
- Tall-formatted normalized expression data

## How to apply

First ensure the melted tall-format data has passed through data_filter() using cell/gene quality summaries (e.g. cells expressing < 10 genes, genes expressed in < 3 cells, dropout rate > 70%). Choose between the two supported normalization types: Global or Simple normalization, which is similar to CPM normalization (counts scaled per cell total), or Quantile Normalization. The Spark-based normalization takes the tall-formatted filtered data and returns both one wide-formatted and one tall-formatted normalized output, so downstream stages (MAD-based HVG selection with default k > 3, Spark-native PCA, clustering) can consume either layout. Default parameters of the normalization/filtering packages can be changed by directly manipulating the filter package in the scSPARKL source code.

## Related tools

- **scSPARKL** (Framework providing the two normalization methods (Global/Simple CPM-like and Quantile Normalization) executed on tall-formatted filtered data) — https://www.github.com/asif7adil/scSPARKL
- **Apache Spark** (Distributed in-memory execution engine on which the parallel normalization routines run)

## Evaluation signals

- Both a wide-formatted and a tall-formatted normalized output are produced from a single tall input, as documented.
- Normalized values follow a CPM-like scale (counts scaled per-cell) when Global/Simple normalization is selected, or quantile-matched distributions when Quantile Normalization is selected.
- Downstream scSPARKL stages (MAD HVG selection, Spark-native PCA, clustering) complete and cluster assignments reproduce the expected Jurkat–293T species separation with ARI ≈ 0.90.
- The full pipeline including normalization, dimension reduction, clustering, and visualization finishes in less than 10 minutes on the benchmark datasets.

## Limitations

- The repository is not actively maintained; a new version is under development.
- Current pipeline only accepts .csv and .parquet input formats; more formats are planned but not yet available.
- Data melting is used as a workaround for Spark RAM limitations and needs further optimization; cluster mode operations are not yet optimized and GPU acceleration is not utilized.
- No benchmarking of scSPARKL against Seurat or Scanpy was performed.

## Evidence
