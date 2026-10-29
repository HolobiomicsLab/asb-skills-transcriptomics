---
name: anndata-layer-table-querying
description: Use when an AnnData object (e.g. pbmc3k or pbmc68k_reduced .h5ad) has
  been converted to an on-disk or in-memory AnnSQL/DuckDB database and results must
  be retrieved with SQL, with output as pandas, AnnData, or Parquet.
when_to_use_negative:
- The data is not an AnnData object or AnnSQL database — this skill queries AnnData-derived
  layer tables only
- You need iterative QC preprocessing (filter_by_cell_counts, normalize, PCA/UMAP)
  rather than ad-hoc SQL retrieval
- The database has more than 30k genes, where a DuckDB memory leak is noted as unresolved
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: AnnSQL
  role: Provides the AnnSQL class whose query method executes SQL against the instantiated
    database and returns pandas, AnnData, or Parquet results; query_raw opens a connection
    and executes SQL directly
  repo: github.com/ArpiarSaundersLab/annsql
- name: DuckDB
  role: In-process engine that executes the SQL statements against the relational
    tables backing each AnnData layer
  repo: github.com/ArpiarSaundersLab/annsql
- name: Pandas
  role: Default container for returned query results
- name: AnnData
  role: Source object format whose layers become the database tables; also an optional
    return format
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:08b96381f520c1d4f0e27eb97abd3cc85e19dcc117265918e623a63898fa99fb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AnnData
  - AnnSQL
  - DuckDB
  - Pandas
  - Python
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# anndata-layer-table-querying

## Summary

Executes SQL statements against an AnnSQL database (an AnnData object converted into a relational DuckDB database with each layer stored as an individual table) via the AnnSQL class's query method, returning a pandas DataFrame, AnnData object, or Parquet file.

## When to use

Use when an AnnData object (e.g. pbmc3k or pbmc68k_reduced .h5ad) has been converted to an on-disk or in-memory AnnSQL/DuckDB database and results must be retrieved with SQL, with output as pandas, AnnData, or Parquet.

## When NOT to use

- The data is not an AnnData object or AnnSQL database — this skill queries AnnData-derived layer tables only
- You need iterative QC preprocessing (filter_by_cell_counts, normalize, PCA/UMAP) rather than ad-hoc SQL retrieval
- The database has more than 30k genes, where a DuckDB memory leak is noted as unresolved

## Inputs

- On-disk AnnSQL database (.asql file, e.g. db/pbmc3k_reduced.asql) built with MakeDb
- In-memory AnnData object or .h5ad file passed via the adata parameter
- SQL query string targeting layer tables (X/adata, obs)

## Outputs

- pandas DataFrame (default return format)
- AnnData object (optional return format)
- Parquet file (optional return format, e.g. query_results.parquet)

## How to apply

Instantiate the AnnSQL class against the on-disk database produced by MakeDb, e.g. AnnSQL(db="db/pbmc3k_reduced.asql"), or pass an AnnData object / h5ad file for in-memory use. Run SQL through asql.query(...), letting the in-process DuckDB engine execute against the layer tables (the expression table is queried as 'adata' or 'X', joined with 'obs' via cell_id). By default query returns a pandas DataFrame; the method also supports returning an AnnData object or writing a Parquet file. For direct low-level execution, query_raw opens a db connection and executes a SQL statement directly. Keep queries within DuckDB semantics — e.g. SELECT ... FROM X WHERE gene > 0.5, LOG10(), CORR(), GROUP BY on obs columns, or SUM(COLUMNS(*)) after EXCLUDE (cell_id).

## Related tools

- **AnnSQL** (Provides the AnnSQL class whose query method executes SQL against the instantiated database and returns pandas, AnnData, or Parquet results; query_raw opens a connection and executes SQL directly) — github.com/ArpiarSaundersLab/annsql
- **DuckDB** (In-process engine that executes the SQL statements against the relational tables backing each AnnData layer) — github.com/ArpiarSaundersLab/annsql
- **Pandas** (Default container for returned query results)
- **AnnData** (Source object format whose layers become the database tables; also an optional return format)

## Examples

```
from AnnSQL import AnnSQL; asql = AnnSQL(db="db/pbmc3k_reduced.asql"); asql.query("SELECT * FROM adata LIMIT 5")
```

## Evaluation signals

- The returned object is a pandas DataFrame by default (or AnnData/Parquet if that format was requested)
- For a LIMIT 5 query the output contains exactly the requested rows (e.g. 5 rows), verifiable after writing query_results.parquet and reading it back as Parquet
- The Parquet output file exists and is readable as a valid Parquet file
- Query results are consistent with the layer tables — e.g. obs joins must match X/adata rows on cell_id
- The database path passed to AnnSQL matches the .asql database produced by MakeDb

## Limitations

- Known unresolved DuckDB memory leak on databases with more than 30k genes
- The database structure mirrors the AnnData object rather than being optimized for performance
- Aggregations like SUM over all genes are memory intensive; chunked methods such as calculate_gene_counts are recommended instead

## Evidence

- [methods] query method interfaces with the database to return a pandas dataframe (default), AnnData, or Parquet file: [evidence span withheld — anchor status: not_found]
- [readme] layers become individual DuckDB tables: "Behind the scenes, AnnSQL converts the layers of an AnnData object into a relational [DuckDB](https://duckdb.org/) database. Each layer is stored as an individual table"
- [readme] on-disk open and query invocation: "asql = AnnSQL(db="db/pbmc3k_reduced.asql")

#query the expression table
asql.query("SELECT * FROM adata LIMIT 5")"
