---
name: single-cell-matrix-serialization
description: Use when a single-cell dataset stored as an AnnData (.h5ad) object or
  a Scanpy sample dataset must be converted into a persistent, SQL-queryable on-disk
  database so expression matrices and metadata can be joined and filtered with SQL.
when_to_use_negative:
- The dataset is small enough for in-memory analysis and an AnnData object with Scanpy
  is already sufficient — in-memory AnnSQL(adata=adata) or plain AnnData avoids the
  serialization cost
- 'The goal is a performance-optimized storage schema: the tables are explicitly designed
  to mirror AnnData structure, not for query performance'
- The matrix has more than 30,000 genes, where a known DuckDB memory leak issue is
  unresolved (TODO in the article's profiling code)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: AnnSQL
  role: Python package whose AnnSQL and MakeDb classes call BuildDb to construct the
    AnnData-inspired relational database and reopen it for SQL queries
  repo: https://github.com/ArpiarSaundersLab/annsql
- name: DuckDB
  role: In-process engine that stores the serialized AnnData layers as a relational
    database persisted to the .asql file
- name: AnnData
  role: Source single-cell object whose layers (X, obs, var, obsm, varm, obsp, uns)
    are converted into individual SQL tables
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:b33617397f30536092d524ab46e350ecf1e9c650fea2020bbcb3dadfcb73c359
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ArpiarSaundersLab/annsql
  tools:
  - AnnData
  - AnnSQL
  - DuckDB
  - Python
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# single-cell-matrix-serialization

## Summary

Serialize an AnnData single-cell object into a persistent relational DuckDB database (.asql) where each AnnData layer (X, obs, var, obsm, varm, obsp, uns) becomes an individual SQL table, enabling SQL queries on large single-cell datasets without converting the data back and forth.

## When to use

Use when a single-cell dataset stored as an AnnData (.h5ad) object or a Scanpy sample dataset must be converted into a persistent, SQL-queryable on-disk database so expression matrices and metadata can be joined and filtered with SQL.

## When NOT to use

- The dataset is small enough for in-memory analysis and an AnnData object with Scanpy is already sufficient — in-memory AnnSQL(adata=adata) or plain AnnData avoids the serialization cost
- The goal is a performance-optimized storage schema: the tables are explicitly designed to mirror AnnData structure, not for query performance
- The matrix has more than 30,000 genes, where a known DuckDB memory leak issue is unresolved (TODO in the article's profiling code)

## Inputs

- AnnData object in memory (e.g. sc.datasets.pbmc68k_reduced())
- h5ad file path (acceptable to the adata parameter of AnnSQL/MakeDb)
- Target database name and directory (e.g. db_name='pbmc3k_reduced', db_path='db/')

## Outputs

- Persistent .asql DuckDB database file (e.g. db/pbmc3k_reduced.asql)
- Per-layer SQL tables: X expression table with gene columns and cell_id key, obs/var property tables, obsm_X_pca / varm_PCs style tables, uns_raw table
- 'adata' convenience view joining the expression table with obs metadata via cell_id

## How to apply

Load the AnnData object (e.g. sc.datasets.pbmc3k() or sc.datasets.pbmc68k_reduced()) and call MakeDb(adata=adata, db_name='pbmc3k_reduced', db_path='db/'), which delegates to the BuildDb class to construct a relational DuckDB database where each AnnData layer becomes its own table. Convert the X layer into an expression table with one column per gene plus a cell_id key column; serialize obs and var metadata into individual property tables keyed by cell_id and gene id; write each obsm, varm, and obsp slot (e.g. X_pca, X_umap, PCs, distances, connectivities) into its own per-property table following naming conventions such as obsm_X_pca and varm_PCs; and serialize unstructured uns mappings into a serialized uns_raw table. Create the joined 'adata' convenience view combining the expression table with obs metadata via cell_id, and persist everything to disk as db/pbmc3k_reduced.asql. The database structure deliberately mirrors the AnnData object structure rather than being optimized for performance, which is the design rationale for one-table-per-layer. In-memory variants skip MakeDb and pass the AnnData object (or an h5ad path) directly to AnnSQL(adata=adata).

## Related tools

- **AnnSQL** (Python package whose AnnSQL and MakeDb classes call BuildDb to construct the AnnData-inspired relational database and reopen it for SQL queries) — https://github.com/ArpiarSaundersLab/annsql
- **DuckDB** (In-process engine that stores the serialized AnnData layers as a relational database persisted to the .asql file)
- **AnnData** (Source single-cell object whose layers (X, obs, var, obsm, varm, obsp, uns) are converted into individual SQL tables)

## Examples

```
from AnnSQL import AnnSQL; from AnnSQL.MakeDb import MakeDb; MakeDb(adata=adata, db_name="pbmc3k_reduced", db_path="db/"); asql = AnnSQL(db="db/pbmc3k_reduced.asql"); asql.query("SELECT * FROM adata LIMIT 5")
```

## Evaluation signals

- Reopening the persisted file with AnnSQL(db='db/pbmc3k_reduced.asql') succeeds and asql.query('SELECT * FROM adata LIMIT 5') returns rows whose dimensions match the source AnnData (n_obs x n_vars)
- The database contains one table per AnnData layer, with per-property tables named per convention (obsm_X_pca, varm_PCs) and a serialized uns_raw table
- The X expression table has exactly one column per gene plus a cell_id key column
- The 'adata' view joins the expression table with obs on cell_id and supports grouped queries such as SELECT obs.cell_type, AVG(gene_1) ... GROUP BY obs.cell_type
- The resulting .asql file can also be opened in R via the duckdb driver (dbConnect(duckdb::duckdb(), dbdir='db/pbmc3k.asql')) without conversion

## Limitations

- Database structure mirrors the AnnData object and is not optimized for performance
- Known unresolved DuckDB memory leak on databases with more than 30k genes
- In-memory AnnSQL is described as ideal only for smaller datasets; on-disk MakeDb serialization is intended for larger-than-memory data
- Memory-intensive whole-matrix aggregations (e.g. SUM(COLUMNS(*))) require the chunked calculate_gene_counts approach

## Evidence

- [intro] each layer stored as individual table: "AnnSQL converts the layers of an AnnData object into a relational [DuckDB](https://duckdb.org/) database. Each layer is stored as an individual table"
- [methods] structure mirrors AnnData not performance: "The database structure is not optimized for performance. Instead, the tables are designed to closely mirror the familiar structure of the AnnData object"
- [readme] MakeDb on-disk build: "MakeDb(adata=adata, db_name="pbmc3k_reduced", db_path="db/")"
- [readme] reopen and validate: "asql = AnnSQL(db="db/pbmc3k_reduced.asql")"
- [results] DuckDB memory leak caveat: "#TODO debug duckdb memory leak on daabases with >30k genes."
