---
name: distributed-pipeline-execution-timing
description: Use when measuring how runtime of a distributed single-cell analysis
  pipeline scales with dataset size (e.g. 2K–10K cell subsamples) under different
  cluster/memory configurations versus a sequential implementation.
when_to_use_negative:
- The goal is benchmarking against Seurat or Scanpy — the paper explicitly states
  no such benchmarking was performed, so this skill cannot support cross-framework
  runtime claims
- GPU-accelerated configurations are involved — the current scSPARKL version does
  not utilize GPU acceleration and no GPU benchmarks exist
- The workload requires optimized Spark cluster mode with per-node memory allocation
  — the paper notes cluster mode operations are not yet optimized and deferred to
  future development
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: scSPARKL
  role: Distributed scRNA-seq pipeline whose execution time is measured across subsample
    sizes and Spark configurations
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Spark
  role: Distributed in-memory execution engine whose configuration (cores, executor/driver
    memory, node count) is varied during benchmarking
- name: Apache Parquet
  role: Columnar storage format for efficient input of benchmark subsample files
- name: Python
  role: Sequential standalone baseline implementation against which scSPARKL timings
    are compared
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:c391b33c6b39c52468c59b2cb4c02dd46c737f3121c1c149565e577567fbb778
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
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# distributed-pipeline-execution-timing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks the execution time of a distributed scRNA-seq pipeline (scSPARKL on Apache Spark) against dataset size and Spark configuration, comparing it to a sequential baseline. Used to quantify speedup from parallelizing single-cell analysis workflows.

## When to use

Use when measuring how runtime of a distributed single-cell analysis pipeline scales with dataset size (e.g. 2K–10K cell subsamples) under different cluster/memory configurations versus a sequential implementation.

## When NOT to use

- The goal is benchmarking against Seurat or Scanpy — the paper explicitly states no such benchmarking was performed, so this skill cannot support cross-framework runtime claims
- GPU-accelerated configurations are involved — the current scSPARKL version does not utilize GPU acceleration and no GPU benchmarks exist
- The workload requires optimized Spark cluster mode with per-node memory allocation — the paper notes cluster mode operations are not yet optimized and deferred to future development

## Inputs

- 68K PBMC scRNA-seq dataset (Zheng et al. 2017, 10x Genomics), subsampled to 2,000–10,000 cells
- Subsample files in .csv or .parquet format
- Apache Spark environment configuration (executor cores, driver/executor memory, single-node or two-node cluster)

## Outputs

- execution_times.csv with a timing entry for every subsample size and Spark configuration
- Per-configuration runtime comparison (e.g. scSPARKL vs sequential baseline) and percent runtime improvement

## How to apply

Obtain the 68K PBMC scRNA-seq dataset (Zheng et al. 2017, 10x Genomics) and draw random cell subsamples from 2,000 to 10,000 cells, written as .csv or .parquet inputs, since scSPARKL currently supports those formats. Configure Apache Spark (3.0+) by tuning executor cores and driver/executor memory allocations per the recommended Spark configuration guidance. For each subsample, run the full scSPARKL pipeline — data melting to tall format, cell/gene quality summaries, data_filter(), normalization (Global/CPM-style or Quantile), and Spark-based PCA — recording total execution time per size and per Spark setup (e.g. two-node vs single-node cluster). Compare timings against a sequential standalone implementation to compute relative runtime improvement, and assemble per-subsample timings into execution_times.csv. Rationale: in-memory tools like Seurat and Scanpy are RAM-constrained, so benchmarking under varying Spark configurations demonstrates the scalability gains of a Spark-native, Parquet-backed framework.

## Related tools

- **scSPARKL** (Distributed scRNA-seq pipeline whose execution time is measured across subsample sizes and Spark configurations) — https://www.github.com/asif7adil/scSPARKL
- **Apache Spark** (Distributed in-memory execution engine whose configuration (cores, executor/driver memory, node count) is varied during benchmarking)
- **Apache Parquet** (Columnar storage format for efficient input of benchmark subsample files)
- **Python** (Sequential standalone baseline implementation against which scSPARKL timings are compared)

## Evaluation signals

- execution_times.csv contains a timing entry for every subsample size in the 2K–10K range
- Each run completed the full pipeline (melting, quality summaries, data_filter(), normalization, PCA) under the chosen Spark configuration
- Runtimes fall in the order of magnitude reported by the paper (total computation time as low as ~8 min on a PBMC dataset)
- Speedup versus the sequential baseline is substantial: ~10 min (two-node) and ~13 min (single-node) versus >50 min sequential on the 10,000-cell subsample, i.e. ~80% and ~74% runtime improvement
- Timing scales monotonically with subsample size across 2K to 10K cells

## Limitations

- The scSPARKL repository is not actively maintained; a new version is under development
- Data melting is a workaround for a RAM limitation and needs further optimization
- Pipeline currently accepts only .csv and .parquet input formats
- Spark cluster mode operations (per-node memory allocation) are not yet optimized; results reported are for two-node and single-node setups on the tested hardware
- Tested on Microsoft Windows 11; other OS environments are not covered by the current implementation

## Evidence

- [results] two-node vs single-node speedup: [evidence span withheld — anchor status: not_found]
