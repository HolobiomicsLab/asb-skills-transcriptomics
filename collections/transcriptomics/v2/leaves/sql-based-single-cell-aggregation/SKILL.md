---
name: sql-based-single-cell-aggregation
description: Use when aggregative operations (per-cell count sums, gene counts, group-by
  expression averages, multi-gene filtered queries, table joins) on a single-cell
  AnnData matrix are too slow or memory-bound in AnnData/Scanpy backed mode or Seurat
  — e.g. datasets larger than memory such as the 4.
when_to_use_negative:
- Small datasets already fully in memory where Scanpy/AnnData operations are fast
  enough and the overhead of building a DuckDB database is not justified
- 'Databases with more than 30k genes, where the article records an unresolved DuckDB
  memory leak (TODO: debug duckdb memory leak on databases with >30k genes)'
- Situations that require a performance-tuned database schema — the AnnSQL tables
  are explicitly designed to mirror the AnnData object structure, not optimized for
  performance
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: AnnSQL
  role: Python package that converts AnnData layers into a relational DuckDB database
    and executes the SQL queries/aggregations, optionally via chunked extended methods
    like calculate_total_counts
  repo: github.com/ArpiarSaundersLab/annsql
- name: DuckDB
  role: In-process SQL engine behind AnnSQL that stores each AnnData layer as an individual
    table and evaluates the queries, including table joins
- name: AnnData
  role: Source single-cell object format (.h5ad) whose X and obs layers are converted
    into DuckDB tables; also a return format for query results
- name: Scanpy
  role: Loads the input datasets (e.g. sc.datasets.pbmc68k_reduced(), sc.read_h5ad
    of the atlas in backed mode) before database construction
- name: duckdb R driver
  role: Opens the on-disk .asql database directly in R via dbConnect(duckdb::duckdb(),
    dbdir=...) for dbGetQuery calls with no conversions
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:f937ced98216e37078e18d41ac7a132e915b30015e2ecf695ad56dc8d146143a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AnnData
  - AnnSQL
  - DuckDB
  - Python
  - Scanpy
  - Seurat
  - duckdb R driver
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# sql-based-single-cell-aggregation

## Summary

Run SQL queries and aggregations (sums, group-by averages, joins, QC counts) directly over AnnData single-cell matrices by converting AnnData layers into an in-process DuckDB relational database via AnnSQL, returning results as Pandas DataFrames, AnnData objects, or Parquet files.

## When to use

Use when aggregative operations (per-cell count sums, gene counts, group-by expression averages, multi-gene filtered queries, table joins) on a single-cell AnnData matrix are too slow or memory-bound in AnnData/Scanpy backed mode or Seurat — e.g. datasets larger than memory such as the 4.4-million-cell Macosko Mouse Atlas.

## When NOT to use

- Small datasets already fully in memory where Scanpy/AnnData operations are fast enough and the overhead of building a DuckDB database is not justified
- Databases with more than 30k genes, where the article records an unresolved DuckDB memory leak (TODO: debug duckdb memory leak on databases with >30k genes)
- Situations that require a performance-tuned database schema — the AnnSQL tables are explicitly designed to mirror the AnnData object structure, not optimized for performance

## Inputs

- AnnData object or .h5ad file (e.g. Scanpy pbmc3k / pbmc68k_reduced sample datasets, PCCM.h5ad, or the 4.4-million-cell Macosko Mouse Atlas Single Nuclei atlas loaded in backed mode)
- SQL query strings (SELECT/aggregation statements referencing the X and obs tables)

## Outputs

- Pandas DataFrame (default return type of asql.query)
- AnnData object (alternative return type)
- Parquet file (alternative return type)
- Derived count columns added by extended methods (total counts, gene counts, variable genes, PCA/UMAP/Leiden results)

## How to apply

