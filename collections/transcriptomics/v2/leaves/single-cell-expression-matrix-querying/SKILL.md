---
name: single-cell-expression-matrix-querying
description: Use when a single-cell scRNA-seq study (e.g. the blood_covid demo, imported
  as H5AD) is loaded in Cellenium and you need the genes whose expression correlates
  with a query gene across all cells of that study.
when_to_use_negative:
- The analysis goal is across-study comparison rather than within a single study —
  Cellenium separates within-study from cross-study analysis and this skill covers
  only the single-study co-expression view.
- The study's expression matrix has not been preprocessed and imported into the Cellenium
  PostgreSQL backend (H5AD import plus notebook-computed DE genes/UMAP are prerequisites).
- The query gene is not present in the study's imported gene set — no expression vector
  can be extracted.
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: PostgreSQL
  role: Server backend that stores the imported single-cell expression matrix and
    serves gene expression vectors for the study
  repo: https://github.com/Bayer-Group/cellenium
- name: Python 3
  role: Implements stored procedures used for server-side computation, including the
    transcriptome-wide correlation of gene expression vectors against the query gene
  repo: https://github.com/Bayer-Group/cellenium
- name: GraphQL
  role: API layer through which the expression matrix and study data are queried (example
    queries in documentation/graphql_api_usage)
  repo: https://github.com/Bayer-Group/cellenium
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad349
    title: Cellenium—a scalable and interactive visual analytics app for exploring
      multimodal single-cell data
  source_content_hash: sha256:61b96a6051a304382b06266d21af20c0402366d69c11c42a9a088fa47f7bd4b9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Bayer-Group/cellenium
  tools:
  - GraphQL
  - PostgreSQL
  - Python 3
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# single-cell-expression-matrix-querying

## Summary

Retrieves per-gene expression vectors from a single-cell RNA-seq expression matrix stored in Cellenium's PostgreSQL backend and computes transcriptome-wide correlations against a query gene within a single study, feeding the coexpression scatter-plot view and a ranked gene list.

## When to use

Use when a single-cell scRNA-seq study (e.g. the blood_covid demo, imported as H5AD) is loaded in Cellenium and you need the genes whose expression correlates with a query gene across all cells of that study.

## When NOT to use

- The analysis goal is across-study comparison rather than within a single study — Cellenium separates within-study from cross-study analysis and this skill covers only the single-study co-expression view.
- The study's expression matrix has not been preprocessed and imported into the Cellenium PostgreSQL backend (H5AD import plus notebook-computed DE genes/UMAP are prerequisites).
- The query gene is not present in the study's imported gene set — no expression vector can be extracted.

## Inputs

- Single-cell expression matrix for one imported study (H5AD-derived, loaded into the Cellenium PostgreSQL backend)
- A query gene of interest (gene name/symbol present in the study's feature set)

## Outputs

- coexpression_ranked_table.csv — one row per analyzed gene with a correlation-coefficient column, ranked in descending order
- Pairwise coexpression values feeding the gene-pair coexpression (scatter) plot view

## How to apply

Load the single-cell expression matrix for the target study (e.g. the public blood_covid demo) from the Cellenium PostgreSQL backend, accessed via the GraphQL API and Python 3 stored procedures. Select a query gene and extract its expression vector across all cells, together with the expression vectors of every other gene in the study. Compute a within-study correlation of each gene's vector against the query gene's vector across cells, then rank all genes by correlation coefficient in descending order. Write the ranked gene-by-correlation table to coexpression_ranked_table.csv; the same pairwise values feed the gene-pair coexpression (scatter) plot view. Note that Cellenium expects studies to be preprocessed (normalized) and imported in H5AD format before querying.

## Related tools

- **PostgreSQL** (Server backend that stores the imported single-cell expression matrix and serves gene expression vectors for the study) — https://github.com/Bayer-Group/cellenium
- **Python 3** (Implements stored procedures used for server-side computation, including the transcriptome-wide correlation of gene expression vectors against the query gene) — https://github.com/Bayer-Group/cellenium
- **GraphQL** (API layer through which the expression matrix and study data are queried (example queries in documentation/graphql_api_usage)) — https://github.com/Bayer-Group/cellenium

## Examples

```
corr = expr.corrwith(query_expr, axis=0, method='pearson').sort_values(ascending=False); corr.to_csv('coexpression_ranked_table.csv')
```

## Evaluation signals

- coexpression_ranked_table.csv exists and contains one row per analyzed gene in the study.
- Every correlation coefficient falls within the mathematically bounded range [-1, 1].
- The table is ranked in descending order of correlation coefficient with the query gene first-documented ranking preserved.
- The pairwise co-expression values in the table reproduce the values plotted in the gene-pair coexpression (scatter) view.
- The query study is the intended one (e.g. blood_covid demo) — verify via the Cellenium study metadata served through GraphQL.

## Limitations

- Correlation is computed within a single study only; cross-study correlated-gene search is a separate Cellenium feature.
- Studies must be preprocessed via the provided Jupyter notebooks (normalization, DE genes, UMAP) and imported as H5AD before the expression matrix can be queried; initially the full setup takes a couple of hours to run.
- Server-side rendered plot features depend on Python stored procedures in the backend, so the PostgreSQL + Python + GraphQL stack must be running (e.g. via docker compose up).

## Evidence

- [abstract] The server backend is implemented in PostgreSQL, Python 3, and GraphQL: "The server backend is implemented in PostgreSQL, Python 3, and GraphQL"
- [intro] draw coexpression plots for pairs of genes and explore the cell types contained in the plots: "draw coexpression plots for pairs of genes, explore the cell types contained in the plots"
- [readme] Cellenium imports scRNA expression data and cell annotations in H5AD format: [evidence span withheld — anchor status: external_doc]
- [readme] find genes which expression is highly correlated to a query gene: "find genes which expression is highly correlated to a query gene"
- [readme] The graphql_api_usage folder contains a couple of example queries to illustrate the API capabilities: "The [graphql_api_usage](documentation/graphql_api_usage/) folder
contains a couple of example queries to illustrate the API capabilities."
