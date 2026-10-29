---
name: benchmark-result-tabulation
description: Use when timing a pipeline (e.g. scSPARKL on Spark) across multiple dataset
  sizes or executor/driver memory configurations and you need a per-subsample runtime
  table comparable to a sequential implementation.
when_to_use_negative:
- The goal is biological output quality (clustering accuracy, ARI) rather than runtime
  measurement — use the biological benchmarking workflow instead
- Only a single dataset/configuration is run with no baseline comparison, so a timing
  table adds no comparative value
- Benchmarking GPU-accelerated or cluster-mode operations, which the paper explicitly
  does not cover
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: scSPARKL
  role: Pipeline being benchmarked; each run's total execution time (melting through
    PCA) is the recorded measurement
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Spark
  role: Execution environment whose configurations (executor cores, driver/executor
    memory) define benchmark conditions
- name: Apache Parquet
  role: Storage format for the benchmark input subsamples
- name: 10x Genomics
  role: Source of the public 68K PBMC dataset subsampled for the benchmark
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:7a9cfe4b7e09cf42a9cb3b3e92ffd6014c9b532d8461f666b2aade382ca62302
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# benchmark-result-tabulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assembles per-run execution timings from benchmarking scSPARKL across scRNA-seq dataset sizes and Spark configurations into a structured table (execution_times.csv) so runtime scaling against a sequential baseline can be quantified and validated.

## When to use

Use when timing a pipeline (e.g. scSPARKL on Spark) across multiple dataset sizes or executor/driver memory configurations and you need a per-subsample runtime table comparable to a sequential implementation.

## When NOT to use

- The goal is biological output quality (clustering accuracy, ARI) rather than runtime measurement — use the biological benchmarking workflow instead
- Only a single dataset/configuration is run with no baseline comparison, so a timing table adds no comparative value
- Benchmarking GPU-accelerated or cluster-mode operations, which the paper explicitly does not cover

## Inputs

- Random cell subsamples (2K-10K cells) of the 68K PBMC scRNA-seq dataset as .csv or .parquet files
- Per-run total execution times from scSPARKL runs under each Spark configuration
- Timing from the sequential standalone Python implementation (baseline)

## Outputs

- execution_times.csv with a timing entry for every subsample size and Spark configuration
- Tabulated runtime improvements / speedups versus the sequential baseline

## How to apply

Run the full pipeline (data melting, cell/gene quality summaries, data_filter(), normalization, PCA) once per condition — e.g. random subsamples of the 68K PBMC dataset (Zheng et al. 2017) from 2,000 to 10,000 cells, written as .csv or .parquet — under each fixed Spark configuration (executor cores and driver/executor memory tuned per hardware). Record the total execution time for each subsample-size/configuration pair and write each entry to execution_times.csv, ensuring one timing per condition. Validate completeness by checking the table contains an entry for every subsample size and that each run completed all pipeline stages; compare against the paper's reference scale (~10 min on a two-node Spark cluster and ~13 min single-node for 10K cells vs. over 50 min sequential, i.e. ~80% and ~74% improvements) and the reported minimum of ~8 min total computation. Finally, tabulate the speedup percentages relative to the sequential standalone implementation to expose how runtime scales with cell count and Spark configuration.

## Related tools

- **scSPARKL** (Pipeline being benchmarked; each run's total execution time (melting through PCA) is the recorded measurement) — https://www.github.com/asif7adil/scSPARKL
- **Apache Spark** (Execution environment whose configurations (executor cores, driver/executor memory) define benchmark conditions)
- **Apache Parquet** (Storage format for the benchmark input subsamples)
- **10x Genomics** (Source of the public 68K PBMC dataset subsampled for the benchmark)

## Examples

```
python scSPARKL.py --input pbmc_10k.parquet --pipeline melt,qc,data_filter,normalize,pca; echo "10000,613,$SECONDS >> execution_times.csv"
```

## Evaluation signals

- execution_times.csv contains exactly one timing entry for every subsample size in the 2K-10K range
- Each recorded run completed the full pipeline (data melting, quality summaries, data_filter(), normalization, PCA) under the chosen Spark configuration
- 10K-cell Spark runtimes fall in the paper's reported order of magnitude (~10 min two-node, ~13 min single-node; total computation as low as ~8 min)
- Computed speedups versus the sequential baseline are near the reported ~80% (two-node) and ~74% (single-node) improvements
- Runtime increases monotonically with subsample size within each Spark configuration

## Limitations

- Runtimes are only comparable across subsamples run under the same fixed Spark configuration; changing executor cores or memory mid-benchmark invalidates comparisons
- No benchmarking against Seurat and Scanpy was performed, so speedups are relative to a sequential standalone implementation only
- Cluster-mode Spark operations were not yet optimized, and no GPU-accelerated benchmarks exist; single-node Spark performance is also configuration-dependent (README notes tuning executor cores and memory allocations is required)

## Evidence

- [results] On the largest 10,000-cell PBMC subsample, scSPARKL completed processing in ~10 min on a two-node Spark cluster and ~13 min on single-node Spark, versus over 50 min for the sequential standalone implementation (runtime improvements of ~80% and ~74%).: [evidence span withheld — anchor status: not_found]
- [results] Cells were randomly sampled and chunked from 68 K-PBMC data: "Cells were randomly sampled and chunked from 68 K-PBMC data"
- [results] with total computation time as low as ~ 8 min: "with total computation time as low as ~ 8 min, demonstrating both the efficiency and reliability of our approach"
- [readme] Apache spark is distributed in-memory analytics engine. It is highly recommended to efficiently determine the number of cores and tune the memory alocations to the executors and drivers.: "It is highly recommended to efficiently determine the number of cores and tune the memory alocations to the executors and drivers"
- [methods] The quality summaries are then passed as arguments to the `data_filter()` to filter out the unwanted genes and cells.: "The quality summaries are then passed as arguments to the `data_filter()` to filter out the unwanted genes and cells"
