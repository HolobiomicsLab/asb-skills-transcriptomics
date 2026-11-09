---
name: cross-platform-runtime-comparison
description: Use when reproducing or extending runtime/memory comparisons of a tool
  across computational environments (laptop vs HPC) or across competing implementations
  (e.g. AnnSQL vs AnnData backed mode vs Seurat) on the same dataset.
when_to_use_negative:
- The dataset fits comfortably in memory and only one implementation is available,
  so no cross-method or cross-machine contrast is possible
- The goal is correctness or biological validation of results rather than runtime/memory
  benchmarking
- The operation is a complex machine-learning step (e.g. clustering) where the compared
  tools implement genuinely different algorithms, making runtimes non-comparable
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: AnnSQL
  role: 'Subject of the benchmark: runs SQL queries on an AnnData-inspired DuckDB
    database, in-memory or on-disk, with extended preprocessing methods (calculate_total_counts,
    expression_normalize, expression_log)'
  repo: https://github.com/ArpiarSaundersLab/annsql
- name: AnnData
  role: 'Baseline implementation: same operations run in backed mode (for datasets
    larger than memory) and in-memory on HPC for comparison'
- name: DuckDB
  role: In-process engine behind AnnSQL enabling fast on-disk SQL execution; databases
    can also be opened in R via the duckdb driver
  repo: https://github.com/ArpiarSaundersLab/annsql
- name: Seurat
  role: R-side baseline whose equivalent operations (e.g. object building) largely
    failed with memory errors or were ~700x slower on HPC
- name: Pandas
  role: Collects timing results into DataFrames exported as summary CSVs
  repo: https://pandas.pydata.org/
- name: Python
  role: Driver language (conda env with Python 3.12, pip install annsql) for benchmark
    scripts using scanpy and memory_profiler
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:f70b24b356a2e6c19f57714f57a4db05e50a388cc2d3d31f31f7fe96411b26fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ArpiarSaundersLab/annsql
  tools:
  - AnnData
  - AnnSQL
  - DuckDB
  - Pandas
  - Python
  - Seurat
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cross-platform-runtime-comparison

## Summary

Benchmark runtime and memory of the same single-cell operations (e.g. SQL expression filtering, count normalization) across AnnSQL/DuckDB, AnnData (backed and in-memory), and Seurat on laptop-class vs HPC machines, to quantify orders-of-magnitude speedups achievable with AnnSQL.

## When to use

Use when reproducing or extending runtime/memory comparisons of a tool across computational environments (laptop vs HPC) or across competing implementations (e.g. AnnSQL vs AnnData backed mode vs Seurat) on the same dataset.

## When NOT to use

- The dataset fits comfortably in memory and only one implementation is available, so no cross-method or cross-machine contrast is possible
- The goal is correctness or biological validation of results rather than runtime/memory benchmarking
- The operation is a complex machine-learning step (e.g. clustering) where the compared tools implement genuinely different algorithms, making runtimes non-comparable

## Inputs

- AnnData object or .h5ad file (e.g. Macosko Mouse Atlas Single Nuclei, 4.4 million cells, loaded in backed mode)
- On-disk AnnSQL database (.asql file) built with MakeDb and asql.save_raw()
- SQL query strings for the equivalent operation (e.g. gene expression > 0 filters)
- Laptop-class machine and optionally a high-performance computing (HPC) environment

## Outputs

- Timing/memory table as a Pandas DataFrame with per-method, per-environment runtimes
- Summary CSV (e.g. gene_filter_runtime_comparison.csv, atlas_profile_hpc.csv)
- Speedup ratios (AnnSQL vs AnnData backed mode ~746x; AnnSQL vs HPC in-memory AnnData ~42x)

## How to apply

