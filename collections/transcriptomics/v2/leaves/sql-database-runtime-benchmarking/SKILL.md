---
name: sql-database-runtime-benchmarking
description: Use when comparing runtime or memory performance of a SQL-based single-cell
  database engine (AnnSQL/DuckDB) versus AnnData across filter/query operations, including
  re-testing on sparser simulated datasets.
when_to_use_negative:
- When the analysis goal is biological interpretation rather than runtime/memory profiling
  of query engines
- When datasets fit easily in memory and a one-off query is needed — the benchmark
  skill targets systematic multi-size comparisons, not single ad-hoc queries
- When databases contain more than 30k genes — a known unresolved DuckDB memory leak
  affects this regime
edam_operation: http://edamontology.org/operation_3840
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0203
tools:
- name: AnnSQL
  role: Python SQL-based package providing in-memory and on-disk (MakeDb, .asql) database
    variants whose runtimes are benchmarked
  repo: https://github.com/ArpiarSaundersLab/annsql
- name: AnnData
  role: Baseline object queried in not-backed and backed modes for runtime comparison
- name: DuckDB
  role: In-process engine underlying AnnSQL databases; used via driver in R or Go
    for cross-language queries
  repo: https://github.com/marcboeker/go-duckdb
- name: Splatter
  role: sc/nRNA-seq simulation tool used to generate sparser in silico datasets matching
    random-data shapes
- name: Python
  role: Environment (Python 3.12) and scripting language for timing, statistics, and
    plotting
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:33f7b0ee9d81884f564ebaeaf6f04a4ca15c4dd5e0b251dab3f7ec9df2d2222d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ArpiarSaundersLab/annsql
  tools:
  - AnnData
  - AnnSQL
  - DuckDB
  - Python
  - Splatter
  edam_operation: http://edamontology.org/operation_3840
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# sql-database-runtime-benchmarking

## Summary

Benchmark runtime and memory of SQL-based database querying (AnnSQL in-memory and on-disk via DuckDB) against AnnData not-backed/backed modes across filter and query operations, and test how data sparsity (e.g., Splatter-simulated sparse UMI counts) changes the comparisons with paired t-tests and Bonferroni correction.

## When to use

Use when comparing runtime or memory performance of a SQL-based single-cell database engine (AnnSQL/DuckDB) versus AnnData across filter/query operations, including re-testing on sparser simulated datasets.

## When NOT to use

- When the analysis goal is biological interpretation rather than runtime/memory profiling of query engines
- When datasets fit easily in memory and a one-off query is needed — the benchmark skill targets systematic multi-size comparisons, not single ad-hoc queries
- When databases contain more than 30k genes — a known unresolved DuckDB memory leak affects this regime

## Inputs

- AnnData objects (.h5ad) across dataset_sizes = [1000, 5000, 10000, 15000, 20000, 25000, 30000, 35000, 40000, 45000, 50000, 75000, 100000, 250000] cells with 10000 genes
- Splatter-simulated in silico sc/nRNA-seq datasets matching the shapes of the random simulated data
- Benchmark filter/query operations (e.g., filter_by_cell_counts, SQL filters on gene expression, group-by aggregate queries)

## Outputs

- Per-size runtime tables written to CSV via df.to_csv(index=False)
- Aggregated runtime comparison plots for AnnSQL in-memory, AnnSQL on-disk, AnnData not-backed, and AnnData backed
- Statistical test results: paired t-tests with Bonferroni-corrected alpha (0.05 / num_comparisons)

## How to apply

Create a conda environment with Python 3.12 and pip install annsql. Generate datasets of increasing size (dataset_sizes = [1000 ... 250000] cells with columns = 10000 genes), either randomly or with Splatter to match shapes while producing sparser UMI counts. Build on-disk AnnSQL databases with MakeDb using benchmark settings (db_name="random_data_"+str(i), create_all_indexes=False, convenience_view=False) and instantiate in-memory AnnSQL objects. Time the benchmark filter/query operations for AnnSQL in-memory, AnnSQL on-disk, AnnData not-backed, and AnnData backed for every size; then aggregate runtimes, apply paired t-tests with Bonferroni correction (alpha_corrected = 0.05 / num_comparisons), plot the aggregation, and write the runtime table with df.to_csv(index=False). To validate a sparsity effect, verify that runtimes of all AnnSQL-versus-AnnData comparisons decrease on Splatter-simulated sparse data relative to the random-data benchmark.

## Related tools

- **AnnSQL** (Python SQL-based package providing in-memory and on-disk (MakeDb, .asql) database variants whose runtimes are benchmarked) — https://github.com/ArpiarSaundersLab/annsql
- **AnnData** (Baseline object queried in not-backed and backed modes for runtime comparison)
- **DuckDB** (In-process engine underlying AnnSQL databases; used via driver in R or Go for cross-language queries) — https://github.com/marcboeker/go-duckdb
- **Splatter** (sc/nRNA-seq simulation tool used to generate sparser in silico datasets matching random-data shapes)
- **Python** (Environment (Python 3.12) and scripting language for timing, statistics, and plotting)

## Examples

```
MakeDb(adata, db_path="../db/random/", db_name="random_data_"+str(i), create_all_indexes=False, convenience_view=False)
```

## Evaluation signals

- All AnnSQL-versus-AnnData comparisons on Splatter-simulated data show decreased runtimes relative to the random-data benchmark, confirming the sparsity effect
- Dataset sizes match the benchmark grid exactly (1,000 to 250,000 cells, 10,000 genes per dataset)
- Paired t-test alpha is Bonferroni-corrected as 0.05 / num_comparisons before significance testing
- On-disk databases built with MakeDb use benchmark settings (create_all_indexes=False, convenience_view=False)
- Runtime table is written to CSV with df.to_csv(index=False) and contains measured values for all four method variants

## Limitations

- No software version numbers are reported in the benchmarking section
- No numeric memory/runtime result values appear in the code; results are only computed at runtime and written to CSV files
- Known unresolved issue: DuckDB memory leak on databases with more than 30k genes
- Runtimes are hardware- and sparsity-dependent: sparser UMI-count data (Splatter-simulated) lowers all runtimes, so conclusions do not transfer verbatim across datasets

## Evidence

- [results] Benchmark dataset sizes spanned the tested grid: "dataset_sizes = [1000, 5000, 10000, 15000, 20000, 25000, 30000, 35000, 40000, 45000, 50000, 75000, 100000, 250000]

columns = 10000"
- [results] Statistical testing used Bonferroni correction: "# Adjusted alpha using Bonferroni correction
alpha_corrected = 0.05 / num_comparisons"
- [results] On-disk databases built with benchmark MakeDb settings: "MakeDb(adata, db_path="../db/random/", db_name="random_data_"+str(i), create_all_indexes=False, convenience_view=False)"
- [readme] README describes the query surface being benchmarked: "The Python based AnnSQL package enables SQL-based queries on [AnnData](https://anndata.readthedocs.io/en/latest/) objects, returning results as either a [Pandas](https://pandas.pydata.org/) DataFrame"
- [methods] AnnSQL performance advantage for larger-than-memory datasets: "Running AnnSQL locally for datasets that are larger than memory, that would typically require AnnData in backed mode see substantial runtime improvements"
