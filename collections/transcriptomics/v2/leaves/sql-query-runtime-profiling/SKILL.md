---
name: sql-query-runtime-profiling
description: Use when comparing query/filter runtime of SQL-based single-cell query
  engines (AnnSQL/DuckDB) versus AnnData backed or in-memory operations across dataset
  sizes, to find crossover points and speedup ratios.
when_to_use_negative:
- On-disk comparison for libraries smaller than 5,000 cells — on-disk AnnData outperformed
  on-disk AnnSQL below this crossover
- In-memory comparison for libraries below 45,000 cells — in-memory AnnData outperformed
  in-memory AnnSQL there, and both run <0.01 s under 75,000 cells
- Databases with more than 30,000 genes — a known unresolved DuckDB memory leak (TODO
  in the paper's code) makes this configuration unreliable
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: AnnSQL
  role: Provides the SQL query interface (asql.query) and filter methods whose runtimes
    are profiled; builds in-memory or on-disk DuckDB-backed databases
  repo: github.com/ArpiarSaundersLab/annsql
- name: AnnData
  role: 'Baseline framework: in-memory and backed-mode boolean-mask subsetting timed
    against AnnSQL queries'
- name: DuckDB
  role: In-process engine executing the SQL filters underlying AnnSQL; runtime subject
    of the benchmark
  repo: https://duckdb.org/
- name: Pandas
  role: Aggregates recorded per-trial runtimes and writes comparison CSVs (df.to_csv,
    index=False)
- name: memory_profiler
  role: Profiles memory alongside runtime during the benchmark scripts
- name: scanpy
  role: Loads h5ad datasets (e.g. sc.read_h5ad for the 4.4M-cell atlas in backed mode)
    and supplies simulated dense matrices
provenance:
  source_task_ids:
  - task_007
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:13335c680626a2fbd2c96977762ffbe90e5b66247daea6eec9071d25e1f4fd37
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AnnData
  - AnnSQL
  - DuckDB
  - Pandas
  - Python
  - matplotlib
  - memory_profiler
  - scanpy
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# sql-query-runtime-profiling

## Summary

Benchmarks SQL query runtimes of AnnSQL (in-process DuckDB engine) against AnnData in backed and in-memory modes across synthetic single-cell datasets from 1,000 to 250,000 cells with 10,000 genes, quantifying speedup ratios and identifying runtime crossover points.

## When to use

Use when comparing query/filter runtime of SQL-based single-cell query engines (AnnSQL/DuckDB) versus AnnData backed or in-memory operations across dataset sizes, to find crossover points and speedup ratios.

## When NOT to use

- On-disk comparison for libraries smaller than 5,000 cells — on-disk AnnData outperformed on-disk AnnSQL below this crossover
- In-memory comparison for libraries below 45,000 cells — in-memory AnnData outperformed in-memory AnnSQL there, and both run <0.01 s under 75,000 cells
- Databases with more than 30,000 genes — a known unresolved DuckDB memory leak (TODO in the paper's code) makes this configuration unreliable

## Inputs

- Simulated dense AnnData count matrices (1,000–250,000 cells x 10,000 genes)
- On-disk AnnSQL databases (.asql files) built with MakeDb
- Large-scale h5ad single-nucleus datasets in backed mode (e.g. Macosko Mouse Atlas Single Nuclei, 4.4 million cells)
- In-memory AnnData objects converted via AnnSQL(adata=adata)

## Outputs

- CSV runtime tables comparing AnnSQL vs AnnData backed/in-memory per dataset size (e.g. runtimes_table.csv, gene_filter_runtime_comparison.csv)
- Aggregated runtime comparison plot (runtime_comparison.png)
- Speedup ratios (AnnSQL vs backed-mode ~746x on atlas; ~400x at 250,000 cells; ~42x vs HPC in-memory AnnData)
- Bonferroni-corrected paired t-test results per dataset size

## How to apply

Simulate dense AnnData count matrices at paper-defined sizes (dataset_sizes = [1000, 5000, 10000, 15000, 20000, 25000, 30000, 35000, 40000, 45000, 50000, 75000, 100000, 250000] cells x 10,000 genes). For on-disk comparison, build AnnSQL databases with MakeDb (create_all_indexes=False, convenience_view=False) and open them with AnnSQL(db=...); for in-memory comparison instantiate AnnSQL(adata=adata). Time equivalent operations — SQL filters like SELECT gene_1 FROM X WHERE gene_1 > 0.5, filter_by_cell_counts(min_cell_count=2500, max_cell_count=40000), and the AnnData backed-mode boolean-mask equivalent (e.g. adata_mem[adata_mem[:, gene].X > 0, gene] for the ENSMUSG00000070880 atlas filter) — repeating queries for stable measurements. Aggregate per-size timings with pandas (df.to_csv, index=False) and plot with matplotlib. Assess per-size differences with paired t-tests using Bonferroni-corrected alpha_corrected = 0.05 / num_comparisons, and compute speedup ratios (expected ~400x at 250,000 cells on-disk; ~746x AnnSQL vs backed-mode AnnData on the 4.4M-cell Macosko Mouse Atlas; ~42x vs HPC in-memory AnnData where a cluster is available).

## Related tools

- **AnnSQL** (Provides the SQL query interface (asql.query) and filter methods whose runtimes are profiled; builds in-memory or on-disk DuckDB-backed databases) — github.com/ArpiarSaundersLab/annsql
- **AnnData** (Baseline framework: in-memory and backed-mode boolean-mask subsetting timed against AnnSQL queries)
- **DuckDB** (In-process engine executing the SQL filters underlying AnnSQL; runtime subject of the benchmark) — https://duckdb.org/
- **Pandas** (Aggregates recorded per-trial runtimes and writes comparison CSVs (df.to_csv, index=False))
- **memory_profiler** (Profiles memory alongside runtime during the benchmark scripts)
- **scanpy** (Loads h5ad datasets (e.g. sc.read_h5ad for the 4.4M-cell atlas in backed mode) and supplies simulated dense matrices)

## Examples

```
MakeDb(adata, db_path="../db/random/", db_name="random_data_"+str(i), create_all_indexes=False, convenience_view=False)
```

## Evaluation signals

- On-disk crossover reproduced below 5,000 cells (AnnData backed faster), matching the reported ~400x AnnSQL advantage at 250,000 cells (0.14 s vs 57 s)
- In-memory crossover reproduced at ~45,000 cells, with both methods under 0.01 s for libraries below 75,000 cells
- Atlas-scale (4.4M cells) laptop speedup ratio lands in the hundreds-fold range (~746x) favoring AnnSQL over backed-mode AnnData
- Per-size paired t-tests use alpha_corrected = 0.05 / num_comparisons (Bonferroni correction)
- Output CSV exists with explicit runtime columns for both AnnSQL and AnnData backed mode; HPC comparison flagged not-applicable when no cluster is available

## Limitations

- Known unresolved issue: DuckDB memory leak on databases with more than 30k genes (TODO in the paper's benchmark code)
- No software version numbers are reported for the benchmarked tools in the results section
- The AnnSQL database structure is not optimized for performance; tables closely mirror the AnnData object structure
- No numeric runtime values appear in the section code itself — results are computed at runtime and written to CSVs, so reproduction depends on hardware

## Evidence

- [results] On-disk AnnData outperformed on-disk AnnSQL only for libraries <5000 cells, while queries of 250 000 cells were on average ∼400 times faster using AnnSQL (0.14 versus 57 seconds).: "On-disk AnnData outperformed on-disk AnnSQL only for libraries <5000 cells, while queries of 250 000 cells were on average ∼400 times faster using AnnSQL (0.14 versus 57 seconds)."
- [results] dataset_sizes = [1000, 5000, 10000, 15000, 20000, 25000, 30000, 35000, 40000, 45000, 50000, 75000, 100000, 250000]: "dataset_sizes = [1000, 5000, 10000, 15000, 20000, 25000, 30000, 35000, 40000, 45000, 50000, 75000, 100000, 250000]"
- [methods] Running AnnSQL locally for datasets that are larger than memory, that would typically require AnnData in backed mode see substantial runtime improvements: "Running AnnSQL locally for datasets that are larger than memory, that would typically require AnnData in backed mode see substantial runtime improvements"
- [abstract] AnnSQL-based operations were executed in minutes on a laptop for which equivalent operations in AnnData or Seurat largely failed (or were ∼700× slower): "AnnSQL-based operations were executed in minutes on a laptop for which equivalent operations in AnnData or Seurat largely failed (or were ∼700× slower)"
- [readme] The Python based AnnSQL package enables SQL-based queries on [AnnData](https://anndata.readthedocs.io/en/latest/) objects, returning results as either a [Pandas](https://pandas.pydata.org/) DataFrame: "The Python based AnnSQL package enables SQL-based queries on [AnnData](https://anndata.readthedocs.io/en/latest/) objects, returning results as either a [Pandas](https://pandas.pydata.org/) DataFrame"
