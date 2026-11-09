---
name: large-scale-single-cell-backed-mode-data-loading
description: Use when an .h5ad single-cell dataset exceeds available memory (millions
  of cells) and must be opened with scanpy's backed mode before building an AnnSQL
  .asql database for SQL-based analysis.
when_to_use_negative:
- Dataset fits comfortably in memory — use the in-memory AnnSQL constructor AnnSQL(adata=adata)
  instead of the MakeDb on-disk path
- Input is not an AnnData/h5ad object or a layer already stored as a relational table
  — AnnSQL converts AnnData layers, each stored as an individual table, and is not
  a generic database import tool
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: AnnSQL
  role: Provides the MakeDb class that converts the backed-mode AnnData object into
    a persistent on-disk .asql DuckDB database, and the AnnSQL class to reopen and
    query it
  repo: https://github.com/ArpiarSaundersLab/annsql
- name: AnnData
  role: Backed-mode container for the oversized h5ad atlas dataset, read via scanpy's
    sc.read_h5ad so the 4.4-million-cell matrix is not fully loaded into memory
- name: DuckDB
  role: In-process relational engine underlying the .asql database produced by MakeDb;
    each AnnData layer is stored as an individual table
  repo: https://github.com/marcboeker/go-duckdb
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:bffa74c895d89645d5d8499a45a8e995390202c2cece4a9fa553c8e595e7f703
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
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# large-scale-single-cell-backed-mode-data-loading

## Summary

Loads very large single-cell AnnData objects (e.g. the 4.4-million-cell Macosko Mouse Atlas Single Nuclei h5ad) in backed mode so they can be converted into an on-disk AnnSQL/DuckDB database without exceeding available memory.

## When to use

Use when an .h5ad single-cell dataset exceeds available memory (millions of cells) and must be opened with scanpy's backed mode before building an AnnSQL .asql database for SQL-based analysis.

## When NOT to use

- Dataset fits comfortably in memory — use the in-memory AnnSQL constructor AnnSQL(adata=adata) instead of the MakeDb on-disk path
- Input is not an AnnData/h5ad object or a layer already stored as a relational table — AnnSQL converts AnnData layers, each stored as an individual table, and is not a generic database import tool

## Inputs

- Large .h5ad single-cell file (e.g. Macosko Mouse Atlas Single Nuclei, 4.4 million cells)
- Backed-mode AnnData object loaded via sc.read_h5ad

## Outputs

- On-disk AnnSQL database file (e.g. db/macosko_mouse_atlas.asql)
- Queryable DuckDB-backed AnnSQL instance returning Pandas DataFrames for SQL queries

## How to apply

Create a Python 3.12 environment (conda create -n annsql python=3.12) and pip install annsql. Open the h5ad file with scanpy's sc.read_h5ad in backed mode because the atlas-scale dataset (4.4 million cells) exceeds memory. Convert the backed AnnData into a persistent on-disk database with MakeDb(adata=adata, db_name="macosko_mouse_atlas", db_path="db/"), setting create_all_indexes=False as in the paper's atlas-scale benchmark code. If memory issues arise during the build, enable the make_buffer_file parameter of MakeDb, as recommended in the installation notes for high-memory builds. Validate by reopening the resulting db/macosko_mouse_atlas.asql with AnnSQL(db=...) and running a smoke-test query such as 'SELECT * FROM adata LIMIT 5' to confirm the database is queryable.

## Related tools

- **AnnSQL** (Provides the MakeDb class that converts the backed-mode AnnData object into a persistent on-disk .asql DuckDB database, and the AnnSQL class to reopen and query it) — https://github.com/ArpiarSaundersLab/annsql
- **AnnData** (Backed-mode container for the oversized h5ad atlas dataset, read via scanpy's sc.read_h5ad so the 4.4-million-cell matrix is not fully loaded into memory)
- **DuckDB** (In-process relational engine underlying the .asql database produced by MakeDb; each AnnData layer is stored as an individual table) — https://github.com/marcboeker/go-duckdb

## Examples

```
from AnnSQL.MakeDb import MakeDb; MakeDb(adata=adata, db_name="macosko_mouse_atlas", db_path="db/", create_all_indexes=False); asql = AnnSQL(db="db/macosko_mouse_atlas.asql"); asql.query("SELECT * FROM adata LIMIT 5")
```

## Evaluation signals

- AnnSQL(db="db/macosko_mouse_atlas.asql") opens without error, confirming the database artifact was written to the expected db/ path
- Smoke-test query 'SELECT * FROM adata LIMIT 5' returns rows, demonstrating the expression table is queryable
- Each AnnData layer is present as an individual table in the DuckDB schema, mirroring the AnnData object structure
- Cell count of the database matches the source dataset size (4.4 million cells in the atlas case)
- Build completes without memory exhaustion; if it did not, re-run with make_buffer_file=True per the installation notes

## Limitations

- Known unresolved DuckDB memory leak on databases with more than 30k genes (TODO in the paper's benchmark code)
- The database structure mirrors the AnnData object rather than being optimized for performance
- No software version numbers are reported for the tools used in the atlas-scale profiling section
- Backed mode is only needed because the dataset is larger than memory; for small datasets (e.g. pbmc68k_reduced) the in-memory AnnSQL path is recommended

## Evidence

- [results] Load atlas in backed mode: "#load the atlas dataset (4.4 million cells) in backed mode"
- [results] MakeDb builds on-disk database: "MakeDb(adata, db_path="../db/random/", db_name="random_data_"+str(i), create_all_indexes=False, convenience_view=False)"
- [methods] Conda env setup: "conda create -n annsql python=3.12"
- [readme] Layers become individual DuckDB tables: "AnnSQL converts the layers of an AnnData object into a relational [DuckDB](https://duckdb.org/) database. Each layer is stored as an individual table"
- [methods] Backed-mode runtime improvements: "Running AnnSQL locally for datasets that are larger than memory, that would typically require AnnData in backed mode see substantial runtime improvements"
