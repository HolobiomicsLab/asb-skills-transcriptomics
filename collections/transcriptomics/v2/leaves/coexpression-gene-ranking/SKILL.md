---
name: coexpression-gene-ranking
description: Use when a single-cell expression matrix (e.g., H5AD-imported scRNA-seq
  study like the blood_covid demo) is loaded and you need to identify genes whose
  expression is highly correlated to a query gene across all cells.
when_to_use_negative:
- The goal is differential expression between clusters of annotated cells rather than
  transcriptome-wide co-expression with a query gene
- The goal is qualitative comparison of marker gene expression across multiple studies
  rather than within a single study
- Input data is not a cell-by-gene single-cell expression matrix (e.g., bulk RNA-seq
  or already-summarized feature tables)
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: PostgreSQL
  role: Server backend database that stores the single-cell expression matrix from
    which per-gene expression vectors across all cells are extracted
  repo: https://github.com/Bayer-Group/cellenium
- name: Python 3
  role: Server backend language used to implement stored-procedure features such as
    the correlation computation over gene expression vectors
  repo: https://github.com/Bayer-Group/cellenium
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad349
    title: Cellenium—a scalable and interactive visual analytics app for exploring
      multimodal single-cell data
  source_content_hash: sha256:a613a5f961c20a2b9394b1f974bc63d8287f50dafd73480e0ac86c02b6f9ee4d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Bayer-Group/cellenium
  tools:
  - PostgreSQL
  - Python 3
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# coexpression-gene-ranking

## Summary

Rank all genes in a single-cell study by the correlation of their expression vectors with a query gene's expression vector across cells, reproducing Cellenium's single-study co-expression analysis that also underlies the pairwise gene coexpression (scatter) plot view.

## When to use

Use when a single-cell expression matrix (e.g., H5AD-imported scRNA-seq study like the blood_covid demo) is loaded and you need to identify genes whose expression is highly correlated to a query gene across all cells.

## When NOT to use

- The goal is differential expression between clusters of annotated cells rather than transcriptome-wide co-expression with a query gene
- The goal is qualitative comparison of marker gene expression across multiple studies rather than within a single study
- Input data is not a cell-by-gene single-cell expression matrix (e.g., bulk RNA-seq or already-summarized feature tables)

## Inputs

- Single-cell expression matrix for one study imported in H5AD format into the Cellenium PostgreSQL database (e.g., the public blood_covid demo study)
- A query gene identifier present in the study's gene set

## Outputs

- coexpression_ranked_table.csv — one row per analyzed gene with a correlation-coefficient column, ranked in descending order of correlation with the query gene
- Pairwise co-expression values for the query gene versus other genes, usable for the gene-pair coexpression (scatter) plot view

## How to apply

Load the single-cell expression matrix for one study (e.g., the public blood_covid demo) from the Cellenium PostgreSQL backend via the Python 3 server backend. Select a query gene and extract its expression vector across all cells together with the expression vectors of every other gene in the study. Compute a within-study correlation of each gene's vector against the query gene's vector across cells — the same pairwise co-expression values that feed Cellenium's gene-pair coexpression (scatter) plot view. Rank all genes by correlation coefficient in descending order and write the ranked gene-by-correlation table to coexpression_ranked_table.csv. This mirrors the README-listed Cellenium capability to 'find genes which expression is highly correlated to a query gene'.

## Related tools

- **PostgreSQL** (Server backend database that stores the single-cell expression matrix from which per-gene expression vectors across all cells are extracted) — https://github.com/Bayer-Group/cellenium
- **Python 3** (Server backend language used to implement stored-procedure features such as the correlation computation over gene expression vectors) — https://github.com/Bayer-Group/cellenium

## Examples

```
python analyze_coexpression.py --study blood_covid --query-gene CXCL10 --out coexpression_ranked_table.csv
```

## Evaluation signals

- The output file coexpression_ranked_table.csv exists and contains one row per analyzed gene in the study
- The table has a correlation-coefficient column and all coefficients fall within the mathematically bounded range [-1, 1]
- Rows are ranked in descending order of correlation coefficient with the query gene
- The query gene itself should appear in the table with a correlation coefficient of 1 (self-correlation)
- Pairwise co-expression values in the table match the values displayed by the gene-pair coexpression (scatter) plot view

## Limitations

- The analysis is within a single study only; cross-study correlation is a different Cellenium feature and is not covered by this workflow
- Cellenium requires preprocessed study data — differential expression, UMAP projection and other study data must be computed in the data-import notebooks before features like this work
- Dockerized setup via docker-compose is required; initially, error messages about the missing 'postgraphile' user can pile up until the database reset step and can be ignored

## Evidence

- [abstract] abstract: "The server backend is implemented in PostgreSQL, Python 3, and GraphQL"
- [intro] intro: "draw coexpression plots for pairs of genes, explore the cell types contained in the plots"
- [readme] readme: "find genes which expression is highly correlated to a query gene"
