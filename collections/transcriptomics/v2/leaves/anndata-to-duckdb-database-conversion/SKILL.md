---
name: anndata-to-duckdb-database-conversion
description: Use when a single-cell AnnData/h5ad dataset is too large for memory (e.g.
  millions of cells) and SQL-based querying, filtering, or aggregation is needed instead
  of AnnData-backed-mode operations that fail or run ~700x slower.
when_to_use_negative:
- Dataset fits comfortably in memory and simple in-memory use suffices — use AnnSQL(adata=adata)
  directly instead of MakeDb
- Input is not an AnnData/h5ad object (e.g. raw count matrices or other feature tables)
  — the database structure mirrors AnnData layers specifically
- Analysis requires a performance-optimized database schema — the article notes 'The
  database structure is not optimized for performance'
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0203
tools:
- name: AnnSQL
  role: Python package providing the MakeDb class that converts AnnData layers into
    relational DuckDB tables and the AnnSQL class that opens/queries the resulting
    .asql database
  repo: github.com/ArpiarSaundersLab/annsql
- name: DuckDB
  role: In-process relational database engine underlying the .asql artifact; each
    AnnData layer is stored as an individual table enabling simple or complex SQL
    queries including joins
- name: AnnData
  role: Source data container (.h5ad) loaded in backed mode via scanpy's sc.read_h5ad
    when the dataset exceeds available memory
- name: Scanpy
  role: Used to read the h5ad file in backed mode before conversion (sc.read_h5ad)
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:0202bf43843f1a214763ab4f0620e72355b0ea07ef7de44e4a3c68e0c56ce5a3
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# anndata-to-duckdb-database-conversion

## Summary

Converts AnnData single-cell objects (.h5ad) into persistent on-disk AnnSQL databases (.asql) backed by an in-process DuckDB engine, where each AnnData layer becomes an individual table. This enables SQL queries on datasets that exceed available memory, such as the 4.4-million-cell Macosko Mouse Atlas.

## When to use

Use when a single-cell AnnData/h5ad dataset is too large for memory (e.g. millions of cells) and SQL-based querying, filtering, or aggregation is needed instead of AnnData-backed-mode operations that fail or run ~700x slower.

## When NOT to use

- Dataset fits comfortably in memory and simple in-memory use suffices — use AnnSQL(adata=adata) directly instead of MakeDb
- Input is not an AnnData/h5ad object (e.g. raw count matrices or other feature tables) — the database structure mirrors AnnData layers specifically
- Analysis requires a performance-optimized database schema — the article notes 'The database structure is not optimized for performance'

## Inputs

- AnnData object or .h5ad file (e.g. Macosko Mouse Atlas Single Nuclei h5ad, 4.4 million cells; or scanpy sample datasets like pbmc68k_reduced)
- Target database name and directory path (e.g. db_name='macosko_mouse_atlas', db_path='db/')

## Outputs

- Persistent on-disk AnnSQL database file (.asql) containing DuckDB tables mirroring AnnData layers
- Queryable AnnSQL handle returning Pandas DataFrames, AnnData objects, or Parquet files

## How to apply

Create a Python 3.12 conda environment and install the package with 'pip install annsql'. Load the .h5ad file with scanpy's sc.read_h5ad in backed mode when the dataset exceeds memory (e.g. the 4.4-million-cell Macosko Mouse Atlas Single Nuclei file). Build the on-disk database by calling MakeDb(adata=adata, db_name=..., db_path="db/"), setting create_all_indexes=False for atlas-scale builds as shown in the paper's benchmark code; if memory issues arise during the build, enable the make_buffer_file parameter of MakeDb as recommended in the installation notes. The resulting .asql artifact stores each AnnData layer (e.g. X, obs) as an individual DuckDB table mirroring the AnnData structure rather than being performance-optimized. Validate by reopening with AnnSQL(db="db/<name>.asql") and running a smoke-test query such as 'SELECT * FROM adata LIMIT 5'; optionally save the raw layer via asql.save_raw() before preprocessing steps.

## Related tools

- **AnnSQL** (Python package providing the MakeDb class that converts AnnData layers into relational DuckDB tables and the AnnSQL class that opens/queries the resulting .asql database) — github.com/ArpiarSaundersLab/annsql
- **DuckDB** (In-process relational database engine underlying the .asql artifact; each AnnData layer is stored as an individual table enabling simple or complex SQL queries including joins)
- **AnnData** (Source data container (.h5ad) loaded in backed mode via scanpy's sc.read_h5ad when the dataset exceeds available memory)
- **Scanpy** (Used to read the h5ad file in backed mode before conversion (sc.read_h5ad))

## Examples

```
from AnnSQL.MakeDb import MakeDb; MakeDb(adata=adata, db_name="macosko_mouse_atlas", db_path="db/", create_all_indexes=False); asql = AnnSQL(db="db/macosko_mouse_atlas.asql"); asql.query("SELECT * FROM adata LIMIT 5")
```

## Evaluation signals

- The MakeDb build completes and produces the expected .asql file at db_path (e.g. db/macosko_mouse_atlas.asql)
- Reopening with AnnSQL(db='db/<name>.asql') and executing 'SELECT * FROM adata LIMIT 5' returns rows without error
- Table structure mirrors the AnnData object — each layer (X, raw, obs) is present as an individual table with cell_id keys
- If create_all_indexes=False was used for atlas-scale builds, the artifact matches the paper's benchmark configuration
- For high-memory builds, the make_buffer_file parameter was enabled if the build hit memory issues, per installation notes

## Limitations

- The database structure mirrors the AnnData object and 'is not optimized for performance' — queries may be slower than a purpose-built schema
- Known unresolved issue: DuckDB memory leak on databases with more than 30k genes (noted as a TODO in the paper's code)
- Building very large databases (e.g. 4.4 million cells) can raise memory issues; enable the make_buffer_file parameter of MakeDb to mitigate
- create_all_indexes=False should be set for atlas-scale builds as in the paper's benchmark code; indexed builds were not benchmarked at that scale

## Evidence

- [intro] AnnSQL converts the layers of an AnnData object into a relational DuckDB database. Each layer is stored as an individual table: "AnnSQL converts the layers of an AnnData object into a relational [DuckDB](https://duckdb.org/) database. Each layer is stored as an individual table"
- [methods] MakeDb(adata=adata, db_name="pbmc3k_reduced", db_path="db/"): "MakeDb(adata=adata, db_name="pbmc3k_reduced", db_path="db/")"
- [results] load the atlas dataset (4.4 million cells) in backed mode: "#load the atlas dataset (4.4 million cells) in backed mode"
- [methods] The database structure is not optimized for performance. Instead, the tables are designed to closely mirror the familiar structure of the AnnData object: "The database structure is not optimized for performance. Instead, the tables are designed to closely mirror the familiar structure of the AnnData object"
- [readme] For larger datasets, AnnSQL can create a local database (asql) from the AnnData object. This database is stored on-disk, can be queried, and is persistent.: [evidence span withheld — anchor status: external_doc]
