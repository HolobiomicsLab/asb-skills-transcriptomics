---
name: duckdb-relational-schema-design
description: Use when converting an AnnData (.h5ad) single-cell object into an on-disk
  relational DuckDB database and you need the layer-to-table mapping, key columns,
  and naming conventions for each AnnData slot.
when_to_use_negative:
- The goal is a performance-optimized analytical schema — the article explicitly states
  the database structure is not optimized for performance and instead mirrors the
  AnnData object
- The data is not an AnnData single-cell object (e.g. a generic feature table or tidy
  CSV) — the layer-to-table mapping depends on AnnData slots X, obs, var, obsm, varm,
  obsp, uns
- An in-memory workflow suffices for small data — AnnSQL(adata=adata) avoids the on-disk
  MakeDb construction entirely
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: AnnSQL
  role: Python package whose AnnSQL and MakeDb classes call BuildDb to construct the
    relational database from AnnData layers
  repo: github.com/ArpiarSaundersLab/annsql
- name: DuckDB
  role: In-process engine that stores the relational database; each AnnData layer
    becomes an individual DuckDB table
- name: AnnData
  role: Source object whose layers (X, obs, var, obsm, varm, obsp, uns) define the
    schema of the converted tables
- name: Python
  role: Language in which the AnnSQL/MakeDb/BuildDb construction pipeline is implemented
    and invoked
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:67a1b6c25a374786082f93f6de1b30444277a9b92541a9b1b0f1c66382be4855
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AnnData
  - AnnSQL
  - DuckDB
  - Python
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# duckdb-relational-schema-design

## Summary

Design a DuckDB relational schema that mirrors the layer structure of an AnnData single-cell object, storing each layer (X, obs, var, obsm, varm, obsp, uns) as an individual table keyed by cell_id/gene id, plus an 'adata' convenience view joining expression with obs metadata.

## When to use

Use when converting an AnnData (.h5ad) single-cell object into an on-disk relational DuckDB database and you need the layer-to-table mapping, key columns, and naming conventions for each AnnData slot.

## When NOT to use

- The goal is a performance-optimized analytical schema — the article explicitly states the database structure is not optimized for performance and instead mirrors the AnnData object
- The data is not an AnnData single-cell object (e.g. a generic feature table or tidy CSV) — the layer-to-table mapping depends on AnnData slots X, obs, var, obsm, varm, obsp, uns
- An in-memory workflow suffices for small data — AnnSQL(adata=adata) avoids the on-disk MakeDb construction entirely

## Inputs

- AnnData object (in-memory or .h5ad file path) with X expression matrix and obs/var metadata
- Scanpy sample datasets such as pbmc3k or pbmc68k_reduced for testing the conversion
- db_name and db_path parameters for the on-disk database location

## Outputs

- Persistent on-disk DuckDB database file (e.g. db/pbmc3k_reduced.asql) with one table per AnnData layer
- Expression table X with one column per gene plus cell_id key column
- Per-property tables named obsm_X_pca, varm_PCs, etc. for obsm/varm/obsp slots
- Serialized uns_raw table for unstructured annotations
- 'adata' convenience view joining expression table with obs metadata via cell_id

## How to apply

Load a small Scanpy sample AnnData object (e.g. sc.datasets.pbmc3k() or sc.datasets.pbmc68k_reduced()) and call MakeDb(adata=..., db_name=..., db_path='db/') to trigger the BuildDb class, which AnnSQL and MakeDb both delegate database construction to. Convert the X layer into an expression table with one column per gene plus a cell_id key column; serialize obs and var metadata into individual property tables keyed by cell_id/gene id. Write each obsm, varm, and obsp slot (e.g. X_pca, X_umap, PCs, distances, connectivities) into its own per-property table following the naming convention obsm_X_pca, varm_PCs, etc., and serialize unstructured uns mappings into a serialized uns_raw table. The schema deliberately mirrors the AnnData object structure rather than being optimized for performance. Finally create the joined 'adata' convenience view combining the expression table with obs via cell_id, and persist the database as a .asql file.

## Related tools

- **AnnSQL** (Python package whose AnnSQL and MakeDb classes call BuildDb to construct the relational database from AnnData layers) — github.com/ArpiarSaundersLab/annsql
- **DuckDB** (In-process engine that stores the relational database; each AnnData layer becomes an individual DuckDB table)
- **AnnData** (Source object whose layers (X, obs, var, obsm, varm, obsp, uns) define the schema of the converted tables)
- **Python** (Language in which the AnnSQL/MakeDb/BuildDb construction pipeline is implemented and invoked)

## Examples

```
from AnnSQL.MakeDb import MakeDb; MakeDb(adata=sc.datasets.pbmc68k_reduced(), db_name='pbmc3k_reduced', db_path='db/'); asql = AnnSQL(db='db/pbmc3k_reduced.asql'); asql.query('SELECT * FROM adata LIMIT 5')
```

## Evaluation signals

- Reopen the persisted database with AnnSQL(db='db/pbmc3k_reduced.asql') and run asql.query('SELECT * FROM adata LIMIT 5'); returned rows must match the source AnnData dimensions (n_obs x n_vars)
- Verify the expression table X contains exactly one column per gene plus the cell_id key column
- Check that each obsm/varm/obsp slot appears as its own table under the naming convention obsm_X_pca, varm_PCs, etc.
- Confirm the uns_raw table exists and holds the serialized unstructured annotations
- Confirm the 'adata' convenience view joins the expression table with obs metadata on cell_id

## Limitations

- The database structure is not optimized for performance; tables are designed to closely mirror the AnnData object structure instead
- Known unresolved issue: DuckDB memory leak on databases with more than 30k genes (noted as a TODO in the benchmark code)
- No software version numbers are reported for the tools used in the construction workflow

## Evidence

- [intro] AnnSQL converts the layers of an AnnData object into a relational DuckDB database: "AnnSQL converts the layers of an AnnData object into a relational [DuckDB](https://duckdb.org/) database"
- [readme] each layer stored as individual table: "Each layer is stored as an individual table, allowing for simple or complex SQL queries, including table joins."
- [methods] schema mirrors AnnData not performance: "The database structure is not optimized for performance. Instead, the tables are designed to closely mirror the familiar structure of the AnnData object"
- [methods] MakeDb on-disk construction: "MakeDb(adata=adata, db_name="pbmc3k_reduced", db_path="db/")"
- [methods] reopen and validate on-disk db: "asql = AnnSQL(db="db/pbmc3k_reduced.asql")"
- [results] memory leak limitation: "#TODO debug duckdb memory leak on daabases with >30k genes."
