---
name: cell-type-expression-specificity-ranking
description: Use when a query gene of interest (e.g. 'insulin') must be matched to
  the annotated cell types that specifically express it, within one study or across
  multiple ingested scRNA-seq studies.
when_to_use_negative:
- The goal is to compute differential expression from raw counts rather than search
  against precalculated marker statistics — Cellenium relies on DE results already
  calculated by the study data preparation notebooks
- The query is for gene–gene correlation or coexpression across pairs of genes, not
  expression specificity of a single gene per cell type
- The studies lack annotated cell types/clusters, so no marker statistics per annotated
  cell type exist to rank
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: PostgreSQL
  role: Backend database storing precalculated differential expression results per
    annotated cell type, queried for marker statistics
  repo: https://github.com/Bayer-Group/cellenium
- name: GraphQL
  role: API layer used to issue gene queries (e.g. 'insulin') against the integrated
    single-cell study database
  repo: https://github.com/Bayer-Group/cellenium
- name: Python 3
  role: Loader scripts that ingest differential expression results from H5AD study
    data into PostgreSQL; stored procedures serve server-side plot features
  repo: https://github.com/Bayer-Group/cellenium
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad349
    title: Cellenium—a scalable and interactive visual analytics app for exploring
      multimodal single-cell data
  source_content_hash: sha256:d763c103ec34ca69d3206481d61ec7f255f72403bc21aed0de0fd41f1f720ad5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Bayer-Group/cellenium
  tools:
  - GraphQL
  - PostgreSQL
  - Python 3
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Cell-type expression specificity ranking

## Summary

Rank annotated cell types within each single-cell study by the specificity of a query gene's expression (e.g. insulin), then aggregate per-study rankings into a cross-study marker search view as demonstrated by Cellenium's use case 3.

## When to use

Use when a query gene of interest (e.g. 'insulin') must be matched to the annotated cell types that specifically express it, within one study or across multiple ingested scRNA-seq studies.

## When NOT to use

- The goal is to compute differential expression from raw counts rather than search against precalculated marker statistics — Cellenium relies on DE results already calculated by the study data preparation notebooks
- The query is for gene–gene correlation or coexpression across pairs of genes, not expression specificity of a single gene per cell type
- The studies lack annotated cell types/clusters, so no marker statistics per annotated cell type exist to rank

## Inputs

- H5AD single-cell expression data with cell annotations (normalized study data)
- Precalculated per-cluster differential expression statistics (marker scores / log fold change per annotated cell type)
- Query gene of interest (e.g. 'insulin')
- PostgreSQL database schema ingested via Cellenium's Python 3 loaders

## Outputs

- Per-study ranked cell-type table for the query gene (e.g. insulin_cross_study_marker_ranking.json)
- Aggregated cross-study ranked list of cell types that specifically express the gene

## How to apply

Ingest precalculated differential expression results (per annotated cell-type cluster, e.g. log fold change / marker score) from H5AD-formatted study data into the Cellenium PostgreSQL backend using the Python 3 loaders shipped in data_import. Issue a GraphQL query against the integrated single-cell study database for the gene of interest to retrieve its differential expression values per annotated cell type in every study. Rank cell types within each study by insulin expression specificity using the stored marker statistics, then aggregate the per-study rankings into a combined cross-study view and flag cell types that consistently rank highest. Verify expected biology — for insulin, pancreatic beta-cells must appear among the top-ranked cell types, matching the paper's use-case-3 example. Since no quantitative threshold is reported, acceptance is qualitative confirmation of the expected cell-type ranking.

## Related tools

- **PostgreSQL** (Backend database storing precalculated differential expression results per annotated cell type, queried for marker statistics) — https://github.com/Bayer-Group/cellenium
- **GraphQL** (API layer used to issue gene queries (e.g. 'insulin') against the integrated single-cell study database) — https://github.com/Bayer-Group/cellenium
- **Python 3** (Loader scripts that ingest differential expression results from H5AD study data into PostgreSQL; stored procedures serve server-side plot features) — https://github.com/Bayer-Group/cellenium

## Examples

```
make reset_database test_studydata_import; # then query via GraphiQL at http://localhost:5000/postgraphile/graphiql for the gene 'insulin'
```

## Evaluation signals

- Output file (e.g. insulin_cross_study_marker_ranking.json) exists and contains ranked cell-type entries per study for the insulin search
- Pancreatic beta-cell annotations appear among the top-ranked insulin-specific cell types, consistent with the paper's use-case-3 example
- Per-study rankings are derived from the marker statistics (log fold change / marker score) stored per cluster of annotated cells, not recomputed ad hoc
- Cross-study view flags cell types that consistently rank highest across studies, not just in one dataset

## Limitations

- No quantitative acceptance threshold is reported in the paper, so validation is qualitative confirmation of expected cell-type ranking (beta-cells for insulin)
- Ranking depends on precalculated differential expression; the ingestion notebooks must have been run and the database reset/populated (e.g. via make reset_database and studydata_import targets) beforehand
- Full example studies take a couple of hours to process and import; small test_studydata covers features but is not real-life data

## Evidence

- [readme] find marker genes in all imported studies and qualitatively compare gene expression across studies: "find marker genes in all imported studies and qualitatively compare gene expression across studies"
- [abstract] The server backend is implemented in PostgreSQL, Python 3, and GraphQL: "The server backend is implemented in PostgreSQL, Python 3, and GraphQL"
- [readme] Cellenium imports scRNA expression data and cell annotations in H5AD format.: [evidence span withheld — anchor status: external_doc]
- [readme] The graphql_api_usage folder contains a couple of example queries to illustrate the API capabilities.: [evidence span withheld — anchor status: not_found]