Profile paired implementations of identical operations on identical inputs: build an on-disk AnnSQL database from the AnnData h5ad with MakeDb (saving the raw layer via asql.save_raw()), then time operations such as 'SELECT ... WHERE ENSMUSG00000070880 > 0' via asql.query() or chunked extended methods (asql.calculate_total_counts(chunk_size=750)), repeating queries for stable measurements. Run the equivalent AnnData backed-mode boolean-mask subsetting (adata_mem[adata_mem[:, gene].X > 0, gene]) and, where available, in-memory AnnData on an HPC as reference baselines; record expected targets of ~746x laptop speedup for filtering and ~700x HPC slowdowns/failures for AnnData/Seurat. Time each operation with runtime/memory profiling (e.g. memory_profiler's memory_usage), aggregate timings into a Pandas DataFrame, and export a summary CSV. Judge correctness by checking that measured ratios are within the reported ranges (e.g. AnnSQL preprocessing runtimes ~4.28-67 minutes on the 4.4 million cell Macosko Mouse Atlas on a laptop) and flag unavailable comparisons (e.g. no HPC cluster) as not-applicable rather than interpolating.

## Related tools

- **AnnSQL** (Subject of the benchmark: runs SQL queries on an AnnData-inspired DuckDB database, in-memory or on-disk, with extended preprocessing methods (calculate_total_counts, expression_normalize, expression_log)) — https://github.com/ArpiarSaundersLab/annsql
- **AnnData** (Baseline implementation: same operations run in backed mode (for datasets larger than memory) and in-memory on HPC for comparison)
- **DuckDB** (In-process engine behind AnnSQL enabling fast on-disk SQL execution; databases can also be opened in R via the duckdb driver) — https://github.com/ArpiarSaundersLab/annsql
- **Seurat** (R-side baseline whose equivalent operations (e.g. object building) largely failed with memory errors or were ~700x slower on HPC)
- **Pandas** (Collects timing results into DataFrames exported as summary CSVs) — https://pandas.pydata.org/
- **Python** (Driver language (conda env with Python 3.12, pip install annsql) for benchmark scripts using scanpy and memory_profiler)

## Examples

```
from AnnSQL import AnnSQL; import time; asql = AnnSQL(db="db/atlas.asql"); t0=time.time(); asql.query("SELECT cell_id FROM X WHERE ENSMUSG00000070880 > 0"); print(time.time()-t0)
```

## Evaluation signals

- Summary CSV exists with runtime columns for both AnnSQL and AnnData backed mode for the same filter (e.g. ENSMUSG00000070880 > 0)
- Reproduced laptop AnnSQL-vs-backed-mode speedup is orders of magnitude (~hundreds-fold; paper target ~746x)
- AnnSQL extended-procedure runtimes fall within the reported ~4.28-67 minute laptop range on the 4.4 million cell atlas
- AnnData/Seurat equivalents are recorded as failed (memory errors) or substantially slower (~700x on HPC), matching the paper's finding
- HPC-only baselines are explicitly flagged not-applicable if no cluster is available, rather than silently omitted

## Limitations

- AnnData analysis failed on both machines and Seurat object building failed due to memory errors, so some baselines cannot be measured at all
- DuckDB has a known unresolved memory leak on databases with more than 30k genes (noted as a TODO in the code)
- The AnnSQL database structure mirrors the AnnData object rather than being optimized for performance; no software version numbers or numeric runtime values are reported in the article code, so exact reproduction depends on the environment

## Evidence

- [abstract] laptop speedup for gene filtering: "AnnSQL-based operations were executed in minutes on a laptop for which equivalent operations in AnnData or Seurat largely failed (or were ∼700× slower)"
- [methods] backed-mode comparison scope: "Running AnnSQL locally for datasets that are larger than memory, that would typically require AnnData in backed mode see substantial runtime improvements"
- [results] equivalent AnnData subsetting to time: "adata_mem[adata_mem[:, "ENSMUSG00000070880"].X > 0, "ENSMUSG00000070880"]"
- [readme] how the database is constructed: "AnnSQL converts the layers of an AnnData object into a relational [DuckDB](https://duckdb.org/) database. Each layer is stored as an individual table"
- [results] chunked extended procedure used in benchmarks: "#calculate total counts
asql.calculate_total_counts(chunk_size=750,print_progress=True)"
