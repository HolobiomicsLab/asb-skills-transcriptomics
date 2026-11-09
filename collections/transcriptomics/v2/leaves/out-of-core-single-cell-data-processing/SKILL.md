---
name: out-of-core-single-cell-data-processing
description: Use when a single-cell dataset (e.g., the 4.4-million-cell Macosko Mouse
  Atlas h5ad) exceeds available memory, making AnnData/Seurat operations fail or run
  ~700x slower, and SQL-based filtering/aggregation is needed.
when_to_use_negative:
- The dataset fits comfortably in memory and a simpler in-memory AnnSQL(adata=adata)
  build suffices
- The workflow requires arbitrary AnnData/scanpy methods (e.g., scanpy PCA pipelines)
  rather than SQL queries or AnnSQL's extended methods
- The database exceeds ~30k genes, where a known DuckDB memory leak remains unresolved
  (TODO in the paper's code)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: AnnSQL
  role: Provides MakeDb to build the on-disk .asql database from a backed-mode AnnData
    object and the AnnSQL class to reopen and query it
  repo: https://github.com/ArpiarSaundersLab/annsql
- name: DuckDB
  role: In-process relational engine underlying the .asql database; each AnnData layer
    is stored as an individual table
- name: AnnData
  role: Source object format (.h5ad) read in backed mode because the dataset exceeds
    memory
- name: Scanpy
  role: Used (sc.read_h5ad) to load the oversized h5ad in backed mode prior to database
    construction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:fc9449e30a3b3783a8b48c791181dca7d8648c8570b91f0d72a49e891baae650
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ArpiarSaundersLab/annsql
  tools:
  - AnnData
  - AnnSQL
  - DuckDB
  - Python
  - Scanpy
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.65
n_claims: 20
---

# out-of-core-single-cell-data-processing

## Summary

Process single-cell RNA-seq datasets that exceed available memory by converting a backed-mode AnnData (.h5ad) object into a persistent on-disk AnnSQL database (DuckDB engine) and querying it with SQL instead of loading the full matrix into RAM.

## When to use

Use when a single-cell dataset (e.g., the 4.4-million-cell Macosko Mouse Atlas h5ad) exceeds available memory, making AnnData/Seurat operations fail or run ~700x slower, and SQL-based filtering/aggregation is needed.

## When NOT to use

- The dataset fits comfortably in memory and a simpler in-memory AnnSQL(adata=adata) build suffices
- The workflow requires arbitrary AnnData/scanpy methods (e.g., scanpy PCA pipelines) rather than SQL queries or AnnSQL's extended methods
- The database exceeds ~30k genes, where a known DuckDB memory leak remains unresolved (TODO in the paper's code)

## Inputs

- AnnData .h5ad file larger than available memory (e.g., Macosko Mouse Atlas Single Nuclei, 4.4 million cells), opened in backed mode
- Scanpy sample datasets (pbmc3k, pbmc68k_reduced) for smaller-scale builds

## Outputs

- Persistent on-disk AnnSQL database (.asql file, e.g., db/macosko_mouse_atlas.asql) backed by DuckDB
- Query results returned as Pandas DataFrames, AnnData objects, or Parquet files

## How to apply

Create a Python 3.12 environment (conda create -n annsql python=3.12) and pip install annsql. Load the oversized .h5ad with scanpy's sc.read_h5ad in backed mode since the matrix cannot fit in memory. Build a persistent on-disk database with MakeDb(adata=adata, db_name=..., db_path='db/'), passing create_all_indexes=False for atlas-scale benchmark builds (as done in the paper's results code) and enabling make_buffer_file if the build itself hits memory limits. Reopen the artifact with AnnSQL(db='db/<name>.asql') and run SQL queries (e.g., 'SELECT * FROM adata LIMIT 5') to verify it is queryable. The rationale is that AnnSQL converts AnnData layers into individual relational DuckDB tables, letting an in-process DuckDB engine scan expression matrices on disk in minutes on a laptop where in-memory AnnData or Seurat failed on an HPC.

## Related tools

- **AnnSQL** (Provides MakeDb to build the on-disk .asql database from a backed-mode AnnData object and the AnnSQL class to reopen and query it) — https://github.com/ArpiarSaundersLab/annsql
- **DuckDB** (In-process relational engine underlying the .asql database; each AnnData layer is stored as an individual table)
- **AnnData** (Source object format (.h5ad) read in backed mode because the dataset exceeds memory)
- **Scanpy** (Used (sc.read_h5ad) to load the oversized h5ad in backed mode prior to database construction)

## Examples

```
from AnnSQL.MakeDb import MakeDb; MakeDb(adata=adata, db_name="macosko_mouse_atlas", db_path="db/", create_all_indexes=False); asql = AnnSQL(db="db/macosko_mouse_atlas.asql"); asql.query("SELECT * FROM adata LIMIT 5")
```

## Evaluation signals

- Reopen the produced .asql with AnnSQL(db='db/<name>.asql') and confirm a smoke-test query ('SELECT * FROM adata LIMIT 5') returns rows
- Confirm each AnnData layer exists as an individual table in the DuckDB database (schema mirrors AnnData structure rather than a performance-optimized layout)
- Verify the .asql file persists at db_path with the requested db_name (e.g., db/macosko_mouse_atlas.asql)
- Check that the build used create_all_indexes=False for atlas-scale data, matching the paper's benchmark code
- For high-memory builds, confirm make_buffer_file was enabled if memory issues occurred during MakeDb

## Limitations

- Known unresolved DuckDB memory leak on databases with more than 30k genes (TODO in the paper's profiling code)
- AnnSQL's database structure is not optimized for performance; tables mirror the AnnData object structure instead
- No software version numbers are reported for the tools in the profiling code, complicating exact reproduction
- Equivalent operations in AnnData or Seurat largely failed or were ~700x slower on an HPC cluster, so side-by-side in-memory comparisons are infeasible at atlas scale

## Evidence

- [results] on-disk build via MakeDb: "MakeDb(adata, db_path="../db/random/", db_name="random_data_"+str(i), create_all_indexes=False, convenience_view=False)"
- [results] backed-mode load of atlas dataset: "#load the atlas dataset (4.4 million cells) in backed mode
adata_mem = sc.read_h5ad("/home/exacloud/gscratch/SaundersLab/24-09_CellTypist/chunked_approach/Macosko_Mouse_Atlas_Single_Nuclei"
- [readme] README on-disk usage: [evidence span withheld — anchor status: external_doc]
- [abstract] abstract performance claim: "AnnSQL-based operations were executed in minutes on a laptop for which equivalent operations in AnnData or Seurat largely failed (or were ∼700× slower)"
- [methods] database design rationale: "The database structure is not optimized for performance. Instead, the tables are designed to closely mirror the familiar structure of the AnnData object"
