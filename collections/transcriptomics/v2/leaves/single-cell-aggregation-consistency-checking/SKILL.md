---
name: single-cell-aggregation-consistency-checking
description: Use when verifying that a database-side aggregation of single-cell expression
  data (e.g. per gene and cell type averages from a sparse PostgreSQL table) matches
  values computed directly from the original AnnData (H5AD) expression matrix.
when_to_use_negative:
- Input data has not been imported into a database in gene-centric sparse form — computing
  averages only in Python from AnnData does not exercise the database aggregation
  path
- The analysis goal is interactive visual exploration (UMAP, violin plots, coexpression)
  rather than validating aggregation correctness — use the Cellenium webapp instead
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: PostgreSQL
  role: Stores the gene-centric sparse expression table and runs the GROUP BY aggregation
    over gene and cell type via database functions as background jobs
  repo: https://github.com/Bayer-Group/cellenium
- name: Python 3
  role: Loads the blood_covid AnnData object, extracts the expression matrix and cell-type
    annotations, builds the sparse gene-centric representation, and recomputes per-cell-type
    averages for verification
  repo: https://github.com/Bayer-Group/cellenium
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad349
    title: Cellenium—a scalable and interactive visual analytics app for exploring
      multimodal single-cell data
  source_content_hash: sha256:85b31e95e433ecb5d5889f4e75a52e113fa9ea3a35d8b52f8ed9f12d661d6bd0
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# single-cell-aggregation-consistency-checking

## Summary

Reconstructs Cellenium's gene-centric sparse storage of single-cell expression data in PostgreSQL and validates database-side gene/cell-type aggregation against recomputed averages from the source AnnData matrix.

## When to use

Use when verifying that a database-side aggregation of single-cell expression data (e.g. per gene and cell type averages from a sparse PostgreSQL table) matches values computed directly from the original AnnData (H5AD) expression matrix.

## When NOT to use

- Input data has not been imported into a database in gene-centric sparse form — computing averages only in Python from AnnData does not exercise the database aggregation path
- The analysis goal is interactive visual exploration (UMAP, violin plots, coexpression) rather than validating aggregation correctness — use the Cellenium webapp instead

## Inputs

- AnnData (H5AD) object from the blood_covid demo study (blood_covid.ipynb) containing a cells x genes expression matrix
- Cell-type annotations per cell from the AnnData object
- PostgreSQL table with gene, cell, and expression-value fields holding the gene-centric sparse expression values

## Outputs

- Gene x cell-type aggregated expression table (e.g. aggregated_expression.csv) computed by PostgreSQL GROUP BY aggregation
- Per-cell-type expression averages recomputed directly from the AnnData matrix in Python 3
- Consistency comparison between PostgreSQL aggregation results and Python-recomputed averages

## How to apply

Load the blood_covid demo study notebook (blood_covid.ipynb) AnnData object in Python 3 and extract the cells x genes expression matrix together with cell-type annotations. Convert the matrix to a gene-centric sparse representation so only non-zero expression entries are stored per gene, then materialize those values in a PostgreSQL table with gene, cell, and expression-value fields. Aggregate expression using PostgreSQL database functions with GROUP BY over gene and annotated cell type — mirroring how Cellenium implements gene and cell-type level aggregation as core API features running as database background jobs. Finally, recompute per-cell-type expression averages directly from the AnnData matrix in Python 3 and compare them against the PostgreSQL aggregation results to confirm consistency.

## Related tools

- **PostgreSQL** (Stores the gene-centric sparse expression table and runs the GROUP BY aggregation over gene and cell type via database functions as background jobs) — https://github.com/Bayer-Group/cellenium
- **Python 3** (Loads the blood_covid AnnData object, extracts the expression matrix and cell-type annotations, builds the sparse gene-centric representation, and recomputes per-cell-type averages for verification) — https://github.com/Bayer-Group/cellenium

## Examples

```
make reset_database test_studydata_import
```

## Evaluation signals

- Per-cell-type expression averages recomputed directly from the AnnData matrix match the PostgreSQL GROUP BY aggregation results
- The sparse PostgreSQL table contains only non-zero expression entries per gene (gene-centric sparse representation respected)
- The aggregated gene x cell-type table covers every gene and annotated cell type present in the source AnnData object
- Cell counts per cell type in the aggregation denominators equal the number of annotated cells in the AnnData object

## Limitations

- Aggregation depends on cell-type annotation quality; Cellenium notes an interactive cell annotation feature allows adding user-defined cell types, so mismatched or user-defined annotations change the aggregation groups
- Full-size real-life studies (normal_studydata_import) contain full amounts of cells and genes, so initial import and database aggregation can take a couple of hours to run
- Initially, error messages about the missing 'postgraphile' user may pile up until the 'make reset_database' step is run; these can be ignored

## Evidence

- [other] Cellenium implements aggregation of expression data on gene and cell-type level as core API features implemented by PostgreSQL database functions: [evidence span withheld — anchor status: not_found]
- [intro] the public study example blood_covid.ipynb: "the public study example blood_covid.ipynb"
- [readme] Cellenium imports scRNA expression data and cell annotations in H5AD format.: [evidence span withheld — anchor status: external_doc]
- [readme] Cellenium is a web application that accesses a PostgreSQL database via GraphQL API.: [evidence span withheld — anchor status: external_doc]