Install AnnSQL with pip (article setup: conda create -n annsql python=3.12; pip install annsql), then load the AnnData object (.h5ad, e.g. sc.read_h5ad) and either instantiate AnnSQL(adata=adata) for small in-memory use, or build a persistent on-disk .asql database with MakeDb(adata=adata, db_name=..., db_path="db/"), remembering to save a raw matrix layer via asql.save_raw() when needed. Each AnnData layer becomes an individual DuckDB table (X, obs), so formulate standard SQL — e.g. SELECT obs.cell_type, AVG(gene_1) FROM X INNER JOIN obs ON X.cell_id = obs.cell_id GROUP BY obs.cell_type — and call asql.query(); results return as a Pandas DataFrame by default, with AnnData or Parquet as alternatives. For memory-intensive whole-matrix operations use the chunked extended methods (calculate_total_counts, calculate_gene_counts with chunk_size=750 and print_progress=True) rather than a single SELECT SUM(COLUMNS(*)) over X excluding cell_id, and use filter_by_cell_counts/filter_by_gene_counts thresholds appropriate to the dataset (e.g. min 2500 / max 40000 counts). The on-disk database can also be opened directly in R by connecting duckdb::duckdb(dbdir="db/....asql") and issuing dbGetQuery, with no format conversions. Note the table schema intentionally mirrors the AnnData object structure rather than being performance-optimized, so validate query logic against that layout.

## Related tools

- **AnnSQL** (Python package that converts AnnData layers into a relational DuckDB database and executes the SQL queries/aggregations, optionally via chunked extended methods like calculate_total_counts) — github.com/ArpiarSaundersLab/annsql
- **DuckDB** (In-process SQL engine behind AnnSQL that stores each AnnData layer as an individual table and evaluates the queries, including table joins)
- **AnnData** (Source single-cell object format (.h5ad) whose X and obs layers are converted into DuckDB tables; also a return format for query results)
- **Scanpy** (Loads the input datasets (e.g. sc.datasets.pbmc68k_reduced(), sc.read_h5ad of the atlas in backed mode) before database construction)
- **duckdb R driver** (Opens the on-disk .asql database directly in R via dbConnect(duckdb::duckdb(), dbdir=...) for dbGetQuery calls with no conversions)

## Examples

```
asql = AnnSQL(adata=sc.datasets.pbmc68k_reduced()); asql.query("SELECT obs.bulk_labels, COUNT(*) FROM obs GROUP BY obs.bulk_labels")
```

## Evaluation signals

- asql.query returns a Pandas DataFrame by default; a bare "SELECT * FROM adata LIMIT 10" should succeed and expose cell_id plus gene columns
- Aggregation correctness: SELECT SUM(COLUMNS(*)) FROM (SELECT * EXCLUDE (cell_id) FROM X) must match asql.calculate_total_counts output on the same data
- Group-by counts (SELECT obs.cell_type, COUNT(*) FROM obs GROUP BY obs.cell_type) must reproduce the obs label value counts
- On-disk .asql file must reopen in R via the duckdb driver and yield identical query results without conversion
- Whole-matrix sums run through chunked extended methods (chunk_size=750) with print_progress=True should complete without memory errors, matching the documented laptop runtime range for the atlas (minutes-scale, ~4.28–67 min for the 4.4M-cell dataset)

## Limitations

- Known unresolved DuckDB memory leak on databases with more than 30,000 genes
- The database schema mirrors AnnData structure and is not optimized for performance; complex aggregations may still be slow if not chunked
- Summing all gene counts with a single SELECT SUM(COLUMNS(*)) over X is memory intensive — use the chunked calculate_gene_counts approach instead
- No software version numbers are reported, and benchmark results are only computed at runtime and exported to CSV (e.g. atlas_profile_hpc.csv), so runtimes must be re-measured rather than looked up

## Evidence

- [intro] SQL queries over AnnData layers and return types: [evidence span withheld — anchor status: not_found]
- [intro] Database construction: each layer stored as an individual table: "AnnSQL converts the layers of an AnnData object into a relational [DuckDB](https://duckdb.org/) database. Each layer is stored as an individual table"
- [results] Join + group-by aggregate query pattern: "SELECT obs.cell_type, AVG(gene_1) as avg_gene_1 FROM X INNER JOIN obs ON X.cell_id = obs.cell_id GROUP BY obs.cell_type"
- [results] Chunked whole-matrix count calculation used for memory-intensive aggregation: "asql.calculate_total_counts(chunk_size=750,print_progress=True)"
- [readme] Query return and in-memory usage from the README: "#query the expression table. Returns Pandas Dataframe by Default
asql.query("SELECT * FROM adata LIMIT 10")"
- [readme] Memory-intensive full-row sum and chunked alternative flagged in README: "#sum all gene counts | Memory intensive | See method calculate_gene_counts for chunked approach.
asql.query("SELECT SUM(COLUMNS(*)) FROM (SELECT * EXCLUDE (cell_id) FROM X)")"
