---
name: anndata-backed-mode-query-benchmarking
description: Decision support for choosing between AnnData backed-mode h5ad access
  and on-disk AnnSQL/DuckDB when filtering or querying large single-cell matrix datasets,
  especially when datasets exceed available memory.
when_to_use_negative:
- Datasets smaller than 5,000 cells, where on-disk AnnData was faster than on-disk
  AnnSQL
- AnnData objects that fit fully in memory and need only simple slicing, where backed-mode
  or in-memory AnnData is adequate
- Databases with more than 30k genes, where a known unresolved DuckDB memory leak
  affects AnnSQL
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_3170
tools:
- name: AnnSQL
  role: On-disk DuckDB-based query engine whose filter/query runtimes are benchmarked
    against AnnData; builds databases via MakeDb
  repo: github.com/ArpiarSaundersLab/annsql
- name: AnnData
  role: Baseline engine queried in backed-mode and in-memory (h5ad) for runtime comparison
- name: DuckDB
  role: In-process engine underlying AnnSQL on-disk databases
  repo: marcboeker/go-duckdb
- name: scanpy
  role: Provides sc.read_h5ad and sample datasets (pbmc3k, pbmc68k_reduced) for constructing/loading
    AnnData objects
- name: Pandas
  role: Stores recorded runtimes and writes CSV files with df.to_csv(..., index=False)
- name: matplotlib
  role: Plots the aggregated runtime comparison across filters and dataset sizes
- name: memory_profiler
  role: Profiles memory usage alongside runtime in the benchmark scripts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:6ba855bfd1ca852c254443e40d84b176cc625c67a667a84ec58bc550e1fbdf66
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5789473684210527
n_claims: 19
---

# anndata-backed-mode-query-benchmarking

## Summary

Benchmarks AnnData (backed-mode and in-memory) query runtimes against on-disk AnnSQL (DuckDB) across simulated single-cell datasets from 1,000 to 250,000 cells with 10,000 genes, to quantify where SQL on-disk querying outperforms h5ad-backed slicing.

## When to use

Decision support for choosing between AnnData backed-mode h5ad access and on-disk AnnSQL/DuckDB when filtering or querying large single-cell matrix datasets, especially when datasets exceed available memory.

## When NOT to use

- Datasets smaller than 5,000 cells, where on-disk AnnData was faster than on-disk AnnSQL
- AnnData objects that fit fully in memory and need only simple slicing, where backed-mode or in-memory AnnData is adequate
- Databases with more than 30k genes, where a known unresolved DuckDB memory leak affects AnnSQL

## Inputs

- Simulated dense AnnData objects (.h5ad) with sizes from 1,000 to 250,000 cells and 10,000 genes of random expression values
- On-disk AnnSQL databases (.asql, DuckDB) built from each simulated AnnData object
- Filter/query definitions (cell-count ranges, SQL predicates on gene expression)

## Outputs

- CSV runtime tables (df.to_csv(..., index=False)) recording per-size, per-filter timings
- Aggregated runtime comparison plot (runtime_comparison.png) from matplotlib
- Paired t-test p-values with Bonferroni-corrected alpha (0.05 / num_comparisons)

## How to apply

Generate simulated dense AnnData objects with 10,000 genes for dataset_sizes = [1000, 5000, 10000, 15000, 20000, 25000, 30000, 35000, 40000, 45000, 50000, 75000, 100000, 250000] using random expression values. Build an on-disk AnnSQL database for each with MakeDb(db_path='../db/random/', db_name='random_data_'+str(i), create_all_indexes=False, convenience_view=False). Time equivalent filter operations on both engines, e.g. filter_by_cell_counts(min_cell_count=2500, max_cell_count=40000) on AnnSQL and backed-mode AnnData slicing such as adata_mem[adata_mem[:, 'ENSMUSG00000070880'].X > 0, 'ENSMUSG00000070880'] or SELECT gene_1 FROM X WHERE gene_1 > 0.5. Aggregate runtimes with pandas (df.to_csv(..., index=False)) and compare per size with paired t-tests using Bonferroni correction alpha_corrected = 0.05 / num_comparisons. Expect the crossover below 5,000 cells (AnnData faster) and roughly 400x speedup for AnnSQL at 250,000 cells (0.14 s vs 57 s).

## Related tools

- **AnnSQL** (On-disk DuckDB-based query engine whose filter/query runtimes are benchmarked against AnnData; builds databases via MakeDb) — github.com/ArpiarSaundersLab/annsql
- **AnnData** (Baseline engine queried in backed-mode and in-memory (h5ad) for runtime comparison)
- **DuckDB** (In-process engine underlying AnnSQL on-disk databases) — marcboeker/go-duckdb
- **scanpy** (Provides sc.read_h5ad and sample datasets (pbmc3k, pbmc68k_reduced) for constructing/loading AnnData objects)
- **Pandas** (Stores recorded runtimes and writes CSV files with df.to_csv(..., index=False))
- **matplotlib** (Plots the aggregated runtime comparison across filters and dataset sizes)
- **memory_profiler** (Profiles memory usage alongside runtime in the benchmark scripts)

## Examples

```
MakeDb(adata=adata, db_path='../db/random/', db_name='random_data_'+str(i), create_all_indexes=False, convenience_view=False); asql.filter_by_cell_counts(min_cell_count=2500, max_cell_count=40000)
```

## Evaluation signals

- Crossover point: AnnData (on-disk) should outperform on-disk AnnSQL only for libraries <5,000 cells
- At 250,000 cells, AnnSQL filter completes in approximately 0.14 s versus approximately 57 s for AnnData (~400x speedup)
- Paired t-tests across sizes use Bonferroni-corrected alpha_corrected = 0.05 / num_comparisons
- CSV runtime tables contain one timing per filter per dataset size with no missing entries for the 14 sizes tested
- On-disk database creation uses create_all_indexes=False and convenience_view=False consistently across sizes so timings are comparable

## Limitations

- No software version numbers are reported for AnnSQL, scanpy, or AnnData in the benchmark scripts
- Known unresolved DuckDB memory leak on databases with more than 30k genes
- The AnnSQL database structure mirrors the AnnData object rather than being optimized for performance
- Numeric result values are computed at runtime and written to CSV; they are not fixed constants in the code and depend on hardware

## Evidence

- [results] On-disk AnnData outperformed on-disk AnnSQL only for libraries <5000 cells, while queries of 250 000 cells were on average ∼400 times faster using AnnSQL (0.14 versus 57 seconds).: "On-disk AnnData outperformed on-disk AnnSQL only for libraries <5000 cells, while queries of 250 000 cells were on average ∼400 times faster using AnnSQL"
- [results] MakeDb(adata, db_path='../db/random/', db_name='random_data_'+str(i), create_all_indexes=False, convenience_view=False): "MakeDb(adata, db_path="../db/random/", db_name="random_data_"+str(i), create_all_indexes=False, convenience_view=False)"
- [results] # Adjusted alpha using Bonferroni correction
alpha_corrected = 0.05 / num_comparisons: "# Adjusted alpha using Bonferroni correction
alpha_corrected = 0.05 / num_comparisons"
- [methods] Running AnnSQL locally for datasets that are larger than memory, that would typically require AnnData in backed mode see substantial runtime improvements: "Running AnnSQL locally for datasets that are larger than memory, that would typically require AnnData in backed mode see substantial runtime improvements"
- [methods] The database structure is not optimized for performance. Instead, the tables are designed to closely mirror the familiar structure of the AnnData object: "The database structure is not optimized for performance. Instead, the tables are designed to closely mirror the familiar structure of the AnnData object"
