---
name: precomputed-result-table-serialization
description: Use when an interactive analytics application must return analysis results
  (e.g. differential expression per annotated cell type) instantly on user selection,
  and precomputing contrasts at data-import time is acceptable.
when_to_use_negative:
- The user needs a differential expression contrast that was not precomputed during
  import (e.g. a newly drawn group of cells added via the interactive cell-annotation
  feature) — that requires on-the-fly computation, not the precalculated tables
- Input is raw sequencing output rather than an annotated H5AD matrix; normalization
  and preprocessing by the study-data preparation notebooks must happen first
- The analysis goal is cross-study qualitative comparison of gene expression rather
  than per-cell-type marker-gene lookup within one study
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Python 3
  role: Runs the study-data preparation notebooks that compute differential expression
    contrasts per annotated cell type during import
  repo: https://github.com/Bayer-Group/cellenium
- name: PostgreSQL
  role: Server backend that stores the serialized per-cell-type differential-expression
    tables for serving at query time
  repo: https://github.com/Bayer-Group/cellenium
- name: GraphQL
  role: API layer (via Postgraphile) through which the precalculated tables are served
    to the frontend on cell-type selection
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad349
    title: Cellenium—a scalable and interactive visual analytics app for exploring
      multimodal single-cell data
  source_content_hash: sha256:a3d5bafc6b3b778bfed7550b05e5a0effc84d02cac8e5ce35e4e046b423d071b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Bayer-Group/cellenium
  tools:
  - GraphQL
  - PostgreSQL
  - Python 3
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# precomputed-result-table-serialization

## Summary

Precalculates per-cell-type differential gene expression contrasts (annotated cell type vs. all other cells) offline in Python 3 and serializes the gene/effect-size/significance tables into the PostgreSQL backend, so Cellenium's GraphQL API can return marker-gene results immediately at interactive query time without recomputation.

## When to use

Use when an interactive analytics application must return analysis results (e.g. differential expression per annotated cell type) instantly on user selection, and precomputing contrasts at data-import time is acceptable.

## When NOT to use

- The user needs a differential expression contrast that was not precomputed during import (e.g. a newly drawn group of cells added via the interactive cell-annotation feature) — that requires on-the-fly computation, not the precalculated tables
- Input is raw sequencing output rather than an annotated H5AD matrix; normalization and preprocessing by the study-data preparation notebooks must happen first
- The analysis goal is cross-study qualitative comparison of gene expression rather than per-cell-type marker-gene lookup within one study

## Inputs

- Annotated single-cell expression matrix with cell-type annotations in H5AD format (e.g. from the blood_covid.ipynb public study example)
- Normalized scRNA-Seq (and optionally scATAC-Seq / CITE-Seq) expression data prepared by the data_import jupyter notebooks
- PostgreSQL database schema set up via 'make reset_database' for result-table ingestion

## Outputs

- One differential-expression result table per annotated cell type, with gene, effect size, and significance columns, stored in the PostgreSQL backend
- Precomputed marker-gene results queryable via the GraphQL API (Postgraphile) at query time without recomputation

## How to apply

Load the annotated single-cell expression matrix and cell-type annotations in H5AD format (e.g. the blood_covid public study example) using the data-import Python 3 environment (cellenium_import). For each annotated cell type, partition cells into a target group (that cell type) and a background group (all other cells of the study), then test every gene for differential expression between the two groups, computing an effect size and a significance measure per gene — this is done in the jupyter notebooks that also compute the UMAP projection and other study data Cellenium's features require. Assemble per-contrast results into tables with gene, effect size, and significance columns, one table per cell type, and ingest them into the PostgreSQL schema via the make import targets. Validation requires that every annotated cell type has a corresponding table that exists, contains the gene/effect-size/significance fields, and is queryable through the GraphQL API (exposed by Postgraphile at /postgraphile/graphiql) so that selecting a cell type in the ReactJS client returns marker genes immediately.

## Related tools

- **Python 3** (Runs the study-data preparation notebooks that compute differential expression contrasts per annotated cell type during import) — https://github.com/Bayer-Group/cellenium
- **PostgreSQL** (Server backend that stores the serialized per-cell-type differential-expression tables for serving at query time) — https://github.com/Bayer-Group/cellenium
- **GraphQL** (API layer (via Postgraphile) through which the precalculated tables are served to the frontend on cell-type selection)

## Examples

```
conda activate cellenium_import; make reset_database test_studydata_import
```

## Evaluation signals

- Every annotated cell type in the source H5AD has a corresponding differential-expression table in the PostgreSQL database (no missing contrasts)
- Each table contains gene, effect size, and significance fields with valid values
- Querying the GraphQL API for a selected cell type returns precalculated marker genes immediately, with no recomputation latency
- The imported study covers all application features after 'make reset_database test_studydata_import', whose test studydata is small enough to be imported in a few minutes

## Limitations

- Only contrasts defined at import time (each annotated cell type vs. all other cells) are precalculated; other comparisons require recomputation or new notebook runs
- Processing and importing full-size real-life studies ('normal_studydata') takes a couple of hours, since notebooks run in headless mode and calculate DE genes, UMAP projections, and other study data
- Initially, error messages about the missing 'postgraphile' user pile up after 'docker compose up' until 'make reset_database' is run; they can be ignored but indicate the API is not yet up

## Evidence

- [full_text] Cellenium precalculates differential gene expressions between each annotated cell type and all other cells of the study, so the user receives immediate feedback when selecting a cell type.: [evidence span withheld — anchor status: not_found]
- [readme] We provide jupyter notebooks for downloading some publicly available scRNA studies, normalize the data if necessary, and calculate differentially expressed genes, a UMAP projection and other study data: "We provide jupyter notebooks for
downloading some publicly available scRNA studies, normalize the data if necessary, and calculate differentially
expressed genes, a UMAP projection and other study"
- [abstract] The server backend is implemented in PostgreSQL, Python 3, and GraphQL: "The server backend is implemented in PostgreSQL, Python 3, and GraphQL"
- [readme] Cellenium is a web application that accesses a PostgreSQL database via GraphQL API.: [evidence span withheld — anchor status: external_doc]
