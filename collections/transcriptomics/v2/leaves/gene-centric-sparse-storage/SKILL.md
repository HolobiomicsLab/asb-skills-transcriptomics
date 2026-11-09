---
name: gene-centric-sparse-storage
description: Use when a large single-cell expression matrix (AnnData/H5AD) must be
  persisted in a relational database so per-gene and per-cell-type expression queries
  run efficiently server-side.
when_to_use_negative:
- Input is already an aggregated gene x cell-type feature table with no single-cell
  resolution left
- The dataset is small enough for in-memory analysis in Python and no cross-study
  server-side querying is needed
- The expression matrix is fully dense (nearly every gene expressed in every cell),
  so sparsification gives no storage benefit
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: PostgreSQL
  role: Stores the gene-centric sparse expression table and executes GROUP BY aggregations
    over gene and cell type as database background-job functions
  repo: https://github.com/Bayer-Group/cellenium
- name: Python 3
  role: Loads the AnnData object, extracts the cells x genes matrix and cell-type
    annotations, converts to sparse representation, and cross-checks aggregation results
  repo: https://github.com/Bayer-Group/cellenium
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad349
    title: Cellenium—a scalable and interactive visual analytics app for exploring
      multimodal single-cell data
  source_content_hash: sha256:77fa644416b8c1a66e90112ce72bd0b5ee32e863535fb1fce838e75defc8a650
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# gene-centric-sparse-storage

## Summary

Represent a cells x genes single-cell expression matrix as a gene-centric sparse table (only non-zero entries stored per gene) in PostgreSQL, enabling efficient gene- and cell-type-level aggregation via database functions as done in Cellenium.

## When to use

Use when a large single-cell expression matrix (AnnData/H5AD) must be persisted in a relational database so per-gene and per-cell-type expression queries run efficiently server-side.

## When NOT to use

- Input is already an aggregated gene x cell-type feature table with no single-cell resolution left
- The dataset is small enough for in-memory analysis in Python and no cross-study server-side querying is needed
- The expression matrix is fully dense (nearly every gene expressed in every cell), so sparsification gives no storage benefit

## Inputs

- AnnData / H5AD single-cell expression matrix (cells x genes)
- Cell-type annotation table keyed by cell barcode
- PostgreSQL database with a schema for sparse expression values (gene, cell, expression value)

## Outputs

- PostgreSQL sparse expression table with gene, cell, and expression-value fields
- Gene x cell-type aggregated expression table (e.g. aggregated_expression.csv)
- Per-cell-type expression summaries computed via GROUP BY aggregation

## How to apply

Load the study's AnnData (H5AD) object in Python 3 and extract the cells x genes expression matrix together with the cell-type annotations. Convert the dense matrix into a gene-centric sparse representation so that only non-zero expression entries are kept per gene, then materialize these values in a PostgreSQL table with gene, cell, and expression-value fields. Aggregate expression with PostgreSQL database functions using GROUP BY over gene and annotated cell type to compute per-cell-type expression summaries; in Cellenium these aggregations are core API features implemented as PostgreSQL functions running as database background jobs. Verify correctness by recomputing per-cell-type expression averages directly from the AnnData matrix in Python 3 and comparing them against the PostgreSQL aggregation results (e.g. writing the gene x cell-type table to aggregated_expression.csv).

## Related tools

- **PostgreSQL** (Stores the gene-centric sparse expression table and executes GROUP BY aggregations over gene and cell type as database background-job functions) — https://github.com/Bayer-Group/cellenium
- **Python 3** (Loads the AnnData object, extracts the cells x genes matrix and cell-type annotations, converts to sparse representation, and cross-checks aggregation results) — https://github.com/Bayer-Group/cellenium

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('blood_covid.h5ad'); X = adata.X.tocsr(); df = X.tocoo(); pg_df = pd.DataFrame({'gene': adata.var_names[df.col], 'cell': adata.obs_names[df.row], 'value': df.data})
```

## Evaluation signals

- Every row in the PostgreSQL sparse table has non-zero expression values; zero entries are absent
- Per-cell-type averages recomputed from the PostgreSQL GROUP BY aggregation match per-cell-type averages recomputed directly from the AnnData matrix in Python 3
- Number of distinct genes in the sparse table equals the number of genes in the source AnnData object
- Gene x cell-type output table (aggregated_expression.csv) has one row per gene and one column per annotated cell type with no missing aggregates

## Limitations

- Initial study data import and processing of notebooks (docker compose up, make reset_database/test_studydata_import) takes a couple of hours for real-life studies with full amounts of cells and genes
- Aggregation is only meaningful for studies whose H5AD files have been normalized and preprocessed with UMAP projections and differentially expressed genes computed during Cellenium's data-import step
- Zero-expression entries are not stored, so queries that need explicit zeros must be handled by the calling logic

## Evidence

- [results] sparse-storage finding from task card: [evidence span withheld — anchor status: not_found]
- [abstract] backend stack: "The server backend is implemented in PostgreSQL, Python 3, and GraphQL"
- [readme] import format and preprocessing: [evidence span withheld — anchor status: external_doc]
- [readme] database access architecture: [evidence span withheld — anchor status: external_doc]
