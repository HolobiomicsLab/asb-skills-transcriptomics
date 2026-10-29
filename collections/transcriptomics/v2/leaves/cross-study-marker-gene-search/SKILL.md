---
name: cross-study-marker-gene-search
description: Use when a specific gene of interest (e.g. 'insulin') is given and the
  goal is to find which annotated cell types specifically express it across multiple
  imported single-cell studies in Cellenium's integrated database.
when_to_use_negative:
- The single-cell study lacks precalculated differential expression results per annotated
  cell cluster — Cellenium requires DE values, a UMAP projection and other study data
  prepared by the data_import notebooks before ingestion
- A quantitative cross-study meta-analysis with significance thresholds is required
  — Cellenium's marker search supports only qualitative cross-study comparison of
  gene expression
- Only a single study is involved and a within-study differentially expressed gene
  lookup suffices, rather than a cross-study marker search
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: PostgreSQL
  role: Backend database storing the integrated single-cell studies and per-cluster
    differential expression / marker statistics queried by the search
  repo: https://github.com/Bayer-Group/cellenium
- name: GraphQL
  role: API layer used to issue queries against the integrated single-cell study database
    to retrieve the gene's differential expression values per annotated cell type
    in every study
  repo: https://github.com/Bayer-Group/cellenium
- name: Python 3
  role: Server-backend language and language of the data_import loaders/notebooks
    that ingest precalculated DE results into PostgreSQL
  repo: https://github.com/Bayer-Group/cellenium
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad349
    title: Cellenium—a scalable and interactive visual analytics app for exploring
      multimodal single-cell data
  source_content_hash: sha256:f015ef2832a92cf807900f9ad238204dbd2cb657613526bf67a43a4bd868349f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Bayer-Group/cellenium
  tools:
  - GraphQL
  - PostgreSQL
  - Python 3
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cross-study-marker-gene-search

## Summary

Reproduce Cellenium's cross-study marker gene search: query the PostgreSQL/GraphQL single-cell study backend for a gene of interest (e.g. insulin), rank annotated cell types by differential-expression marker statistics within each study, and aggregate per-study rankings to identify cell types consistently specific for that gene (paper use-case 3 expected result: pancreatic beta-cells for insulin).

## When to use

Use when a specific gene of interest (e.g. 'insulin') is given and the goal is to find which annotated cell types specifically express it across multiple imported single-cell studies in Cellenium's integrated database.

## When NOT to use

- The single-cell study lacks precalculated differential expression results per annotated cell cluster — Cellenium requires DE values, a UMAP projection and other study data prepared by the data_import notebooks before ingestion
- A quantitative cross-study meta-analysis with significance thresholds is required — Cellenium's marker search supports only qualitative cross-study comparison of gene expression
- Only a single study is involved and a within-study differentially expressed gene lookup suffices, rather than a cross-study marker search

## Inputs

- Precalculated differential expression results per annotated cell-type cluster from single-cell study datasets (H5AD expression data plus cell annotations)
- Public example study notebooks shipped with the Cellenium repository (e.g. blood_covid.ipynb)
- Gene of interest identifier (e.g. 'insulin') supplied to the GraphQL query

## Outputs

- Per-study ranked cell-type table for the marker gene search (e.g. insulin_cross_study_marker_ranking.json)
- Cross-study view of top-ranked, consistently high-scoring cell types for the gene of interest (expected: pancreatic beta-cells for insulin)

## How to apply

First ingest the precalculated differential expression results of the example study datasets (e.g. the public blood_covid.ipynb notebook shipped with the Cellenium repository, provided in H5AD format) into the PostgreSQL backend using the Python 3 loaders via the make targets (e.g. 'make reset_database test_studydata_import' or 'make normal_studydata_import'). Then issue a GraphQL query against the integrated single-cell study database for the gene of interest to retrieve its differential expression values per annotated cell type in every study. Rank the cell types within each study by the marker statistics stored per cluster of annotated cells (e.g. log fold change / marker score of the gene in each annotated cluster). Aggregate the per-study rankings into a combined cross-study view and flag cell types that consistently rank highest; Cellenium's stated purpose is to 'qualitatively compare gene expression across studies', so no quantitative cross-study threshold is applied. Finally, write the per-study ranked cell-type table (e.g. insulin_cross_study_marker_ranking.json) and confirm the expected specific cell type — for insulin, pancreatic beta-cells — appears among the top-ranked entries.

## Related tools

- **PostgreSQL** (Backend database storing the integrated single-cell studies and per-cluster differential expression / marker statistics queried by the search) — https://github.com/Bayer-Group/cellenium
- **GraphQL** (API layer used to issue queries against the integrated single-cell study database to retrieve the gene's differential expression values per annotated cell type in every study) — https://github.com/Bayer-Group/cellenium
- **Python 3** (Server-backend language and language of the data_import loaders/notebooks that ingest precalculated DE results into PostgreSQL) — https://github.com/Bayer-Group/cellenium

## Examples

```
make reset_database normal_studydata_import && curl -X POST -H 'Content-Type: application/json' -d '{"query":"{ geneByName(name:\"insulin\") { differentialExpressionPerCellType } }"}' http://localhost:5000/postgraphile/graphql
```

## Evaluation signals

- Output file (e.g. insulin_cross_study_marker_ranking.json) exists and contains ranked cell-type entries per study for the insulin marker search
- The beta-cell/pancreatic cell type appears among the top-ranked insulin-specific cell types, consistent with the paper's reported use-case-3 example
- Rankings are derived from marker statistics stored per cluster of annotated cells (log fold change / marker score) rather than raw expression means
- Every imported study's differential expression values are reachable via the GraphQL API before ranking is performed
- No quantitative threshold is reported by the paper, so acceptance is the qualitative confirmation of the beta-cell ranking

## Limitations

- The paper reports no quantitative threshold for cross-study marker specificity; acceptance is qualitative confirmation that the expected cell type (e.g. pancreatic beta-cells for insulin) ranks among the top
- Initial study data preparation and ingestion is slow — running the setup, including execution of the study data processing notebooks, 'initially ... will take a couple of hours'
- During initial setup, error messages about the missing 'postgraphile' user pile up until 'make reset_database' is run; these can be ignored but may look like a failed ingestion
- Only precalculated study data (DE results, UMAP projection) computed by the shipped notebooks is searchable; raw scRNA matrices without these artifacts cannot be queried

## Evidence

- [abstract] Backend stack: "The server backend is implemented in PostgreSQL, Python 3, and GraphQL"
- [readme] Marker search purpose: "find marker genes in all imported studies and qualitatively compare gene expression across studies"
- [readme] Input data format: [evidence span withheld — anchor status: external_doc]
- [readme] GraphQL API access: [evidence span withheld — anchor status: external_doc]
