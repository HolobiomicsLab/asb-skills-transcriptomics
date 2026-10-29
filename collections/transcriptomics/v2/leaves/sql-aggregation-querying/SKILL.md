---
name: sql-aggregation-querying
description: Use when a cells x genes single-cell matrix (e.g. from an AnnData/H5AD
  file) is loaded into PostgreSQL and per-gene or per-cell-type expression summaries
  must be computed or queried efficiently across integrated studies.
when_to_use_negative:
- The data is already a gene x cell-type summary table and needs no sparse re-representation
- The expression matrix is small enough that in-memory Python/pandas aggregation over
  the AnnData object is sufficient, so database materialization adds no value
- The data is not cell-by-gene expression (e.g. ATAC peak counts being analyzed without
  expression aggregation semantics)
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_0091
tools:
- name: PostgreSQL
  role: Stores the gene-centric sparse expression table and executes the gene/cell-type
    GROUP BY aggregation via database functions running as background jobs
  repo: https://github.com/Bayer-Group/cellenium
- name: Python 3
  role: Loads the AnnData object, builds the gene-centric sparse representation, and
    recomputes per-cell-type averages for verification against the SQL results
  repo: https://github.com/Bayer-Group/cellenium
- name: GraphQL
  role: API layer through which the PostgreSQL-backed aggregation features are exposed
    to the Cellenium web application
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad349
    title: Cellenium—a scalable and interactive visual analytics app for exploring
      multimodal single-cell data
  source_content_hash: sha256:7c2a34cbe910e33dd84f0cee78a944cd5a6004c33a242391169dff9c262cb1a3
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
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# sql-aggregation-querying

## Summary

Represents a sparse single-cell expression matrix as gene-centric rows in PostgreSQL and computes per-cell-type expression summaries with SQL GROUP BY aggregation, mirroring Cellenium's core API approach of delegating expression aggregation to PostgreSQL database functions.

## When to use

Use when a cells x genes single-cell matrix (e.g. from an AnnData/H5AD file) is loaded into PostgreSQL and per-gene or per-cell-type expression summaries must be computed or queried efficiently across integrated studies.

## When NOT to use

- The data is already a gene x cell-type summary table and needs no sparse re-representation
- The expression matrix is small enough that in-memory Python/pandas aggregation over the AnnData object is sufficient, so database materialization adds no value
- The data is not cell-by-gene expression (e.g. ATAC peak counts being analyzed without expression aggregation semantics)

## Inputs

- AnnData (H5AD) single-cell expression matrix, e.g. from the blood_covid demo study
- Cell-type annotations per cell from the AnnData object
- PostgreSQL table storing sparse expression entries (gene, cell, expression value)

## Outputs

- Gene x cell-type aggregated expression table (e.g. exported as aggregated_expression.csv)
- Per-cell-type expression summary rows produced by SQL GROUP BY queries

## How to apply

Load the AnnData object (e.g. the blood_covid demo study blood_covid.ipynb) in Python 3 and convert the cells x genes matrix into a gene-centric sparse representation so that only non-zero expression entries are stored per gene. Materialize the sparse values in a PostgreSQL table with gene, cell, and expression-value fields. Compute per-cell-type summaries by running PostgreSQL aggregation functions with GROUP BY over gene and the annotated cell type, following Cellenium's design where aggregation on gene and cell-type level is implemented by PostgreSQL database functions running as database background jobs. Verify correctness by recomputing per-cell-type expression averages directly from the AnnData matrix in Python 3 and comparing them against the SQL aggregation results.

## Related tools

- **PostgreSQL** (Stores the gene-centric sparse expression table and executes the gene/cell-type GROUP BY aggregation via database functions running as background jobs) — https://github.com/Bayer-Group/cellenium
- **Python 3** (Loads the AnnData object, builds the gene-centric sparse representation, and recomputes per-cell-type averages for verification against the SQL results) — https://github.com/Bayer-Group/cellenium
- **GraphQL** (API layer through which the PostgreSQL-backed aggregation features are exposed to the Cellenium web application)

## Examples

```
SELECT gene, cell_type, AVG(expression_value) AS mean_expression FROM expression_sparse GROUP BY gene, cell_type;
```

## Evaluation signals

- Per-cell-type expression averages computed in Python directly from the AnnData matrix match the PostgreSQL GROUP BY aggregation results (the workflow's explicit consistency check)
- The sparse expression table contains only non-zero entries per gene (no zero-valued rows materialized)
- Every cell in the aggregated output maps to exactly one annotated cell type from the AnnData annotations
- The aggregated gene x cell-type table covers the same set of genes and cell types present in the source H5AD study
- Querying aggregated values through the GraphQL API returns results consistent with the underlying PostgreSQL tables

## Limitations

- Initial study data processing and ingestion into PostgreSQL can take a couple of hours (per the README setup instructions), so this approach is costly for quick one-off analyses
- Aggregation is implemented as PostgreSQL database background jobs, so results depend on database function execution rather than immediate computation
- Zero-expression entries are deliberately not stored, so any analysis requiring explicit zeros must reconstruct them from the AnnData matrix

## Evidence

- [other] Aggregation of expression data on gene and cell-type level as core API features: [evidence span withheld — anchor status: not_found]
- [abstract] Backend implemented in PostgreSQL, Python 3, and GraphQL: "The server backend is implemented in PostgreSQL, Python 3, and GraphQL"
- [readme] Web application accessing PostgreSQL via GraphQL API: [evidence span withheld — anchor status: external_doc]
