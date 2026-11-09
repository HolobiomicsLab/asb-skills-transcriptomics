---
name: pandas-dataframe-result-handling
description: Use when SQL has been executed against an AnnSQL database (e.g. on an
  .asql file built from an .h5ad AnnData object) and the result must be captured as
  a pandas DataFrame, AnnData object, or Parquet file.
when_to_use_negative:
- When the data still needs to be converted into an AnnSQL database first — use the
  MakeDb database-construction step instead of querying
- When a memory-intensive whole-matrix aggregation is needed on a large dataset (e.g.
  SUM over all genes); the README notes asql.query("SELECT SUM(COLUMNS(*)) FROM ...")
  is memory intensive and recommends the chunked calculate_gene_counts method instead
- When the input is not AnnData-derived and layer tables are not required — a plain
  DuckDB or pandas workflow is simpler
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: AnnSQL
  role: Provides the query method that executes SQL against the DuckDB database and
    returns a pandas DataFrame, AnnData object, or Parquet file; query_raw opens a
    db connection and executes a SQL statement directly
  repo: https://github.com/ArpiarSaundersLab/annsql
- name: DuckDB
  role: In-process engine that executes the SQL statement against the layer tables
    backing the AnnSQL database
- name: Pandas
  role: Default return container for query results, written out to Parquet for validation
- name: AnnData
  role: Alternate return format for query results used in Scanpy-style downstream
    workflows
  repo: https://anndata.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:561375c5c86c9c08a1a5d1c0a28028224faed65dc802add2034acfcc68b878d0
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# pandas-dataframe-result-handling

## Summary

Captures and materializes AnnSQL query results, which return as a pandas DataFrame by default with AnnData and Parquet formats also supported, after executing SQL against an on-disk or in-memory AnnSQL (DuckDB) database.

## When to use

Use when SQL has been executed against an AnnSQL database (e.g. on an .asql file built from an .h5ad AnnData object) and the result must be captured as a pandas DataFrame, AnnData object, or Parquet file.

## When NOT to use

- When the data still needs to be converted into an AnnSQL database first — use the MakeDb database-construction step instead of querying
- When a memory-intensive whole-matrix aggregation is needed on a large dataset (e.g. SUM over all genes); the README notes asql.query("SELECT SUM(COLUMNS(*)) FROM ...") is memory intensive and recommends the chunked calculate_gene_counts method instead
- When the input is not AnnData-derived and layer tables are not required — a plain DuckDB or pandas workflow is simpler

## Inputs

- On-disk AnnSQL database file (db/pbmc3k_reduced.asql) built by MakeDb from an AnnData .h5ad object
- In-memory AnnSQL instance created from an AnnData object (e.g. sc.datasets.pbmc68k_reduced())
- SQL SELECT statement string (e.g. "SELECT * FROM adata LIMIT 5")

## Outputs

- pandas DataFrame (default return format of asql.query)
- AnnData object (alternate return format)
- Parquet file (alternate return format, e.g. query_results.parquet)

## How to apply

First instantiate the AnnSQL class against the database: for an on-disk database produced by MakeDb, use AnnSQL(db="db/pbmc3k_reduced.asql"); for small datasets, pass the AnnData object directly (AnnSQL(adata=adata)). Then call asql.query("SELECT * FROM adata LIMIT 5"), which lets the in-process DuckDB engine execute against the layer tables; the return value defaults to a pandas DataFrame. Decide the output format based on downstream needs — DataFrame for tabular inspection, AnnData for Scanpy-style workflows, Parquet for persistence/interchange. To persist a result, write the returned DataFrame to a .parquet file (e.g. query_results.parquet) and verify it exists, is readable as Parquet, and contains exactly the rows requested by the SQL. For direct low-level access, the query_raw method opens a database connection and executes the SQL statement directly.

## Related tools

- **AnnSQL** (Provides the query method that executes SQL against the DuckDB database and returns a pandas DataFrame, AnnData object, or Parquet file; query_raw opens a db connection and executes a SQL statement directly) — https://github.com/ArpiarSaundersLab/annsql
- **DuckDB** (In-process engine that executes the SQL statement against the layer tables backing the AnnSQL database)
- **Pandas** (Default return container for query results, written out to Parquet for validation)
- **AnnData** (Alternate return format for query results used in Scanpy-style downstream workflows) — https://anndata.readthedocs.io/en/latest/

## Examples

```
from AnnSQL import AnnSQL; asql = AnnSQL(db="db/pbmc3k_reduced.asql"); df = asql.query("SELECT * FROM adata LIMIT 5"); df.to_parquet("query_results.parquet")
```

## Evaluation signals

- query_results.parquet exists on disk and is readable as a Parquet file
- The Parquet/DataFrame contains exactly the rows requested by the SQL (e.g. 5 rows for a LIMIT 5 query)
- Returned object is a pandas DataFrame by default, or AnnData/Parquet when an alternate format is requested
- Column names reflect the queried layer table (e.g. X gene columns or obs fields such as bulk_labels)
- For query_raw, the statement executes over an explicitly opened database connection without format conversion

## Limitations

- Whole-matrix aggregations via query (e.g. SUM(COLUMNS(*)) over the X table) are memory intensive; use chunked methods like calculate_gene_counts for large datasets
- DuckDB has a known unresolved memory leak on databases with more than 30k genes
- The AnnSQL database structure mirrors the AnnData object rather than being optimized for performance

## Evidence

- [readme] Query method returns DataFrame by default: "#query the expression table. Returns Pandas Dataframe by Default
asql.query("SELECT * FROM adata LIMIT 10")"
- [intro] Article: return formats: [evidence span withheld — anchor status: not_found]
- [methods] On-disk query flow: "asql = AnnSQL(db="db/pbmc3k_reduced.asql")"
- [readme] README: Parquet interchange: "Return query results as **Pandas** Dataframes, **Parquet** files, or **AnnData** objects."
- [readme] README: memory-intensive aggregation caveat: [evidence span withheld — anchor status: external_doc]
