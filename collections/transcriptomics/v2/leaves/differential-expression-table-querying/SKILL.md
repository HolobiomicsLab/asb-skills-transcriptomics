---
name: differential-expression-table-querying
description: Use when a gene of interest (e.g. 'insulin') must be matched to the cell
  types in which it is specifically expressed, using precalculated per-cluster differential
  expression/marker scores already ingested into Cellenium's database across one or
  more scRNA-seq studies.
when_to_use_negative:
- The input is a raw scRNA count matrix for which differential expression per annotated
  cluster has not yet been calculated — Cellenium queries precalculated DE results
  produced by its study-data preparation notebooks, it does not compute DE on the
  fly.
- Quantitative per-cell expression values or a novel statistical significance threshold
  for marker specificity are required — the workflow's acceptance criterion is qualitative
  because the paper reports no quantitative threshold.
- The goal is interactive UMAP/violin-plot exploration rather than ranked marker-gene
  retrieval across studies — those are distinct Cellenium features.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: PostgreSQL
  role: Backend database holding the precalculated differential expression results
    per annotated cluster; queried for per-cell-type marker statistics
  repo: https://github.com/Bayer-Group/cellenium
- name: GraphQL
  role: API layer (Postgraphile) through which the gene query is issued against the
    integrated single-cell study database
  repo: https://github.com/Bayer-Group/cellenium
- name: Python 3
  role: Loader scripts (Python 3 stored procedures/loaders) that ingest the precalculated
    DE results from H5AD study data into PostgreSQL
  repo: https://github.com/Bayer-Group/cellenium
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad349
    title: Cellenium—a scalable and interactive visual analytics app for exploring
      multimodal single-cell data
  source_content_hash: sha256:6a34cb19a05ee6e7b04db4d0b2222895fce7a66e7e018003d94218b2edb95abf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Bayer-Group/cellenium
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

# differential-expression-table-querying

## Summary

Query Cellenium's PostgreSQL/GraphQL-backed store of precalculated differential expression results to retrieve, rank, and cross-study aggregate marker statistics per annotated cell type — e.g. confirming that pancreatic beta-cells rank highest for the gene 'insulin' (paper use case 3).

## When to use

Use when a gene of interest (e.g. 'insulin') must be matched to the cell types in which it is specifically expressed, using precalculated per-cluster differential expression/marker scores already ingested into Cellenium's database across one or more scRNA-seq studies.

## When NOT to use

- The input is a raw scRNA count matrix for which differential expression per annotated cluster has not yet been calculated — Cellenium queries precalculated DE results produced by its study-data preparation notebooks, it does not compute DE on the fly.
- Quantitative per-cell expression values or a novel statistical significance threshold for marker specificity are required — the workflow's acceptance criterion is qualitative because the paper reports no quantitative threshold.
- The goal is interactive UMAP/violin-plot exploration rather than ranked marker-gene retrieval across studies — those are distinct Cellenium features.

## Inputs

- Precalculated differential expression results per cluster of annotated cells, ingested into PostgreSQL from H5AD study data (e.g. blood_covid.ipynb public example study)
- Gene of interest identifier (e.g. 'insulin')
- Annotated cell-type labels per study, as stored in the single-cell study database

## Outputs

- Per-study ranked cell-type tables by marker specificity for the query gene
- Aggregated cross-study ranking of consistently top-ranked cell types (e.g. insulin_cross_study_marker_ranking.json)
- Qualitative confirmation of expected cell type (pancreatic beta-cells for insulin)

## How to apply

First ensure the precalculated differential expression results of the study datasets (e.g. the public blood_covid.ipynb example, exported as H5AD) have been ingested into the PostgreSQL backend with the Python 3 loaders. Issue a GraphQL query against the integrated single-cell study database (Postgraphile API) for the gene of interest to retrieve its differential expression values per annotated cell type in every study. Rank cell types within each study by the marker statistics stored per cluster of annotated cells (e.g. log fold change / marker score of the gene in each annotated cluster). Aggregate the per-study rankings into a combined cross-study view and flag cell types that consistently rank highest; for the paper's use case 3, pancreatic beta-cells should surface as specifically expressing insulin. Acceptance is qualitative — the paper reports no quantitative threshold — so validation is the confirmation of the expected top-ranked cell type.

## Related tools

- **PostgreSQL** (Backend database holding the precalculated differential expression results per annotated cluster; queried for per-cell-type marker statistics) — https://github.com/Bayer-Group/cellenium
- **GraphQL** (API layer (Postgraphile) through which the gene query is issued against the integrated single-cell study database) — https://github.com/Bayer-Group/cellenium
- **Python 3** (Loader scripts (Python 3 stored procedures/loaders) that ingest the precalculated DE results from H5AD study data into PostgreSQL) — https://github.com/Bayer-Group/cellenium

## Evaluation signals

- The output file (e.g. insulin_cross_study_marker_ranking.json) exists and contains ranked cell-type entries per study for the insulin search.
- The beta-cell/pancreatic cell type appears among the top-ranked insulin-specific cell types, consistent with the paper's reported use-case-3 example.
- Per-study rankings are computed from the marker statistics (log fold change / marker score) stored per annotated cluster, not recomputed from raw counts.
- The cross-study aggregate flags cell types that consistently rank highest across studies, not just in one study.
- The GraphQL response covers the gene's differential expression values per annotated cell type in every ingested study.

## Limitations

- No quantitative threshold is reported for marker specificity, so acceptance is only a qualitative confirmation of the expected cell-type ranking.
- Rankings depend on the quality of prior cell annotations and on precalculated DE results per cluster of annotated cells; upstream annotation errors propagate into the ranking.
- Qualitative cross-study comparison only — the README describes marker-gene comparison across imported studies as qualitative.

## Evidence

- [abstract] Server backend tools: "The server backend is implemented in PostgreSQL, Python 3, and GraphQL"
- [readme] Architecture: GraphQL over PostgreSQL: [evidence span withheld — anchor status: external_doc]
- [readme] GraphQL API explorer location: [evidence span withheld — anchor status: external_doc]
- [readme] Marker-gene comparison across studies: "find marker genes in all imported studies and qualitatively compare gene expression across studies"
- [intro] DE based on annotated clusters: "find differentially expressed genes based on clusters of annotated cells"
