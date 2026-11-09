---
name: database-build-validation-querying
description: Use when an .h5ad single-cell dataset (e.g. a 4.4-million-cell Macosko
  Mouse Atlas single-nuclei file) is too large for memory-backed AnnData analysis
  and a queryable, persistent SQL database is needed.
when_to_use_negative:
- The dataset fits comfortably in memory and a small sample like sc.datasets.pbmc68k_reduced()
  is being explored — an in-memory AnnSQL(adata=adata) instance is the documented
  choice for smaller datasets
- The input is not an AnnData object or .h5ad file (AnnSQL converts AnnData layers;
  other formats are out of scope)
- The database must be performance-optimized rather than mirroring AnnData structure
  — the article states the table design intentionally mirrors AnnData and is not optimized
  for performance
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0622
tools:
- name: AnnSQL
  role: Python package providing MakeDb for building the on-disk .asql DuckDB database
    and the AnnSQL class for reopening and querying it
  repo: github.com/ArpiarSaundersLab/annsql
- name: DuckDB
  role: In-process relational engine underlying the .asql database; each AnnData layer
    is stored as an individual DuckDB table
- name: AnnData
  role: Source object format (.h5ad) whose layers are converted into the database;
    read in backed mode with scanpy for atlas-scale files
- name: Python
  role: Execution environment (Python 3.12 conda environment) for the build and validation
    workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:fa9124766b52177ad174ecb604e1f4c2075f069f23eb2847cf756c32c7566004
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
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# database-build-validation-querying

## Summary

Converts an AnnData object into a persistent on-disk AnnSQL (.asql) DuckDB-backed database using MakeDb, then validates the build by reopening the database and executing smoke-test SQL queries against the expression table.

## When to use

Use when an .h5ad single-cell dataset (e.g. a 4.4-million-cell Macosko Mouse Atlas single-nuclei file) is too large for memory-backed AnnData analysis and a queryable, persistent SQL database is needed.

## When NOT to use

- The dataset fits comfortably in memory and a small sample like sc.datasets.pbmc68k_reduced() is being explored — an in-memory AnnSQL(adata=adata) instance is the documented choice for smaller datasets
- The input is not an AnnData object or .h5ad file (AnnSQL converts AnnData layers; other formats are out of scope)
- The database must be performance-optimized rather than mirroring AnnData structure — the article states the table design intentionally mirrors AnnData and is not optimized for performance

## Inputs

- AnnData object or .h5ad file (e.g. Macosko Mouse Atlas Single Nuclei h5ad, 4.4 million cells, read in backed mode via scanpy)
- db_name and db_path parameters for the output database location

## Outputs

- Persistent on-disk AnnSQL .asql DuckDB database (e.g. db/macosko_mouse_atlas.asql)
- Query results as Pandas DataFrames (default), AnnData objects, or Parquet files

## How to apply

Load the .h5ad with scanpy's sc.read_h5ad in backed mode when the dataset exceeds available memory (e.g. the 4.4-million-cell Macosko Mouse Atlas Single Nuclei file). Set up a Python 3.12 conda environment and pip install annsql. Build the on-disk artifact with MakeDb(adata=adata, db_name='...', db_path='db/'), setting create_all_indexes=False for atlas-scale builds as in the paper's benchmark code; if memory issues arise during the build, enable the make_buffer_file parameter as recommended in the installation notes. The build converts each AnnData layer into an individual DuckDB table mirroring the AnnData structure. Validate by reopening with AnnSQL(db='db/<name>.asql') and running a smoke-test query such as 'SELECT * FROM adata LIMIT 5' to confirm the database is queryable.

## Related tools

- **AnnSQL** (Python package providing MakeDb for building the on-disk .asql DuckDB database and the AnnSQL class for reopening and querying it) — github.com/ArpiarSaundersLab/annsql
- **DuckDB** (In-process relational engine underlying the .asql database; each AnnData layer is stored as an individual DuckDB table)
- **AnnData** (Source object format (.h5ad) whose layers are converted into the database; read in backed mode with scanpy for atlas-scale files)
- **Python** (Execution environment (Python 3.12 conda environment) for the build and validation workflow)

## Examples

```
from AnnSQL import AnnSQL
from AnnSQL.MakeDb import MakeDb
MakeDb(adata=adata, db_name="macosko_mouse_atlas", db_path="db/", create_all_indexes=False)
asql = AnnSQL(db="db/macosko_mouse_atlas.asql"); asql.query("SELECT * FROM adata LIMIT 5")
```

## Evaluation signals

- Reopen the artifact with AnnSQL(db='db/<name>.asql') without errors, confirming the .asql file is valid
- A smoke-test query 'SELECT * FROM adata LIMIT 5' returns rows, proving the expression table is populated and queryable
- The DuckDB relational schema contains one table per AnnData layer (e.g. X and obs), mirroring the AnnData object structure
- Build completes within available memory; if it did not, retry with make_buffer_file enabled and confirm success
- create_all_indexes=False was passed for atlas-scale builds, matching the paper's benchmark construction code

## Limitations

- Known unresolved issue: DuckDB memory leak on databases with more than 30k genes
- Memory issues can arise during MakeDb builds of large atlases; the recommended mitigation is enabling the make_buffer_file parameter
- No software version numbers are reported for the tools in the results section, complicating exact reproduction
- The database structure is not optimized for performance; tables deliberately mirror the AnnData object structure

## Evidence

- [methods] Build on-disk AnnSQL database using MakeDb: "MakeDb(adata=adata, db_name="pbmc3k_reduced", db_path="db/")"
- [results] Create AnnSQL database from AnnData with MakeDb at atlas scale: "MakeDb(adata, db_path="../db/random/", db_name="random_data_"+str(i), create_all_indexes=False, convenience_view=False)"
- [results] Atlas dataset loaded in backed mode due to size: "#load the atlas dataset (4.4 million cells) in backed mode
adata_mem = sc.read_h5ad("
- [readme] On-disk database usage pattern from README: [evidence span withheld — anchor status: external_doc]
- [methods] Database mirrors AnnData structure: "The database structure is not optimized for performance. Instead, the tables are designed to closely mirror the familiar structure of the AnnData object"
