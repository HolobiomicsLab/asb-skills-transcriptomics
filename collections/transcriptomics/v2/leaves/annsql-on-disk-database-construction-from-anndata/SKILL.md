---
name: annsql-on-disk-database-construction-from-anndata
description: Use when an .h5ad single-cell dataset is too large for memory and AnnData
  backed-mode reads are slow — e.g. filtering or aggregate queries on hundreds of
  thousands of cells — and persistent, SQL-queryable storage is needed.
when_to_use_negative:
- 'The dataset is small (<5,000 cells with ~10,000 genes): on-disk AnnData was faster
  than on-disk AnnSQL at that scale, so in-memory AnnData is simpler'
- The database schema must be performance-optimized rather than mirroring AnnData
  structure — the authors explicitly state tables are not optimized for performance
- The dataset has more than ~30,000 genes, given the known unresolved DuckDB memory
  leak on such databases
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: AnnSQL
  role: Python package that builds and queries the on-disk DuckDB (.asql) database
    from AnnData; provides MakeDb, AnnSQL.query, save_raw, calculate_total_counts,
    calculate_gene_counts
  repo: https://github.com/ArpiarSaundersLab/annsql
- name: DuckDB
  role: In-process engine that stores each AnnData layer as a relational table and
    executes the SQL queries
- name: AnnData
  role: Source single-cell object (.h5ad) whose layers are converted into the database
- name: scanpy
  role: Loads sample and user .h5ad datasets (sc.read_h5ad, sc.datasets.pbmc68k_reduced)
    before conversion
- name: Pandas
  role: Default return type for AnnSQL query results
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:9e7eae94684828822f38762da538415d235a57b679fbab51f7392bc9c2b05316
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
  - matplotlib
  - memory_profiler
  - scanpy
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6818181818181818
n_claims: 22
---

# annsql-on-disk-database-construction-from-anndata

## Summary

Build a persistent on-disk AnnSQL (.asql) DuckDB database from an AnnData object so that large single-cell datasets can be queried with SQL without fitting the full expression matrix in memory. AnnSQL mirrors the AnnData structure (each layer stored as an individual table) rather than optimizing the schema for performance.

## When to use

Use when an .h5ad single-cell dataset is too large for memory and AnnData backed-mode reads are slow — e.g. filtering or aggregate queries on hundreds of thousands of cells — and persistent, SQL-queryable storage is needed.

## When NOT to use

- The dataset is small (<5,000 cells with ~10,000 genes): on-disk AnnData was faster than on-disk AnnSQL at that scale, so in-memory AnnData is simpler
- The database schema must be performance-optimized rather than mirroring AnnData structure — the authors explicitly state tables are not optimized for performance
- The dataset has more than ~30,000 genes, given the known unresolved DuckDB memory leak on such databases

## Inputs

- AnnData object or .h5ad file (single-cell expression matrix with layers, X, obs)
- db_path directory and db_name for the on-disk database
- Optional MakeDb options: create_all_indexes (bool), convenience_view (bool)

## Outputs

- Persistent on-disk AnnSQL database file (.asql) backed by DuckDB
- SQL query results returned as a Pandas DataFrame, AnnData object, or Parquet file
- Optional precomputed columns: total_counts, gene_counts, raw matrix layer

## How to apply

Create a Python 3.12 environment and pip install annsql. Load the .h5ad into AnnData (e.g. via scanpy's sc.read_h5ad or sc.datasets.pbmc68k_reduced) and construct the database with MakeDb(adata=adata, db_name=..., db_path=...), optionally passing create_all_indexes=False and convenience_view=False as done in the benchmark scripts. Open the resulting .asql file with AnnSQL(db='db/name.asql') and issue SQL against the X and obs tables; optionally call asql.save_raw() to persist a raw layer and asql.calculate_total_counts()/calculate_gene_counts() with chunk_size (e.g. 750) to precompute counts. The database file is persistent and can also be opened in R via the DuckDB driver for dbGetQuery. Rationale: keeping the structure aligned with AnnData (one table per layer, cell_id join key) makes queries intuitive, while the in-process DuckDB engine delivers the speedup.

## Related tools

- **AnnSQL** (Python package that builds and queries the on-disk DuckDB (.asql) database from AnnData; provides MakeDb, AnnSQL.query, save_raw, calculate_total_counts, calculate_gene_counts) — https://github.com/ArpiarSaundersLab/annsql
- **DuckDB** (In-process engine that stores each AnnData layer as a relational table and executes the SQL queries)
- **AnnData** (Source single-cell object (.h5ad) whose layers are converted into the database)
- **scanpy** (Loads sample and user .h5ad datasets (sc.read_h5ad, sc.datasets.pbmc68k_reduced) before conversion)
- **Pandas** (Default return type for AnnSQL query results)

## Examples

```
from AnnSQL.MakeDb import MakeDb; MakeDb(adata=adata, db_name='pbmc3k_reduced', db_path='db/'); asql = AnnSQL(db='db/pbmc3k_reduced.asql'); asql.query('SELECT * FROM adata LIMIT 5')
```

## Evaluation signals

- AnnSQL(db='db/<name>.asql') opens the file without error and 'SELECT * FROM adata LIMIT 5' returns rows matching the source AnnData
- obs-level filters (e.g. filter_by_cell_counts(min_cell_count=2500, max_cell_count=40000)) return cell counts consistent with equivalent AnnData subsetting
- Join queries ('SELECT obs.cell_type, AVG(gene_1) FROM X INNER JOIN obs ON X.cell_id = obs.cell_id GROUP BY obs.cell_type') execute and match expected per-group averages
- For the 250,000-cell benchmark dataset, the SQL filter (SELECT gene_1 FROM X WHERE gene_1 > 0.5) completes in ~0.14 s versus ~57 s for AnnData
- The .asql file persists across sessions and can be reopened in R via the duckdb driver with dbGetQuery

## Limitations

- DuckDB memory leak on databases with more than 30k genes remains an unresolved TODO
- The database structure is not optimized for performance; tables are designed to mirror the AnnData object
- Summing all gene counts via SQL (SUM(COLUMNS(*))) is memory intensive; use the chunked calculate_gene_counts approach instead
- No software version numbers reported in the article for AnnSQL/DuckDB, so reproducibility pinning is left to the user

## Evidence

- [intro] AnnSQL converts the layers of an AnnData object into a relational DuckDB database. Each layer is stored as an individual table: "AnnSQL converts the layers of an AnnData object into a relational [DuckDB](https://duckdb.org/) database. Each layer is stored as an individual table"
- [methods] The database structure is not optimized for performance. Instead, the tables are designed to closely mirror the familiar structure of the AnnData object: "The database structure is not optimized for performance. Instead, the tables are designed to closely mirror the familiar structure of the AnnData object"
- [results] MakeDb(adata, db_path="../db/random/", db_name="random_data_"+str(i), create_all_indexes=False, convenience_view=False): "MakeDb(adata, db_path="../db/random/", db_name="random_data_"+str(i), create_all_indexes=False, convenience_view=False)"
- [abstract] AnnSQL-based operations were executed in minutes on a laptop for which equivalent operations in AnnData or Seurat largely failed (or were ∼700× slower): "AnnSQL-based operations were executed in minutes on a laptop for which equivalent operations in AnnData or Seurat largely failed (or were ∼700× slower)"
- [readme] #open the AnnSQL database
asql = AnnSQL(db="db/pbmc3k_reduced.asql"): "#open the AnnSQL database
asql = AnnSQL(db="db/pbmc3k_reduced.asql")"
