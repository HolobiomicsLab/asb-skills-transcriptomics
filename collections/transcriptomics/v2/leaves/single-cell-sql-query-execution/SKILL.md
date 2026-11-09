---
name: single-cell-sql-query-execution
description: Use when an AnnData/.h5ad single-cell dataset has been converted into
  an AnnSQL DuckDB database and you need to run SELECT-style SQL (including joins
  on layer tables) and capture results as a DataFrame, AnnData object, or Parquet
  file.
when_to_use_negative:
- The data is still a raw AnnData/.h5ad object and no AnnSQL database or AnnData input
  has been prepared — build the database first via MakeDb or pass adata directly to
  AnnSQL.
- The workflow requires memory-light iterative single-cell preprocessing on datasets
  larger than memory where AnnData backed mode is sufficient and SQL semantics are
  not needed.
- The query requires a relational schema optimized for performance rather than the
  AnnData-mirroring table layout AnnSQL uses.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3170
tools:
- name: AnnSQL
  role: Python package whose AnnSQL class is instantiated against the .asql database;
    the query method executes SQL via the DuckDB engine and returns pandas/AnnData/Parquet
    results, while query_raw opens a db connection and executes SQL directly
  repo: https://github.com/ArpiarSaundersLab/annsql
- name: DuckDB
  role: In-process engine underlying the AnnSQL database; executes the SQL against
    relational tables converted from AnnData layers
- name: Pandas
  role: Default container for query results returned by the query method
- name: AnnData
  role: Source object format whose layers become DuckDB tables; also an optional return
    format for query results
- name: Scanpy
  role: Provides sample datasets (pbmc3k, pbmc68k_reduced) used to build the AnnData
    objects queried in examples
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:035fc94bc77df4899ecff2580334e2d2596a6af5a6238b4a5aaddcf84f606033
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
  - Scanpy
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# single-cell-sql-query-execution

## Summary

Executes SQL statements against an AnnSQL on-disk (.asql) or in-memory DuckDB database built from an AnnData object, returning results as a pandas DataFrame (default), AnnData object, or Parquet file.

## When to use

Use when an AnnData/.h5ad single-cell dataset has been converted into an AnnSQL DuckDB database and you need to run SELECT-style SQL (including joins on layer tables) and capture results as a DataFrame, AnnData object, or Parquet file.

## When NOT to use

- The data is still a raw AnnData/.h5ad object and no AnnSQL database or AnnData input has been prepared — build the database first via MakeDb or pass adata directly to AnnSQL.
- The workflow requires memory-light iterative single-cell preprocessing on datasets larger than memory where AnnData backed mode is sufficient and SQL semantics are not needed.
- The query requires a relational schema optimized for performance rather than the AnnData-mirroring table layout AnnSQL uses.

## Inputs

- On-disk AnnSQL database file (e.g. db/pbmc3k_reduced.asql) built by MakeDb from an AnnData object
- AnnData object or .h5ad file (for in-memory instantiation, e.g. sc.datasets.pbmc68k_reduced())
- SQL statement string against layer tables (e.g. 'SELECT * FROM adata LIMIT 5')

## Outputs

- pandas DataFrame (default return format)
- AnnData object (alternative return format)
- Parquet file (alternative return format, e.g. query_results.parquet)

## How to apply

Instantiate the AnnSQL class against the on-disk database produced by MakeDb (e.g. AnnSQL(db="db/pbmc3k_reduced.asql")) or against an AnnData/h5ad object for in-memory use. Run SQL through asql.query(), e.g. asql.query("SELECT * FROM adata LIMIT 5"), letting the in-process DuckDB engine execute against the layer tables (X, obs, etc.), each stored as an individual table so joins are supported. Capture the returned result — by default a pandas DataFrame; pass return-format options to get an AnnData object or a Parquet file instead. For direct connection-level execution, use query_raw, which opens a db connection and executes the SQL statement directly. Validate by confirming the output exists (e.g. query_results.parquet), is readable as Parquet, and contains exactly the rows requested (e.g. 5 rows for LIMIT 5).

## Related tools

- **AnnSQL** (Python package whose AnnSQL class is instantiated against the .asql database; the query method executes SQL via the DuckDB engine and returns pandas/AnnData/Parquet results, while query_raw opens a db connection and executes SQL directly) — https://github.com/ArpiarSaundersLab/annsql
- **DuckDB** (In-process engine underlying the AnnSQL database; executes the SQL against relational tables converted from AnnData layers)
- **Pandas** (Default container for query results returned by the query method)
- **AnnData** (Source object format whose layers become DuckDB tables; also an optional return format for query results)
- **Scanpy** (Provides sample datasets (pbmc3k, pbmc68k_reduced) used to build the AnnData objects queried in examples)

## Examples

```
asql = AnnSQL(db="db/pbmc3k_reduced.asql"); df = asql.query("SELECT * FROM adata LIMIT 5"); df.to_parquet("query_results.parquet")
```

## Evaluation signals

- query_results.parquet exists and is readable as a Parquet file
- The output contains exactly the rows requested by the SQL (e.g. 5 rows for a LIMIT 5 query)
- Default query() return is a pandas DataFrame with columns matching the SELECT clause (e.g. gene columns from the X table)
- The database path passed to AnnSQL(db=...) points to an existing .asql on-disk database built by MakeDb
- For query_raw, a db connection is opened and the SQL statement executes directly against the DuckDB tables without error

## Limitations

- The AnnSQL database structure mirrors the AnnData object structure rather than being optimized for performance.
- Known unresolved issue: DuckDB memory leak on databases with more than 30k genes.
- Aggregative queries such as SUM over all gene columns are memory intensive; chunked methods (e.g. calculate_gene_counts with chunk_size=750) are recommended instead.

## Evidence

- [intro] Query method returns pandas/AnnData/Parquet: [evidence span withheld — anchor status: not_found]
- [intro] Layers stored as individual tables: [evidence span withheld — anchor status: not_found]
- [readme] On-disk open and query pattern: "asql = AnnSQL(db="db/pbmc3k_reduced.asql")"
- [readme] Example SQL query returning DataFrame: "asql.query("SELECT * FROM adata LIMIT 5")"
- [methods] Database mirrors AnnData not performance-optimized: "The database structure is not optimized for performance. Instead, the tables are designed to closely mirror the familiar structure of the AnnData object"
