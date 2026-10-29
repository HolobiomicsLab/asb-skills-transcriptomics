---
name: on-disk-sql-database-construction
description: Use when a .h5ad single-cell dataset is too large for memory (e.g., 4.4M-cell
  atlas in backed mode) and needs a persistent, queryable SQL database for downstream
  runtime benchmarking or repeated SQL queries.
when_to_use_negative:
- Dataset fits comfortably in memory and only a few ad-hoc queries are needed — the
  in-memory AnnSQL(adata=adata) path is simpler and avoids build overhead
- You need a performance-optimized relational schema — the paper states the table
  structure mirrors AnnData 'rather than being optimized for performance'
- The AnnData object has more than ~30k genes — the paper notes a known unresolved
  DuckDB memory leak on such databases
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0203
tools:
- name: AnnSQL
  role: Provides the MakeDb class that converts AnnData layers into an on-disk DuckDB
    (.asql) database and the AnnSQL class to reopen it
  repo: https://github.com/ArpiarSaundersLab/annsql
- name: DuckDB
  role: In-process relational engine underlying the .asql database; each AnnData layer
    is stored as an individual table
- name: AnnData
  role: Source object format (backed-mode h5ad) converted into the SQL database
- name: Python
  role: Runtime environment (conda env with Python 3.12) in which annsql is pip installed
    and the build is executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:97c4c761cbd1915f7788ac2fd82df80f5cfdbdec1ec9c7f7c540281a24c1aa49
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ArpiarSaundersLab/annsql
  tools:
  - AnnData
  - AnnSQL
  - DuckDB
  - Python
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# on-disk-sql-database-construction

## Summary

Constructs a persistent on-disk AnnSQL (.asql) DuckDB database from an AnnData object using the MakeDb class, enabling SQL queries on single-cell datasets that exceed available memory (e.g., the 4.4-million-cell Macosko Mouse Atlas loaded in backed mode).

## When to use

Use when a .h5ad single-cell dataset is too large for memory (e.g., 4.4M-cell atlas in backed mode) and needs a persistent, queryable SQL database for downstream runtime benchmarking or repeated SQL queries.

## When NOT to use

- Dataset fits comfortably in memory and only a few ad-hoc queries are needed — the in-memory AnnSQL(adata=adata) path is simpler and avoids build overhead
- You need a performance-optimized relational schema — the paper states the table structure mirrors AnnData 'rather than being optimized for performance'
- The AnnData object has more than ~30k genes — the paper notes a known unresolved DuckDB memory leak on such databases

## Inputs

- AnnData object or .h5ad file (e.g., Macosko Mouse Atlas Single Nuclei, 4.4 million cells, loaded in backed mode)
- db_name and db_path parameters for the output database location

## Outputs

- Persistent on-disk AnnSQL .asql database (DuckDB relational database with AnnData layers as individual tables)
- Queryable X/obs tables validated via SQL smoke test

## How to apply

Create a Python 3.12 environment (conda create -n annsql python=3.12) and pip install annsql. Load the h5ad with scanpy's sc.read_h5ad in backed mode when the dataset exceeds memory (e.g., Macosko_Mouse_Atlas_Single_Nuclei with 4.4 million cells). Build the database with MakeDb(adata=adata, db_name=..., db_path='db/'); for atlas-scale builds set create_all_indexes=False as in the paper's benchmark code to skip index construction, and enable the make_buffer_file parameter if memory issues arise during the build. The AnnData layers are converted into a relational DuckDB database, each layer stored as an individual table mirroring AnnData structure rather than optimized for performance. Validate by reopening with AnnSQL(db='db/<name>.asql') and running a smoke-test query such as 'SELECT * FROM adata LIMIT 5'.

## Related tools

- **AnnSQL** (Provides the MakeDb class that converts AnnData layers into an on-disk DuckDB (.asql) database and the AnnSQL class to reopen it) — https://github.com/ArpiarSaundersLab/annsql
- **DuckDB** (In-process relational engine underlying the .asql database; each AnnData layer is stored as an individual table)
- **AnnData** (Source object format (backed-mode h5ad) converted into the SQL database)
- **Python** (Runtime environment (conda env with Python 3.12) in which annsql is pip installed and the build is executed)

## Examples

```
from AnnSQL import AnnSQL
from AnnSQL.MakeDb import MakeDb
adata = sc.read_h5ad("Macosko_Mouse_Atlas_Single_Nuclei.h5ad", backed="r"); MakeDb(adata=adata, db_name="macosko_mouse_atlas", db_path="db/", create_all_indexes=False); asql = AnnSQL(db="db/macosko_mouse_atlas.asql"); asql.query("SELECT * FROM adata LIMIT 5")
```

## Evaluation signals

- The .asql file exists at db_path/db_name.asql after MakeDb completes without memory errors
- AnnSQL(db='db/<name>.asql') opens and 'SELECT * FROM adata LIMIT 5' returns rows (smoke-test queryability)
- For atlas-scale builds, create_all_indexes=False was passed, matching the paper's benchmark code
- Table schema mirrors AnnData structure (X and obs as separate tables with cell_id keys)
- If the build exceeded memory, the make_buffer_file parameter was enabled per installation notes

## Limitations

- Database structure is not optimized for performance; tables deliberately mirror the AnnData object structure
- Known unresolved DuckDB memory leak on databases with more than 30k genes (TODO in the paper's code)
- Very large h5ad files must be loaded in backed mode; no software version numbers reported in the paper's results section

## Evidence

- [results] Backed-mode AnnData to on-disk MakeDb build: "#load the atlas dataset (4.4 million cells) in backed mode"
- [results] MakeDb call with create_all_indexes=False: "MakeDb(adata, db_path="../db/random/", db_name="random_data_"+str(i), create_all_indexes=False, convenience_view=False)"
- [intro] Layers converted into relational DuckDB database: "AnnSQL converts the layers of an AnnData object into a relational [DuckDB](https://duckdb.org/) database. Each layer is stored as an individual table"
- [readme] README on-disk usage pattern: "MakeDb(adata=adata, db_name="pbmc3k_reduced", db_path="db/")"
- [methods] Schema mirrors AnnData, not optimized: "The database structure is not optimized for performance. Instead, the tables are designed to closely mirror the familiar structure of the AnnData object"
