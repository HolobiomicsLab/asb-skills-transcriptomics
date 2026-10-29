---
name: duckdb-in-process-database-access
description: Use when single-cell data stored as AnnData layers must be filtered or
  aggregated with SQL through an in-process or on-disk DuckDB database (.asql), with
  results returned as pandas DataFrames, AnnData objects, or Parquet files.
when_to_use_negative:
- The input is not AnnData-derived — AnnSQL expects AnnData layers converted to relational
  DuckDB tables, so raw mzML/FASTQ or other formats need a different access path
- The database has more than 30k genes — a known unresolved DuckDB memory leak (TODO
  debug duckdb memory leak on daabases with >30k genes) makes this risky
- You need a performance-optimized schema; AnnSQL tables deliberately mirror the AnnData
  object structure instead
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: AnnSQL
  role: Python wrapper that opens in-memory or on-disk .asql databases and exposes
    the query method returning DataFrame/AnnData/Parquet
  repo: https://github.com/ArpiarSaundersLab/annsql
- name: DuckDB
  role: In-process engine that executes the SQL statements against the individual
    layer tables
- name: Pandas
  role: Default container for query results returned by AnnSQL.query()
- name: AnnData
  role: Source object format whose layers become the database tables; also an optional
    query return type
  repo: https://anndata.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:f293dad1e2b925c6eda8bff32b82856d2111be000b17aedb493db22f5737117b
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
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# duckdb-in-process-database-access

## Summary

Execute SQL against an AnnData-inspired database running on the in-process DuckDB engine via AnnSQL: on-disk .asql databases are opened with AnnSQL(db=...), and the query method returns a pandas DataFrame by default, or AnnData/Parquet files.

## When to use

Use when single-cell data stored as AnnData layers must be filtered or aggregated with SQL through an in-process or on-disk DuckDB database (.asql), with results returned as pandas DataFrames, AnnData objects, or Parquet files.

## When NOT to use

- The input is not AnnData-derived — AnnSQL expects AnnData layers converted to relational DuckDB tables, so raw mzML/FASTQ or other formats need a different access path
- The database has more than 30k genes — a known unresolved DuckDB memory leak (TODO debug duckdb memory leak on daabases with >30k genes) makes this risky
- You need a performance-optimized schema; AnnSQL tables deliberately mirror the AnnData object structure instead

## Inputs

- AnnData object or .h5ad file (e.g. sc.datasets.pbmc68k_reduced(), pbmc3k)
- On-disk AnnSQL database (.asql) built by MakeDb
- SQL query string (e.g. SELECT * FROM adata LIMIT 5)

## Outputs

- pandas DataFrame (default return format)
- AnnData object (optional return format)
- Parquet file (optional return format, e.g. query_results.parquet)

## How to apply

Build the database from an AnnData object with MakeDb(adata=adata, db_name="pbmc3k_reduced", db_path="db/"), which persists a .asql database, or instantiate in-memory with AnnSQL(adata=adata) for smaller datasets like pbmc68k_reduced. Open the on-disk database with AnnSQL(db="db/pbmc3k_reduced.asql") and run statements through the query method (e.g. asql.query("SELECT * FROM adata LIMIT 5")); DuckDB executes each statement against layer tables, each stored as an individual table, so simple filters, aggregations (SUM, AVG, COUNT with GROUP BY), and table joins on cell_id all work. Decide the return format per call: pandas DataFrame is the default, with AnnData and Parquet also supported — write Parquet results to files like query_results.parquet for downstream tools. Use query_raw when you need to open a db connection and execute a SQL statement directly rather than through the wrapped return-format machinery. For whole-matrix sums avoid the memory-intensive SUM(COLUMNS(*)) pattern and prefer the chunked helper asql.calculate_gene_counts(chunk_size=750). In R, connect to the same .asql file via dbConnect(duckdb::duckdb(), dbdir="db/pbmc3k.asql") and dbGetQuery — no conversion necessary.

## Related tools

- **AnnSQL** (Python wrapper that opens in-memory or on-disk .asql databases and exposes the query method returning DataFrame/AnnData/Parquet) — https://github.com/ArpiarSaundersLab/annsql
- **DuckDB** (In-process engine that executes the SQL statements against the individual layer tables)
- **Pandas** (Default container for query results returned by AnnSQL.query())
- **AnnData** (Source object format whose layers become the database tables; also an optional query return type) — https://anndata.readthedocs.io/en/latest/

## Examples

```
asql = AnnSQL(db="db/pbmc3k_reduced.asql"); df = asql.query("SELECT * FROM adata LIMIT 5")
```

## Evaluation signals

- query_results.parquet exists, is readable as Parquet, and contains exactly the rows requested by the SQL (e.g. 5 rows for a LIMIT 5 query)
- Default query returns a pandas DataFrame; requesting AnnData or Parquet return formats yields objects/files of the corresponding type
- Joins such as SELECT obs.cell_type, AVG(gene_1) FROM X INNER JOIN obs ON X.cell_id = obs.cell_id GROUP BY obs.cell_type execute without schema errors, confirming layer tables are present
- The same .asql file queried from R via the duckdb driver with dbGetQuery returns rows consistent with the Python query
- In-memory (AnnSQL(adata=adata)) and on-disk (AnnSQL(db=...)) instantiations both expose the adata table for identical queries

## Limitations

- Known unresolved issue: DuckDB memory leak on databases with more than 30k genes
- The database structure is not optimized for performance; tables closely mirror the AnnData object structure
- SELECT SUM(COLUMNS(*)) across the X table is memory intensive — use the chunked calculate_gene_counts method instead
- No software version numbers are reported for the tools in this article

## Evidence

- [abstract] constructs an AnnData-inspired database using the in-process DuckDb engine: "constructs an AnnData-inspired database using the in-process DuckDb engine"
- [intro] AnnSQL converts the layers of an AnnData object into a relational [DuckDB](https://duckdb.org/) database: "AnnSQL converts the layers of an AnnData object into a relational [DuckDB](https://duckdb.org/) database"
- [methods] The Python based AnnSQL package enables SQL-based queries on AnnData objects, returning results as either a Pandas DataFrame, an AnnData object, or a Parquet file: [evidence span withheld — anchor status: not_found]
- [methods] asql = AnnSQL(db="db/pbmc3k_reduced.asql"): "asql = AnnSQL(db="db/pbmc3k_reduced.asql")"
- [readme] Each layer is stored as an individual table, allowing for simple or complex SQL queries, including table joins.: "Each layer is stored as an individual table, allowing for simple or complex SQL queries, including table joins."
- [results] #TODO debug duckdb memory leak on daabases with >30k genes.: "#TODO debug duckdb memory leak on daabases with >30k genes."
