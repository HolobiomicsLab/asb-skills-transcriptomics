---
name: sql-convenience-view-join-construction
description: Use when building an AnnSQL database from an AnnData object and you need
  single-table SQL access to gene expression joined with cell metadata (obs), e.g.
  group-by queries on cell_type with per-gene averages.
when_to_use_negative:
- You only need raw X-layer filtering (e.g. SELECT gene_1 FROM X WHERE gene_1 > 0.5)
  with no obs metadata — querying the X table directly avoids join overhead
- Bulk benchmarking with create_all_indexes=False and convenience_view=False where
  per-query explicit joins are preferred over a persistent view
- Datasets with >30k genes where the known DuckDB memory leak makes large joined view
  construction risky
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0091
tools:
- name: AnnSQL
  role: Provides the AnnSQL/MakeDb/BuildDb classes that create the joined 'adata'
    convenience view and executes queries against it, returning Pandas DataFrames
  repo: https://github.com/ArpiarSaundersLab/annsql
- name: DuckDB
  role: In-process relational engine that stores the X and obs tables and evaluates
    the cell_id join underlying the convenience view
  repo: https://duckdb.org/
- name: AnnData
  role: Source object whose X layer and obs metadata are converted into the joined
    relational structure
  repo: https://anndata.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:f6891527dff52ce165d990403f5263465a89f6399ceda02439e7c7f245389f82
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ArpiarSaundersLab/annsql
  tools:
  - AnnData
  - AnnSQL
  - DuckDB
  - Python
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# sql-convenience-view-join-construction

## Summary

Construct a joined SQL convenience view (named 'adata') that combines the per-gene expression table with obs cell metadata via the cell_id key, mirroring the AnnData object structure so downstream SQL queries can access expression and metadata in one place.

## When to use

Use when building an AnnSQL database from an AnnData object and you need single-table SQL access to gene expression joined with cell metadata (obs), e.g. group-by queries on cell_type with per-gene averages.

## When NOT to use

- You only need raw X-layer filtering (e.g. SELECT gene_1 FROM X WHERE gene_1 > 0.5) with no obs metadata — querying the X table directly avoids join overhead
- Bulk benchmarking with create_all_indexes=False and convenience_view=False where per-query explicit joins are preferred over a persistent view
- Datasets with >30k genes where the known DuckDB memory leak makes large joined view construction risky

## Inputs

- AnnData object (.h5ad or in-memory) with X expression matrix, obs cell metadata, and cell_id keys
- DuckDB database connection (in-memory or on-disk .asql file)

## Outputs

- Joined SQL view named 'adata' combining expression table columns with obs metadata via cell_id
- Pandas DataFrame results from asql.query() against the view

## How to apply

When converting an AnnData object (e.g. sc.datasets.pbmc68k_reduced() or pbmc3k) with MakeDb or BuildDb, serialize the X layer into an expression table with one column per gene plus a cell_id key column, and obs into its own property table keyed by cell_id. Then create the joined 'adata' convenience view that combines the expression table with obs metadata via cell_id, enabling queries like 'SELECT obs.cell_type, AVG(gene_1) ... FROM X INNER JOIN obs ON X.cell_id = obs.cell_id'. Note the view is a convenience construct — the database mirrors AnnData structure rather than being performance-optimized. When building benchmark databases you may pass convenience_view=False (as done with MakeDb for the 1,000–250,000 cell random datasets) and instead write explicit INNER JOIN clauses in each query.

## Related tools

- **AnnSQL** (Provides the AnnSQL/MakeDb/BuildDb classes that create the joined 'adata' convenience view and executes queries against it, returning Pandas DataFrames) — https://github.com/ArpiarSaundersLab/annsql
- **DuckDB** (In-process relational engine that stores the X and obs tables and evaluates the cell_id join underlying the convenience view) — https://duckdb.org/
- **AnnData** (Source object whose X layer and obs metadata are converted into the joined relational structure) — https://anndata.readthedocs.io/en/latest/

## Examples

```
from AnnSQL import AnnSQL; asql = AnnSQL(db="db/pbmc3k_reduced.asql"); asql.query("SELECT obs.cell_type, AVG(gene_1) as avg_gene_1 FROM X INNER JOIN obs ON X.cell_id = obs.cell_id GROUP BY obs.cell_type")
```

## Evaluation signals

- Opening db/pbmc3k_reduced.asql with AnnSQL(db='db/pbmc3k_reduced.asql') and running asql.query('SELECT * FROM adata LIMIT 5') returns rows matching the source AnnData dimensions (n_obs x n_vars)
- The view resolves obs columns not present in X (e.g. bulk_labels, cell_type), proving the cell_id join succeeded
- Aggregative query 'SELECT obs.cell_type, AVG(gene_1) ... GROUP BY obs.cell_type' returns one row per observed cell_type label
- SQL-level query 'SELECT obs.bulk_labels, COUNT(*) FROM obs GROUP BY obs.bulk_labels' runs without error, confirming obs table is join-compatible on cell_id

## Limitations

- The database structure is not optimized for performance; tables are designed to closely mirror the AnnData object structure rather than for query speed
- Known unresolved DuckDB memory leak on databases with more than 30k genes (TODO noted in benchmark code)
- The convenience view is optional — MakeDb benchmarking calls pass convenience_view=False, so code relying on 'adata' must request view creation

## Evidence

- [methods] Convenience view creation: [evidence span withheld — anchor status: not_found]
- [intro] AnnData-to-relational conversion: "AnnSQL converts the layers of an AnnData object into a relational [DuckDB](https://duckdb.org/) database. Each layer is stored as an individual table"
- [methods] Mirroring over optimization: "The database structure is not optimized for performance. Instead, the tables are designed to closely mirror the familiar structure of the AnnData object"
- [results] Join-based group-by query pattern: "SELECT obs.cell_type, AVG(gene_1) as avg_gene_1 FROM X INNER JOIN obs ON X.cell_id = obs.cell_id GROUP BY obs.cell_type"
- [results] Optional view in benchmarking: "MakeDb(adata, db_path="../db/random/", db_name="random_data_"+str(i), create_all_indexes=False, convenience_view=False)"
- [readme] View usage returns DataFrame: "query the expression table. Returns Pandas Dataframe by Default
asql.query("SELECT * FROM adata LIMIT 10")"
