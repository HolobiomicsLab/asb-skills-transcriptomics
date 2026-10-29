---
name: unstructured-annotation-serialization
description: Use when a Scanpy AnnData object (.h5ad or in-memory) needs to be persisted
  as a queryable relational DuckDB database, including unstructured uns mappings that
  have no natural table form.
when_to_use_negative:
- The dataset is small enough to hold in memory and only transient queries are needed
  — an in-memory AnnSQL(adata=adata) instance avoids building the on-disk table structure
- The input is not an AnnData object (e.g. raw FASTQ or a flat CSV matrix) — MakeDb/BuildDb
  expects AnnData slots (X, obs, var, obsm, varm, obsp, uns) to serialize
- A performance-optimized relational schema is required for heavy production workloads
  — the AnnSQL schema explicitly mirrors AnnData structure instead of optimizing for
  performance
tools:
- name: AnnSQL
  role: Python package whose MakeDb/BuildDb classes construct the AnnData-inspired
    DuckDB database and reopen the persisted .asql file
  repo: github.com/ArpiarSaundersLab/annsql
- name: DuckDB
  role: In-process relational engine that stores each AnnData layer as an individual
    table in the .asql database file
  repo: https://duckdb.org/
- name: AnnData
  role: Source object whose X, obs, var, obsm, varm, obsp, and uns slots define the
    table structure to be serialized
  repo: https://anndata.readthedocs.io/en/latest/
- name: Scanpy
  role: Provides the sample pbmc3k and pbmc68k_reduced datasets used as conversion
    inputs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:9add31af4816efd5adbf5c6e72d2c9549249e68720585d4ba550cc91e5377e32
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AnnData
  - AnnSQL
  - DuckDB
  - Python
  - Scanpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# unstructured-annotation-serialization

## Summary

Serialize the full slot structure of a single-cell AnnData object — X expression matrix, obs/var metadata, obsm/varm/obsp mappings, and unstructured uns annotations — into an AnnSQL/DuckDB relational database (.asql) where each slot becomes an individual SQL table mirroring the AnnData object structure.

## When to use

Use when a Scanpy AnnData object (.h5ad or in-memory) needs to be persisted as a queryable relational DuckDB database, including unstructured uns mappings that have no natural table form.

## When NOT to use

- The dataset is small enough to hold in memory and only transient queries are needed — an in-memory AnnSQL(adata=adata) instance avoids building the on-disk table structure
- The input is not an AnnData object (e.g. raw FASTQ or a flat CSV matrix) — MakeDb/BuildDb expects AnnData slots (X, obs, var, obsm, varm, obsp, uns) to serialize
- A performance-optimized relational schema is required for heavy production workloads — the AnnSQL schema explicitly mirrors AnnData structure instead of optimizing for performance

## Inputs

- AnnData object (in-memory from sc.datasets.pbmc68k_reduced() or read from .h5ad)
- Database name and output directory for the .asql file
- Optional MakeDb flags such as create_all_indexes and convenience_view

## Outputs

- Persistent on-disk AnnSQL database file (e.g. db/pbmc3k_reduced.asql)
- X expression table (one column per gene + cell_id key)
- obs and var property tables keyed by cell_id / gene id
- Per-slot tables following obsm_X_pca, varm_PCs style naming
- Serialized uns_raw table for unstructured mappings
- 'adata' convenience view joining X with obs via cell_id

## How to apply

Load a Scanpy sample AnnData (e.g. sc.datasets.pbmc3k() or sc.datasets.pbmc68k_reduced()) as input and call MakeDb(adata=..., db_name=..., db_path='db/') to trigger the BuildDb construction path. Convert the X layer into an expression table with one column per gene plus a cell_id key column; serialize obs and var metadata into individual property tables keyed by cell_id/gene id. Write each obsm, varm, and obsp slot (e.g. X_pca, X_umap, PCs, distances, connectivities) into its own per-property table following the naming convention obsm_X_pca, varm_PCs, etc., and serialize the unstructured uns mappings into a serialized uns_raw table. The rationale is fidelity over speed: tables are deliberately designed to mirror the AnnData object structure rather than being performance-optimized. Finally create the joined 'adata' convenience view combining the expression table with obs metadata via cell_id, and persist to disk as an .asql file.

## Related tools

- **AnnSQL** (Python package whose MakeDb/BuildDb classes construct the AnnData-inspired DuckDB database and reopen the persisted .asql file) — github.com/ArpiarSaundersLab/annsql
- **DuckDB** (In-process relational engine that stores each AnnData layer as an individual table in the .asql database file) — https://duckdb.org/
- **AnnData** (Source object whose X, obs, var, obsm, varm, obsp, and uns slots define the table structure to be serialized) — https://anndata.readthedocs.io/en/latest/
- **Scanpy** (Provides the sample pbmc3k and pbmc68k_reduced datasets used as conversion inputs)

## Examples

```
from AnnSQL.MakeDb import MakeDb; MakeDb(adata=sc.datasets.pbmc68k_reduced(), db_name="pbmc3k_reduced", db_path="db/"); asql = AnnSQL(db="db/pbmc3k_reduced.asql")
```

## Evaluation signals

- Reopen the database with AnnSQL(db='db/pbmc3k_reduced.asql') and confirm asql.query('SELECT * FROM adata LIMIT 5') returns rows matching the source AnnData dimensions (n_obs x n_vars)
- Verify each slot produced its own table with expected names (X, obs, var, obsm_X_pca, varm_PCs, uns_raw) rather than merged or dropped slots
- Check the obs/var property tables are keyed by cell_id / gene id and that the 'adata' view joins X to obs on cell_id
- Confirm the uns_raw table round-trips the unstructured uns mappings from the source AnnData object
- Compare per-gene and per-cell counts in the X table against the original AnnData X matrix

## Limitations

- The database structure is not optimized for performance; tables are designed to closely mirror the AnnData object structure instead
- Known unresolved DuckDB memory leak on databases with more than 30k genes
- Summing all gene counts via raw SQL (SELECT SUM(COLUMNS(*))) is memory intensive; the chunked calculate_gene_counts method is recommended

## Evidence

- [methods] Tables mirror AnnData structure: "The database structure is not optimized for performance. Instead, the tables are designed to closely mirror the familiar structure of the AnnData object"
- [readme] Each layer stored as individual table: "Each layer is stored as an individual table, allowing for simple or complex SQL queries, including table joins."
- [readme] On-disk database build and reopen workflow: "MakeDb(adata=adata, db_name="pbmc3k_reduced", db_path="db/")

#open the AnnSQL database
asql = AnnSQL(db="db/pbmc3k_reduced.asql")"
- [intro] Converts layers into relational DuckDB database: "AnnSQL converts the layers of an AnnData object into a relational [DuckDB](https://duckdb.org/) database"
