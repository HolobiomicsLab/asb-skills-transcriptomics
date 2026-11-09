---
name: anndata-expression-matrix-loading
description: Use when a scRNA-seq study is available as an AnnData/H5AD file (e.g.
  the blood_covid demo) and the cells x genes expression matrix must be restructured
  into a gene-centric sparse form for PostgreSQL-backed aggregation by cell type.
when_to_use_negative:
- The input is already an aggregated gene x cell-type expression table (e.g. a precomputed
  aggregated_expression.csv), so no sparse reshaping is needed.
- The data is not in AnnData/H5AD format and no Python 3 environment is available
  to load it.
- The goal is frontend visualization in the Cellenium ReactJS client rather than backend
  data ingestion and aggregation.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Python 3
  role: Loads the AnnData object, extracts the cells x genes matrix with cell-type
    annotations, converts it to a gene-centric sparse representation, and recomputes
    per-cell-type averages for verification
  repo: https://github.com/Bayer-Group/cellenium
- name: PostgreSQL
  role: Stores the gene-centric sparse expression table and runs GROUP BY aggregation
    over gene and annotated cell type via database background-job functions
  repo: https://github.com/Bayer-Group/cellenium
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad349
    title: Cellenium—a scalable and interactive visual analytics app for exploring
      multimodal single-cell data
  source_content_hash: sha256:222a4324be58d1f6a35d525d5c2c4a7e1e305fdb4ee299b6d505b4e4b26eded8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Bayer-Group/cellenium
  tools:
  - PostgreSQL
  - Python 3
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# anndata-expression-matrix-loading

## Summary

Loads single-cell expression data from an AnnData (H5AD) object, converts the cells x genes matrix into a gene-centric sparse representation, and materializes it in PostgreSQL for gene and cell-type level aggregation via database functions.

## When to use

Use when a scRNA-seq study is available as an AnnData/H5AD file (e.g. the blood_covid demo) and the cells x genes expression matrix must be restructured into a gene-centric sparse form for PostgreSQL-backed aggregation by cell type.

## When NOT to use

- The input is already an aggregated gene x cell-type expression table (e.g. a precomputed aggregated_expression.csv), so no sparse reshaping is needed.
- The data is not in AnnData/H5AD format and no Python 3 environment is available to load it.
- The goal is frontend visualization in the Cellenium ReactJS client rather than backend data ingestion and aggregation.

## Inputs

- AnnData / H5AD file of a single-cell RNA-seq study (e.g. blood_covid demo study)
- Cell-type annotations associated with the AnnData cells
- Cells x genes expression matrix extracted from the AnnData object

## Outputs

- PostgreSQL table of sparse expression values with gene, cell, and expression-value fields
- aggregated_expression.csv containing the gene x cell-type expression summary

## How to apply

Load the demo study's AnnData object (blood_covid.ipynb) in Python 3 and extract the cells x genes expression matrix together with the cell-type annotations. Convert the matrix into a gene-centric sparse representation so only non-zero expression entries are stored per gene, which keeps storage proportional to the number of expressed cells rather than cells x genes. Materialize the sparse values in a PostgreSQL table with gene, cell, and expression-value fields, mirroring how Cellenium imports scRNA expression data in H5AD format into its PostgreSQL backend. Aggregate expression with PostgreSQL database functions using GROUP BY over gene and annotated cell type to compute per-cell-type expression summaries, writing the result to aggregated_expression.csv. Verify correctness by recomputing per-cell-type averages directly from the AnnData matrix in Python 3 and comparing them against the PostgreSQL aggregation.

## Related tools

- **Python 3** (Loads the AnnData object, extracts the cells x genes matrix with cell-type annotations, converts it to a gene-centric sparse representation, and recomputes per-cell-type averages for verification) — https://github.com/Bayer-Group/cellenium
- **PostgreSQL** (Stores the gene-centric sparse expression table and runs GROUP BY aggregation over gene and annotated cell type via database background-job functions) — https://github.com/Bayer-Group/cellenium

## Examples

```
(cd data_import && PYTHONPATH=$(pwd) jupyter-lab)
```

## Evaluation signals

- Recomputed per-cell-type expression averages directly from the AnnData matrix in Python 3 must match the PostgreSQL GROUP BY aggregation results.
- The PostgreSQL table stores only non-zero expression entries per gene (gene, cell, expression-value fields), not a dense cells x genes matrix.
- aggregated_expression.csv has gene x cell-type shape, with one summary value per gene/cell-type pair.
- The H5AD import preserves the association between each cell and its cell-type annotation.

## Limitations

- Full-size studies (e.g. heart_failure_reichart2022) require editing the download URL in the notebook before processing; setup and study data processing 'initially ... will take a couple of hours to run'.
- During initial docker-compose startup, error messages about the missing 'postgraphile' user pile up until make reset_database is run and can be ignored only at that stage.
- CellO resource files require a manual workaround (downloading and extracting resources_v2.0.0.tar.gz) before setup.

## Evidence

- [readme] Cellenium imports scRNA expression data and cell annotations in H5AD format.: [evidence span withheld — anchor status: external_doc]
- [readme] Notebooks prepare publicly available scRNA studies including normalization and other study data needed by the app: [evidence span withheld — anchor status: external_doc]
- [abstract] The server backend stack that stores and aggregates the expression data: "The server backend is implemented in PostgreSQL, Python 3, and GraphQL"
