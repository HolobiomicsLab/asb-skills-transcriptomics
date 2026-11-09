---
name: cell-type-vs-rest-contrast-computation
description: 'Use when preparing single-cell study data for interactive exploration
  (e.g. Cellenium import) and marker genes must be served instantly per annotated
  cell type: annotated scRNA expression matrix in H5AD plus cell-type labels are available
  offline.'
when_to_use_negative:
- When the goal is to see differentially expressed genes for a newly drawn group of
  cells selected in the plot (user-defined, ad-hoc annotation) rather than predefined
  annotated cell types — this skill covers the predefined cell-type vs rest contrasts
  that are precomputed at import time.
- When the input is raw sequencing data or an unannotated matrix with no cell-type
  labels — the target/background partition requires existing annotations.
- When results must reflect a live, on-the-fly statistical recomputation per query
  rather than precalculated contrasts stored in the backend.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Python 3
  role: Runs the study data preparation notebooks that compute the per-cell-type differential
    expression contrasts on the H5AD expression matrices
  repo: https://github.com/Bayer-Group/cellenium
- name: PostgreSQL
  role: Server backend that stores the precalculated per-cell-type differential expression
    tables so they can be queried without recomputation
  repo: https://github.com/Bayer-Group/cellenium
- name: GraphQL
  role: API layer that serves the precalculated differential expression results to
    the frontend upon cell-type selection
  repo: https://github.com/Bayer-Group/cellenium
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad349
    title: Cellenium—a scalable and interactive visual analytics app for exploring
      multimodal single-cell data
  source_content_hash: sha256:196039f61ee402d6c679d0ecc354a6ddfd3f4ace683e5127abdad06ed0d8066c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Bayer-Group/cellenium
  tools:
  - GraphQL
  - PostgreSQL
  - Python 3
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# cell-type-vs-rest-contrast-computation

## Summary

Precalculates differential gene expression between each annotated cell type and all other cells of a single-cell study, so that Cellenium can serve immediate marker-gene results from the PostgreSQL/GraphQL backend when a user selects a cell type, without recomputation at query time.

## When to use

Use when preparing single-cell study data for interactive exploration (e.g. Cellenium import) and marker genes must be served instantly per annotated cell type: annotated scRNA expression matrix in H5AD plus cell-type labels are available offline.

## When NOT to use

- When the goal is to see differentially expressed genes for a newly drawn group of cells selected in the plot (user-defined, ad-hoc annotation) rather than predefined annotated cell types — this skill covers the predefined cell-type vs rest contrasts that are precomputed at import time.
- When the input is raw sequencing data or an unannotated matrix with no cell-type labels — the target/background partition requires existing annotations.
- When results must reflect a live, on-the-fly statistical recomputation per query rather than precalculated contrasts stored in the backend.

## Inputs

- Annotated single-cell expression matrix with cell-type annotations in H5AD format (e.g. blood_covid study)
- Cell-type annotation labels per cell
- Cellenium data-import conda environment (environment.yml) and study data preparation Jupyter notebook

## Outputs

- One differential-expression table per annotated cell type with gene, effect-size, and significance columns
- Per-cell-type tables serialized as CSV/parquet, structured for ingestion into the PostgreSQL backend and serving via the GraphQL API

## How to apply

Load the annotated single-cell expression matrix and cell-type annotations from H5AD study data (e.g. the blood_covid public study example) in a Python 3 data-import environment. For each annotated cell type, partition the study's cells into a target group (that cell type) and a background group (all other cells of the study), then test every gene for differential expression between the two groups, computing an effect size and a significance measure per gene. Assemble the per-contrast results into a table with gene, effect-size, and significance columns for each cell type, and serialize one table per cell type (CSV/parquet) for ingestion into the PostgreSQL server backend. This precalculation during the study data preparation Jupyter notebook stage is the key design decision: the GraphQL API then serves precomputed contrasts at query time, so selecting a cell type in the web app returns results immediately.

## Related tools

- **Python 3** (Runs the study data preparation notebooks that compute the per-cell-type differential expression contrasts on the H5AD expression matrices) — https://github.com/Bayer-Group/cellenium
- **PostgreSQL** (Server backend that stores the precalculated per-cell-type differential expression tables so they can be queried without recomputation) — https://github.com/Bayer-Group/cellenium
- **GraphQL** (API layer that serves the precalculated differential expression results to the frontend upon cell-type selection) — https://github.com/Bayer-Group/cellenium

## Examples

```
conda activate cellenium_import; make reset_database test_studydata_import
```

## Evaluation signals

- Every annotated cell type in the study has a corresponding differential-expression output table that exists
- Each output table contains the gene, effect-size, and significance fields for its cell type vs all-other-cells contrast
- Tables are queryable from the PostgreSQL backend so that cell-type selection in the web app returns differential expression results immediately, without recomputation
- Import completes via the documented make targets (e.g. test_studydata_import, normal_studydata_import) with the GraphQL API (Postgraphile) reflecting the loaded schema/data
- Marker genes returned for a selected cell type qualitatively match expected markers for that cell type in the study

## Limitations

- Precalculation is limited to contrasts of each annotated cell type against all other cells; other comparisons (e.g. ad-hoc user-selected cell groups) are handled by separate Cellenium features and are not covered by these precomputed tables
- Full study data processing and import can take a couple of hours; huge example studies require editing the notebook to define a download URL before import
- Setup initially logs error messages about the missing 'postgraphile' user until the database reset step; these can be ignored

## Evidence

- [other] Precalculated per-cell-type contrasts at import: [evidence span withheld — anchor status: not_found]
- [abstract] Backend implementation stack: "The server backend is implemented in PostgreSQL, Python 3, and GraphQL"
- [readme] H5AD import and offline calculation in notebooks: [evidence span withheld — anchor status: external_doc]
- [readme] Web app served from PostgreSQL via GraphQL: [evidence span withheld — anchor status: external_doc]
- [readme] Import make targets: [evidence span withheld — anchor status: external_doc]
