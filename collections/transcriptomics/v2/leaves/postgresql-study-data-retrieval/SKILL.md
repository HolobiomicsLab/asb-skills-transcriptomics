---
name: postgresql-study-data-retrieval
description: Use when a downstream analysis (e.g. ranking genes correlated to a query
  gene) needs the expression vector of a gene and of every other gene across all cells
  of one imported Cellenium study.
when_to_use_negative:
- The analysis goal is comparing gene expression across multiple studies rather than
  within a single study — use Cellenium's cross-study marker-gene comparison feature
  instead.
- The study data has not yet been prepared and imported (no H5AD processed by the
  data_import notebooks), since the PostgreSQL backend has no expression matrix to
  retrieve.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: PostgreSQL
  role: Server backend database that stores the imported scRNA expression matrices
    and cell annotations; queried for per-gene expression vectors across cells.
  repo: https://github.com/Bayer-Group/cellenium
- name: Python 3
  role: Implements stored procedures for API features such as server-side rendered
    plots and the transcriptome-wide correlation computation against the query gene.
  repo: https://github.com/Bayer-Group/cellenium
- name: GraphQL
  role: API layer (served via Postgraphile) through which the web app and clients
    access the PostgreSQL database; example queries are provided in documentation/graphql_api_usage.
  repo: https://github.com/Bayer-Group/cellenium
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad349
    title: Cellenium—a scalable and interactive visual analytics app for exploring
      multimodal single-cell data
  source_content_hash: sha256:884e577b1b83f5f04cf18aa00657a174b9f8872ac31abecfeac28be88ab164b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Bayer-Group/cellenium
  tools:
  - GraphQL
  - PostgreSQL
  - Python 3
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# postgresql-study-data-retrieval

## Summary

Retrieves single-cell expression matrices and per-gene expression vectors from Cellenium's PostgreSQL backend via the GraphQL API and Python 3 stored procedures, providing the data-access layer for within-study analyses such as transcriptome-wide co-expression of a query gene.

## When to use

Use when a downstream analysis (e.g. ranking genes correlated to a query gene) needs the expression vector of a gene and of every other gene across all cells of one imported Cellenium study.

## When NOT to use

- The analysis goal is comparing gene expression across multiple studies rather than within a single study — use Cellenium's cross-study marker-gene comparison feature instead.
- The study data has not yet been prepared and imported (no H5AD processed by the data_import notebooks), since the PostgreSQL backend has no expression matrix to retrieve.

## Inputs

- Single-cell RNA-seq expression matrix in H5AD format, imported into the Cellenium PostgreSQL database (e.g. the public blood_covid demo study)
- A query gene identifier within the selected study
- GraphQL API queries (see the documentation/graphql_api_usage example queries)

## Outputs

- Expression vector of the query gene across all cells of the study
- Ranked gene-by-correlation table (coexpression_ranked_table.csv) with one row per analyzed gene and a correlation-coefficient column bounded to [-1, 1]
- Pairwise gene-pair expression values feeding the coexpression (scatter) plot view

## How to apply

Point the client at the PostgreSQL database exposed through the GraphQL API (GraphiQL explorer at http://localhost:5000/postgraphile/graphiql), after studies have been imported from H5AD via the data_import notebooks and make targets. For the public blood_covid demo study, query the expression matrix and extract the query gene's expression vector across all cells together with the vectors of every other gene in the study. Route compute-heavy steps (server-side rendered plots, transcriptome-wide correlation) through Python 3 stored procedures in the backend. Rank all genes by their correlation coefficient with the query gene in descending order and persist the result, e.g. to coexpression_ranked_table.csv, keeping the pairwise values that also feed the gene-pair coexpression (scatter) plot view. Validate that the ranked table has one row per analyzed gene and that all coefficients fall within the bounded range [-1, 1].

## Related tools

- **PostgreSQL** (Server backend database that stores the imported scRNA expression matrices and cell annotations; queried for per-gene expression vectors across cells.) — https://github.com/Bayer-Group/cellenium
- **Python 3** (Implements stored procedures for API features such as server-side rendered plots and the transcriptome-wide correlation computation against the query gene.) — https://github.com/Bayer-Group/cellenium
- **GraphQL** (API layer (served via Postgraphile) through which the web app and clients access the PostgreSQL database; example queries are provided in documentation/graphql_api_usage.) — https://github.com/Bayer-Group/cellenium

## Examples

```
make reset_database test_studydata_import
```

## Evaluation signals

- The output file (e.g. coexpression_ranked_table.csv) exists and contains exactly one row per analyzed gene with a correlation-coefficient column.
- All correlation coefficients fall within the mathematically bounded range [-1, 1].
- Genes are ranked in descending order by correlation coefficient with the query gene.
- The query gene's own correlation with itself equals 1.0 (sanity anchor for the ranking).
- The pairwise co-expression values retrieved are consistent with the values shown in the gene-pair coexpression (scatter) plot view.

## Limitations

- Only studies that were processed by the data_import notebooks and imported as H5AD into PostgreSQL are retrievable; the initial setup and study data processing 'will take a couple of hours to run'.
- Before running 'make reset_database', error messages about the missing 'postgraphile' user pile up and can be misleading during startup.
- Some API features depend on Python stored procedures, so retrieval-backed analyses (e.g. server-side rendered plots) require the full dockerized stack, not just a bare database.

## Evidence

- [abstract] The server backend is implemented in PostgreSQL, Python 3, and GraphQL: "The server backend is implemented in PostgreSQL, Python 3, and GraphQL"
- [intro] the public study example blood_covid.ipynb: "the public study example blood_covid.ipynb"
- [readme] Cellenium is a web application that accesses a PostgreSQL database via GraphQL API.: [evidence span withheld — anchor status: external_doc]
- [readme] find genes which expression is highly correlated to a query gene: "find genes which expression is highly correlated to a query gene"
