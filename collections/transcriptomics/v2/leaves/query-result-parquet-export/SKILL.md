---
name: query-result-parquet-export
description: Use when a single-cell expression or annotation query (e.g. SELECT ...
  FROM X/obs with LIMIT or WHERE clauses) has been executed against an on-disk or
  in-memory AnnSQL DuckDB database and the result must be persisted as a Parquet file
  for reuse.
when_to_use_negative:
- You need an interactive in-memory result for immediate further AnnSQL/Scanpy operations
  — the default pandas DataFrame return is more appropriate than writing Parquet
- The data is not stored as AnnData layers converted to a relational DuckDB database
  (AnnSQL only queries AnnData-derived databases)
- The database has more than 30k genes, where a known unresolved DuckDB memory leak
  is flagged in the article
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: AnnSQL
  role: Provides the AnnSQL class whose query method executes SQL and returns pandas/AnnData/Parquet
    results; query_raw opens a db connection and executes a SQL statement directly
  repo: https://github.com/ArpiarSaundersLab/annsql
- name: DuckDB
  role: In-process engine that executes the SQL statement against the layer tables
    of the AnnData-inspired database
- name: Pandas
  role: Default return format of the query method, wrapping the DuckDB result rows
- name: AnnData
  role: Object model whose layers form the queried database; also an alternative return
    format for query results
- name: Python
  role: Language in which the AnnSQL package and query workflow are implemented
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:3b0a46d5b31eac3dc41912169cc8a23225ca036f9ffec5c8754117398f2e6a9e
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# query-result-parquet-export

## Summary

Run SQL against an instantiated AnnSQL database via the query method and capture results as a Parquet file (pandas DataFrame is the default return format; AnnData is also supported), persisting query output for downstream tools.

## When to use

Use when a single-cell expression or annotation query (e.g. SELECT ... FROM X/obs with LIMIT or WHERE clauses) has been executed against an on-disk or in-memory AnnSQL DuckDB database and the result must be persisted as a Parquet file for reuse.

## When NOT to use

- You need an interactive in-memory result for immediate further AnnSQL/Scanpy operations — the default pandas DataFrame return is more appropriate than writing Parquet
- The data is not stored as AnnData layers converted to a relational DuckDB database (AnnSQL only queries AnnData-derived databases)
- The database has more than 30k genes, where a known unresolved DuckDB memory leak is flagged in the article

## Inputs

- On-disk AnnSQL database file (e.g. db/pbmc3k_reduced.asql) or an in-memory AnnSQL instance built from an AnnData object (e.g. sc.datasets.pbmc68k_reduced())
- SQL statement string targeting layer tables such as X or obs (e.g. "SELECT * FROM adata LIMIT 5")

## Outputs

- query_results.parquet (Parquet file of query results)
- Optional alternative return formats: pandas DataFrame (default) or AnnData object

## How to apply

Instantiate the AnnSQL class against the on-disk database produced by MakeDb using AnnSQL(db="db/pbmc3k_reduced.asql"), then execute the SQL statement with asql.query(...), which lets the in-process DuckDB engine run against the layer tables. The query method returns results as a pandas DataFrame by default, with AnnData and Parquet return formats supported by the same method — request the Parquet output for persistence. For direct control, query_raw opens a db connection and executes the SQL statement directly. Validate by checking that query_results.parquet exists, reads as a valid Parquet file, and contains exactly the rows the SQL requested (e.g. 5 rows for a LIMIT 5 query).

## Related tools

- **AnnSQL** (Provides the AnnSQL class whose query method executes SQL and returns pandas/AnnData/Parquet results; query_raw opens a db connection and executes a SQL statement directly) — https://github.com/ArpiarSaundersLab/annsql
- **DuckDB** (In-process engine that executes the SQL statement against the layer tables of the AnnData-inspired database)
- **Pandas** (Default return format of the query method, wrapping the DuckDB result rows)
- **AnnData** (Object model whose layers form the queried database; also an alternative return format for query results)
- **Python** (Language in which the AnnSQL package and query workflow are implemented)

## Examples

```
from AnnSQL import AnnSQL; asql = AnnSQL(db="db/pbmc3k_reduced.asql"); df = asql.query("SELECT * FROM adata LIMIT 5"); df.to_parquet("query_results.parquet")
```

## Evaluation signals

- query_results.parquet exists on disk after the query call
- The file is readable as a valid Parquet file (e.g. loaded without error by a Parquet reader)
- Row count of the Parquet file exactly matches the SQL request (e.g. 5 rows for a LIMIT 5 query)
- Column content matches the SELECT list (e.g. gene columns from X, or obs.cell_type from a grouped join query)
- The database path passed to AnnSQL(db=...) points to an existing .asql file produced by MakeDb

## Limitations

- Known unresolved issue: DuckDB memory leak on databases with more than 30k genes
- Query results must be written to CSV/Parquet to capture values; numeric memory/runtime values are computed only at runtime
- Summing all gene counts via SQL (SELECT SUM(COLUMNS(*)) FROM X) is memory intensive; the chunked asql.calculate_gene_counts approach is recommended instead

## Evidence

- [methods] Query method return formats: [evidence span withheld — anchor status: not_found]
- [readme] On-disk open and query usage: "#open the AnnSQL database
asql = AnnSQL(db="db/pbmc3k_reduced.asql")

#query the expression table
asql.query("SELECT * FROM adata LIMIT 5")"
- [intro] Parquet import into analysis tools: "returning results as either a [Pandas](https://pandas.pydata.org/) DataFrame, an AnnData object, or a [Parquet](https://parquet.apache.org/) file"
