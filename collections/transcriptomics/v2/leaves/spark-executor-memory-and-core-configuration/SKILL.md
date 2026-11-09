---
name: spark-executor-memory-and-core-configuration
description: Use when benchmarking or executing scSPARKL (or another Spark-based pipeline)
  on scRNA-seq count data in .csv/.parquet form and you must configure executor cores
  and driver/executor memory for the available hardware first.
when_to_use_negative:
- The dataset fits comfortably in RAM and an in-memory framework like Seurat or Scanpy
  is adequate — the authors note these popular pipelines are constrained by RAM but
  remain standard for smaller data
- Input is not .csv or .parquet (the pipeline currently supports only these formats,
  with more formats under development)
- The goal requires optimized Spark cluster-mode operation with per-node memory allocation,
  which the paper defers to future development
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: Apache Spark
  role: Distributed in-memory execution engine whose executor core counts and driver/executor
    memory allocations are the tuning targets of this skill
- name: scSPARKL
  role: The scRNA-seq pipeline whose runtime performance is benchmarked under each
    Spark configuration
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Parquet
  role: Efficient columnar storage format accepted as scSPARKL input for benchmarked
    runs
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:4455eb8ec31b28b1c01160e6dbac304e1b697b82c15aab00869c92f9497bddb8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/asif7adil/scSparkl
  tools:
  - 10x Genomics
  - Apache Parquet
  - Apache Spark
  - Python
  - scSPARKL
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# spark-executor-memory-and-core-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tuning Apache Spark executor cores and driver/executor memory allocations before running the scSPARKL scRNA-seq pipeline, since Spark's in-memory performance depends directly on this configuration and it determines standalone-vs-distributed runtime behavior.

## When to use

Use when benchmarking or executing scSPARKL (or another Spark-based pipeline) on scRNA-seq count data in .csv/.parquet form and you must configure executor cores and driver/executor memory for the available hardware first.

## When NOT to use

- The dataset fits comfortably in RAM and an in-memory framework like Seurat or Scanpy is adequate — the authors note these popular pipelines are constrained by RAM but remain standard for smaller data
- Input is not .csv or .parquet (the pipeline currently supports only these formats, with more formats under development)
- The goal requires optimized Spark cluster-mode operation with per-node memory allocation, which the paper defers to future development

## Inputs

- scRNA-seq count matrix as .csv or .parquet (the only input formats currently supported by the pipeline)
- Random cell subsamples of increasing size (e.g., 2,000–10,000 cells drawn from the 68K PBMC dataset, Zheng et al. 2017)
- Target hardware specification (cores, RAM) for Spark standalone or multi-node cluster setup

## Outputs

- A tuned Spark configuration (executor cores, driver and executor memory allocations)
- Per-subsample execution times assembled into execution_times.csv
- Pipeline artifacts: cell/gene quality summaries in the auto-generated analyses folder, filtered/normalized wide and tall data, and PCA output

## How to apply

Because Spark is a distributed in-memory analytics engine, first determine the number of cores and tune memory allocations for executors and drivers to match available hardware (scSPARKL was tested with Apache Spark 3.0+ and Python 3.9+ on Windows 11, with the spark-notes executor/driver configuration tutorial as the tuning reference). Choose standalone mode for smaller dataloads and distributed mode for larger ones. Run the full scSPARKL pipeline — data melting, cell/gene quality summaries, data_filter(), Global/CPM or Quantile normalization, MAD or coefficient-of-variance gene selection, and the spark-native PCA — on each input subsample under the fixed configuration, recording total execution time (e.g., assembling timings into execution_times.csv for 2K–10K cell subsamples of the 68K PBMC dataset). Validate by confirming every subsample size has a timing entry and that runtimes fall in the paper's scale: ~10 min on a two-node cluster and ~13 min single-node for the 10,000-cell PBMC subsample versus >50 min sequential.

## Related tools

- **Apache Spark** (Distributed in-memory execution engine whose executor core counts and driver/executor memory allocations are the tuning targets of this skill)
- **scSPARKL** (The scRNA-seq pipeline whose runtime performance is benchmarked under each Spark configuration) — https://www.github.com/asif7adil/scSPARKL
- **Apache Parquet** (Efficient columnar storage format accepted as scSPARKL input for benchmarked runs)

## Evaluation signals

- execution_times.csv contains a timing entry for every subsample size (2K–10K cells), each from a run that completed the full pipeline under the chosen Spark configuration
- 10,000-cell PBMC runtime lands near the paper's scale: ~10 min (two-node Spark) or ~13 min (single-node Spark), versus >50 min sequential — roughly 80% and 74% improvements
- Quality summaries for cells and genes are auto-written to the analyses folder after data melting, confirming the configured cluster executed the parallel routines
- Total computation time on a PBMC dataset is on the order of ~8 minutes as the reference magnitude
- Spark launches without memory errors across runs, indicating executor/driver memory allocations were sized correctly for the dataload

## Limitations

- Cluster mode operations are not yet optimized — each node has separate memory allocation and optimization is deferred to future development
- The repository is not being actively maintained; a new version is under development
- Tested only on Microsoft Windows 11; other OS configurations are unverified
- Data melting is a workaround for RAM limitations and the authors state further optimization is needed
- No GPU-accelerated configuration benchmarks exist; GPU acceleration is not utilized in the current version

## Evidence
