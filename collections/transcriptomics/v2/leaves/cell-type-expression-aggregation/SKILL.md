---
name: cell-type-expression-aggregation
description: Use when a scRNA-seq study stored as an AnnData/H5AD object must be loaded
  into a relational sparse expression table and queried for gene-level per-cell-type
  averages, e.g. to replicate Cellenium's server-side aggregation.
when_to_use_negative:
- The input is already a pre-aggregated gene x cell-type feature table rather than
  a cells x genes matrix with cell annotations
- The goal is interactive visual exploration (UMAP plots, violin plots) rather than
  materializing aggregated expression summaries
- The study data is not in H5AD/AnnData format with cell-type annotations available,
  since both the matrix and the annotation are required for the GROUP BY aggregation
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: PostgreSQL
  role: Stores the gene-centric sparse expression table (gene, cell, expression value)
    and runs the per-cell-type aggregation via database functions/GROUP BY queries
  repo: https://github.com/Bayer-Group/cellenium
- name: Python 3
  role: Loads the AnnData object, extracts the expression matrix and cell-type annotations,
    converts to sparse gene-centric form, and recomputes per-cell-type averages for
    cross-validation
  repo: https://github.com/Bayer-Group/cellenium
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad349
    title: Cellenium—a scalable and interactive visual analytics app for exploring
      multimodal single-cell data
  source_content_hash: sha256:2227c341c8521062e9ae14dd7a4780cf9a63f9927b77caa63a9bb45b03f48b33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Bayer-Group/cellenium
  tools:
  - PostgreSQL
  - Python 3
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cell-type-expression-aggregation

## Summary

Reconstructs Cellenium's gene-centric sparse storage of single-cell expression data in PostgreSQL and computes per-cell-type expression summaries via SQL aggregation, mirroring how Cellenium implements gene and cell-type-level aggregation as database functions.

## When to use

Use when a scRNA-seq study stored as an AnnData/H5AD object must be loaded into a relational sparse expression table and queried for gene-level per-cell-type averages, e.g. to replicate Cellenium's server-side aggregation.

## When NOT to use

- The input is already a pre-aggregated gene x cell-type feature table rather than a cells x genes matrix with cell annotations
- The goal is interactive visual exploration (UMAP plots, violin plots) rather than materializing aggregated expression summaries
- The study data is not in H5AD/AnnData format with cell-type annotations available, since both the matrix and the annotation are required for the GROUP BY aggregation

## Inputs

- AnnData / H5AD single-cell study object (e.g. blood_covid.ipynb demo study) containing a cells x genes expression matrix and cell-type annotations
- PostgreSQL database instance for materializing the sparse expression table

## Outputs

- PostgreSQL table with gene, cell, and expression-value fields (gene-centric sparse representation, zero entries omitted)
- aggregated_expression.csv: gene x cell-type table of per-cell-type expression summaries

## How to apply

Load the study AnnData object (e.g. the blood_covid demo from blood_covid.ipynb) in Python 3 and extract the cells x genes expression matrix together with cell-type annotations. Convert the matrix to a gene-centric sparse representation so only non-zero expression entries are kept, then materialize them in a PostgreSQL table with gene, cell, and expression-value fields. Aggregate with a SQL GROUP BY over gene and annotated cell type (PostgreSQL database functions running as background jobs in the real Cellenium system) to compute per-cell-type expression summaries, and write the gene x cell-type table to aggregated_expression.csv. Rationale: Cellenium imports scRNA expression data and cell annotations in H5AD format into PostgreSQL accessed via a GraphQL API, with aggregation features implemented by database functions.

## Related tools

- **PostgreSQL** (Stores the gene-centric sparse expression table (gene, cell, expression value) and runs the per-cell-type aggregation via database functions/GROUP BY queries) — https://github.com/Bayer-Group/cellenium
- **Python 3** (Loads the AnnData object, extracts the expression matrix and cell-type annotations, converts to sparse gene-centric form, and recomputes per-cell-type averages for cross-validation) — https://github.com/Bayer-Group/cellenium

## Examples

```
SELECT gene, cell_type, AVG(expr_value) FROM expression JOIN cell_annotations USING (cell) GROUP BY gene, cell_type;
```

## Evaluation signals

- The PostgreSQL sparse table contains only non-zero expression entries, each carrying gene, cell, and expression-value fields
- Recomputing per-cell-type expression averages directly from the AnnData matrix in Python 3 matches the PostgreSQL GROUP BY aggregation results
- Output aggregated_expression.csv is a gene x cell-type table with one summary value per gene/cell-type combination
- SQL query groups by both gene and annotated cell type, producing cell-type-level (not cell-level) summaries

## Limitations

- Full-scale study import ('normal_studydata' with full amounts of cells and genes) is slow — Cellenium's setup notebooks initially take a couple of hours to run
- Cellenium's aggregation runs as PostgreSQL database background jobs, so timing/asynchronicity differs from a direct interactive SQL query
- Sparse storage omits zero-expression entries, so queries must handle missing cell x gene pairs when reconstructing dense values

## Evidence

- [other] aggregation of expression data on gene and cell-type level as core API features implemented by PostgreSQL database functions: [evidence span withheld — anchor status: not_found]
- [readme] H5AD import into PostgreSQL via GraphQL API: [evidence span withheld — anchor status: external_doc]
- [readme] web application that accesses a PostgreSQL database via GraphQL API: [evidence span withheld — anchor status: external_doc]
- [abstract] backend implemented in PostgreSQL, Python 3, and GraphQL: "The server backend is implemented in PostgreSQL, Python 3, and GraphQL"
