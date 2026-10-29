---
name: single-cell-study-integration-metadata-harmonization
description: Use when multiple single-cell RNA/ATAC/CITE-seq studies in H5AD format
  must be imported into a common queryable database and a gene's specific expression
  must be compared across studies to identify the annotated cell types it marks.
when_to_use_negative:
- Only a single study is available and no qualitative cross-study comparison of marker
  expression is required
- The study data has not been processed into H5AD files with cell annotations and
  precalculated differential expression results (raw counts or FASTQ inputs are out
  of scope for this skill)
- A quantitative, batch-corrected cross-study integration of expression values is
  needed — Cellenium's marker search is explicitly a qualitative comparison of gene
  expression across studies
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: PostgreSQL
  role: Backend database into which per-study differential expression results and
    cell annotations are ingested via Python 3 loaders
  repo: https://github.com/Bayer-Group/cellenium
- name: GraphQL
  role: Query API (served via Postgraphile) used to retrieve differential expression
    values per annotated cell type in every study for the gene of interest
  repo: https://github.com/Bayer-Group/cellenium
- name: Python 3
  role: Server backend language and source of the data-import loaders/notebooks that
    process H5AD studies into the PostgreSQL backend
  repo: https://github.com/Bayer-Group/cellenium
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad349
    title: Cellenium—a scalable and interactive visual analytics app for exploring
      multimodal single-cell data
  source_content_hash: sha256:eacf2249bce5ec2613b73b02aa8825491600d51a1f58c9b0c635380f39f465fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Bayer-Group/cellenium
  tools:
  - GraphQL
  - PostgreSQL
  - Python 3
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-cell-study-integration-metadata-harmonization

## Summary

Integrates multiple single-cell studies into Cellenium's PostgreSQL backend and runs cross-study marker gene searches over precalculated differential expression of annotated cell clusters, so a query gene (e.g. insulin) can be qualitatively compared across studies to identify the cell types where it is specifically expressed.

## When to use

Use when multiple single-cell RNA/ATAC/CITE-seq studies in H5AD format must be imported into a common queryable database and a gene's specific expression must be compared across studies to identify the annotated cell types it marks.

## When NOT to use

- Only a single study is available and no qualitative cross-study comparison of marker expression is required
- The study data has not been processed into H5AD files with cell annotations and precalculated differential expression results (raw counts or FASTQ inputs are out of scope for this skill)
- A quantitative, batch-corrected cross-study integration of expression values is needed — Cellenium's marker search is explicitly a qualitative comparison of gene expression across studies

## Inputs

- Single-cell study expression data and cell annotations in H5AD format
- Precalculated differential expression results per cluster of annotated cells (e.g. log fold change / marker score per gene per cell type)
- Ontologized study metadata for tissues and diseases
- Gene of interest identifier (e.g. 'insulin') supplied as a GraphQL query
- Public example study notebooks shipped with the Cellenium repository (e.g. blood_covid.ipynb)

## Outputs

- Cross-study ranked cell-type table for the searched gene (e.g. insulin_cross_study_marker_ranking.json) with ranked cell-type entries per study
- List of cell types consistently ranked highest for the query gene across studies
- GraphQL response containing per-study differential expression values for the gene across all annotated cell types

## How to apply

Prepare each study's expression data and cell annotations in H5AD format, then use the Python 3 loaders in the Cellenium repository to ingest precalculated differential expression results (per annotated cell cluster, e.g. log fold change / marker score) into the PostgreSQL backend; studies are found and organized via ontologized metadata for tissues and diseases. Deploy the stack via docker-compose (docker compose up) and activate the cellenium_import conda environment before running the ingestion make targets. Issue a GraphQL query (via the Postgraphile API at the graphiql explorer) for the gene of interest to retrieve its differential expression values per annotated cell type in every study, rank cell types within each study by expression specificity, aggregate the per-study rankings into a combined cross-study view, and flag cell types that consistently rank highest. Validate qualitatively against a known biology case: for the gene insulin, pancreatic beta-cells should appear among the top-ranked insulin-specific cell types, as reported in the paper's use-case-3 example.

## Related tools

- **PostgreSQL** (Backend database into which per-study differential expression results and cell annotations are ingested via Python 3 loaders) — https://github.com/Bayer-Group/cellenium
- **GraphQL** (Query API (served via Postgraphile) used to retrieve differential expression values per annotated cell type in every study for the gene of interest) — https://github.com/Bayer-Group/cellenium
- **Python 3** (Server backend language and source of the data-import loaders/notebooks that process H5AD studies into the PostgreSQL backend) — https://github.com/Bayer-Group/cellenium

## Examples

```
conda activate cellenium_import; make reset_database normal_studydata_import
```

## Evaluation signals

- The output file (e.g. insulin_cross_study_marker_ranking.json) exists and contains ranked cell-type entries per study for the insulin search
- The beta-cell/pancreatic cell type appears among the top-ranked insulin-specific cell types, matching the paper's use-case-3 finding (qualitative acceptance; no quantitative threshold is reported)
- The PostgreSQL backend contains ingested differential expression results for the imported example studies (verifiable after 'make reset_database test_studydata_import')
- The GraphQL query against the integrated database returns per-annotated-cell-type differential expression values for the query gene in every imported study
- Per-study rankings aggregate into a combined cross-study view with consistently top-ranked cell types flagged

## Limitations

- Cross-study marker comparison is qualitative; the paper reports no quantitative threshold for accepting a top-ranked cell type
- Initial study data preparation and ingestion via the make targets takes a couple of hours; the test_studydata target is small enough to import in a few minutes but the normal_studydata target imports full real-life studies
- Only precalculated differential expression statistics stored per annotated cluster are searched; the skill depends on prior cell annotation quality and on study metadata being ontologized
- Until the 'make reset_database' step is run, error messages about the missing 'postgraphile' user pile up; these can be ignored per the README

## Evidence

- [readme] H5AD ingestion format: [evidence span withheld — anchor status: external_doc]
- [readme] cross-study marker gene search: "find marker genes in all imported studies and qualitatively compare gene expression across studies"
- [abstract] backend stack: "The server backend is implemented in PostgreSQL, Python 3, and GraphQL"
- [intro] ontologized metadata harmonization: "organize and semantically find scRNA studies with ontologized metadata for tissues and diseases"
