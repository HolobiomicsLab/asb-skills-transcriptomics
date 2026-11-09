---
name: graphql-database-querying-for-scrnaseq
description: Use when you need to retrieve per-cell-type differential expression or
  marker scores for a gene (e.g. insulin) from ingested H5AD-derived single-cell studies
  stored in Cellenium's PostgreSQL backend, especially for cross-study comparisons.
when_to_use_negative:
- The study data has not yet been ingested — the database is empty until the data_import
  notebooks are run and a make ..._studydata_import target completes (before make
  reset_database, errors about the missing 'postgraphile' user appear).
- The task is raw H5AD processing, normalization, or DE calculation — that is done
  in the data_import jupyter notebooks, not via the GraphQL API.
- The goal is interactive frontend visualization (UMAP, violin, coexpression plots)
  rather than programmatic retrieval of marker statistics.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: PostgreSQL
  role: Backend database storing integrated single-cell studies, differential expression
    results and per-cluster marker statistics queried via the API
- name: GraphQL
  role: Query language used to retrieve per-gene, per-cell-type differential expression
    values across all studies
- name: Python 3
  role: Implemented in the server backend with PostgreSQL and GraphQL; provides stored
    procedures for server-side rendered plots and the data loaders
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad349
    title: Cellenium—a scalable and interactive visual analytics app for exploring
      multimodal single-cell data
  source_content_hash: sha256:53fa5fcadf2b005bf9e657b6073a4838d6cab70521111690285c10faf5e16102
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GraphQL
  - PostgreSQL
  - Python 3
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# graphql-database-querying-for-scrnaseq

## Summary

Query Cellenium's PostgreSQL-backed single-cell study database through its Postgraphile GraphQL API to retrieve differential expression values and marker statistics per annotated cell type across imported scRNA-seq studies (e.g. the blood_covid.ipynb example), enabling within- and cross-study marker gene searches such as finding pancreatic beta-cells for insulin.

## When to use

Use when you need to retrieve per-cell-type differential expression or marker scores for a gene (e.g. insulin) from ingested H5AD-derived single-cell studies stored in Cellenium's PostgreSQL backend, especially for cross-study comparisons.

## When NOT to use

- The study data has not yet been ingested — the database is empty until the data_import notebooks are run and a make ..._studydata_import target completes (before make reset_database, errors about the missing 'postgraphile' user appear).
- The task is raw H5AD processing, normalization, or DE calculation — that is done in the data_import jupyter notebooks, not via the GraphQL API.
- The goal is interactive frontend visualization (UMAP, violin, coexpression plots) rather than programmatic retrieval of marker statistics.

## Inputs

- H5AD scRNA expression data with cell annotations imported into the PostgreSQL backend via the Python loaders
- Precalculated differential expression results, UMAP projections and marker statistics per annotated cell cluster (as produced by the study data preparation jupyter notebooks)
- GraphQL query for a gene of interest (e.g. 'insulin')

## Outputs

- Per-study ranked cell-type tables of marker/differential-expression statistics for the queried gene (e.g. insulin_cross_study_marker_ranking.json)
- Aggregated cross-study ranking of cell types with consistently highest expression specificity for the gene of interest

## How to apply

First ingest precalculated differential expression results from the public example study notebooks (e.g. blood_covid.ipynb) into PostgreSQL using the Python 3 loaders in data_import (make reset_database plus a studydata_import target). Then issue GraphQL queries against the Postgraphile API at http://localhost:5000/postgraphile/graphiql, using the example queries in the documentation/graphql_api_usage folder as templates, to fetch each gene's differential expression values per annotated cell type in every study. Rank cell types within each study by insulin expression specificity using the marker statistics stored per annotated cell cluster (e.g. log fold change / marker score), aggregate the per-study rankings into a combined cross-study view, and flag consistently top-ranked cell types. Postgraphile listens to database schema changes, so updated API capabilities are visible immediately after ingestion; Python stored procedures back server-side-rendered plot features.

## Related tools

- **PostgreSQL** (Backend database storing integrated single-cell studies, differential expression results and per-cluster marker statistics queried via the API)
- **GraphQL** (Query language used to retrieve per-gene, per-cell-type differential expression values across all studies)
- **Python 3** (Implemented in the server backend with PostgreSQL and GraphQL; provides stored procedures for server-side rendered plots and the data loaders)

## Evaluation signals

- The GraphQL API explorer responds at http://localhost:5000/postgraphile/graphiql and exposes the ingested study schema.
- The output file (e.g. insulin_cross_study_marker_ranking.json) exists and contains ranked cell-type entries per study for the insulin search.
- Pancreatic beta-cell annotations appear among the top-ranked insulin-specific cell types, qualitatively matching the paper's use-case-3 finding (no quantitative threshold is reported).
- Query results include marker statistics (e.g. log fold change / marker score) for each annotated cell-type cluster in each study.
- Postgraphile reflects database schema changes immediately, confirming the API tracks the current ingestion state.

## Limitations

- No quantitative significance threshold is reported for the insulin beta-cell finding; acceptance is qualitative confirmation of the beta-cell ranking.
- Running the study data processing notebooks and ingestion initially takes a couple of hours; the 'test_studydata' set is small enough for a few minutes but 'normal_studydata' imports full real-life studies with all cells and genes.
- Before 'make reset_database' is run, error messages about the missing 'postgraphile' user pile up; some API features (server-side rendered plots) depend on Python stored procedures.

## Evidence

- [readme] Cellenium is a web application that accesses a PostgreSQL database via GraphQL API.: [evidence span withheld — anchor status: external_doc]
- [abstract] The server backend is implemented in PostgreSQL, Python 3, and GraphQL: "The server backend is implemented in PostgreSQL, Python 3, and GraphQL"
- [readme] The GraphQL API explorer is available at http://localhost:5000/postgraphile/graphiql .: [evidence span withheld — anchor status: external_doc]
- [readme] Example queries in the documentation folder illustrate API capabilities: [evidence span withheld — anchor status: external_doc]
- [abstract] Users can analyze single-cell data within and across studies: "Users can then find relevant studies and analyze single-cell data within and across studies"
