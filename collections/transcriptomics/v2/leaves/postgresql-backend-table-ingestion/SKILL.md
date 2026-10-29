---
name: postgresql-backend-table-ingestion
description: Use when preparing a single-cell study (H5AD expression matrix plus cell-type
  annotations) for import into Cellenium and interactive marker-gene lookup must return
  without recomputation at query time.
when_to_use_negative:
- The user has just added a new user-defined cell annotation interactively — Cellenium
  computes differentially expressed genes for such ad-hoc plot selections on the fly
  rather than from precalculated tables
- The expression data are not yet in H5AD format or not normalized — run the data_import
  Jupyter notebooks first (initial setup takes a couple of hours)
- The goal is cross-study marker-gene comparison rather than per-cell-type tables
  for a single imported study
edam_operation: http://edamontology.org/operation_2434
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: Python 3
  role: Runs the data-import/study-preparation Jupyter notebooks that compute differentially
    expressed genes and build the per-cell-type result tables; also implements stored
    procedures behind some GraphQL API features
  repo: https://github.com/Bayer-Group/cellenium
- name: PostgreSQL
  role: Database backend that stores the precalculated per-cell-type differential-expression
    tables so they can be served at query time without recomputation
- name: GraphQL
  role: API layer (via Postgraphile) through which the precalculated tables are queried
    when a user selects a cell type in the webapp
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad349
    title: Cellenium—a scalable and interactive visual analytics app for exploring
      multimodal single-cell data
  source_content_hash: sha256:8c4e1cd6a5c1d1efca9adef2d833ab185af4d5ff613b14c35ec310254bf9fd35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Bayer-Group/cellenium
  tools:
  - GraphQL
  - PostgreSQL
  - Python 3
  edam_operation: http://edamontology.org/operation_2434
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# postgresql-backend-table-ingestion

## Summary

Precalculates differential gene expression between each annotated cell type and all other cells of a study, serializes the per-contrast result tables, and ingests them into the PostgreSQL backend so Cellenium's GraphQL API can serve marker genes immediately when a user selects a cell type.

## When to use

Use when preparing a single-cell study (H5AD expression matrix plus cell-type annotations) for import into Cellenium and interactive marker-gene lookup must return without recomputation at query time.

## When NOT to use

- The user has just added a new user-defined cell annotation interactively — Cellenium computes differentially expressed genes for such ad-hoc plot selections on the fly rather than from precalculated tables
- The expression data are not yet in H5AD format or not normalized — run the data_import Jupyter notebooks first (initial setup takes a couple of hours)
- The goal is cross-study marker-gene comparison rather than per-cell-type tables for a single imported study

## Inputs

- Annotated single-cell expression matrix in H5AD format (e.g. from the blood_covid public study example)
- Cell-type annotations per cell for the imported study
- Cellenium data-import Jupyter notebook outputs (normalized expression, UMAP projection, and other study data)

## Outputs

- One differential-expression table per annotated cell type with gene, effect-size, and significance columns
- Serialized per-cell-type tables (CSV/parquet) ingested into the PostgreSQL schema
- Queryable differential-expression results exposed through the GraphQL API (Postgraphile)

## How to apply

Load the annotated single-cell expression matrix and cell-type annotations (H5AD format, as produced by Cellenium's data-import Jupyter notebooks) using Python 3. For each annotated cell type, partition the study's cells into a target group (the annotated cell type) and a background group (all other cells in the study), then test every gene for differential expression, computing an effect size and a significance measure per gene. Assemble per-contrast tables with gene, effect-size, and significance columns and serialize one table per cell type (CSV/parquet) structured for the PostgreSQL server backend. Ingest the tables into the database schema via the provided make targets (e.g. 'make reset_database test_studydata_import' for the small test study, 'normal_studydata_import' for full-size studies), so the GraphQL API served through Postgraphile exposes the precalculated rows. The rationale is that precomputation at study-import time — which may take a couple of hours for real-life studies with full cells and genes — trades upfront cost for immediate differential-expression feedback during interactive cell-type selection.

## Related tools

- **Python 3** (Runs the data-import/study-preparation Jupyter notebooks that compute differentially expressed genes and build the per-cell-type result tables; also implements stored procedures behind some GraphQL API features) — https://github.com/Bayer-Group/cellenium
- **PostgreSQL** (Database backend that stores the precalculated per-cell-type differential-expression tables so they can be served at query time without recomputation)
- **GraphQL** (API layer (via Postgraphile) through which the precalculated tables are queried when a user selects a cell type in the webapp)

## Examples

```
make reset_database test_studydata_import
```

## Evaluation signals

- Every annotated cell type in the imported study has a corresponding output table
- Each output table contains the gene, effect-size, and significance fields
- Tables are queryable from the PostgreSQL backend so that cell-type selection in the webapp returns differential-expression results immediately without recomputation
- The GraphQL API explorer (http://localhost:5000/postgraphile/graphiql) exposes the ingested tables; Postgraphile reflects database schema changes immediately
- Data completeness for all application features can be checked with the 'test_studydata' import, which is small enough to be imported in a few minutes

## Limitations

- Precalculation covers only predefined annotated cell types; differential expression for newly drawn, user-defined cell groups must be computed interactively instead
- Processing and importing full-size studies (normal_studydata) takes a couple of hours per study during initial setup
- During initial 'docker compose up', error messages about the missing 'postgraphile' user pile up until 'make reset_database' is run; they can be ignored but may look like a failure

## Evidence

- [abstract] backend implemented in PostgreSQL/Python/GraphQL: "The server backend is implemented in PostgreSQL, Python 3, and GraphQL"
- [intro] DE genes based on annotated cell clusters: "find differentially expressed genes based on clusters of annotated cells"
- [readme] H5AD import and precomputation in notebooks: [evidence span withheld — anchor status: external_doc]
- [readme] notebooks calculate DE genes and study data: [evidence span withheld — anchor status: external_doc]
- [readme] webapp accesses PostgreSQL via GraphQL: [evidence span withheld — anchor status: external_doc]
