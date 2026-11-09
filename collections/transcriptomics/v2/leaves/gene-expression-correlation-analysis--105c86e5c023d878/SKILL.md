---
name: gene-expression-correlation-analysis
description: Use when a normalized single-cell expression matrix (imported from H5AD
  into Cellenium's PostgreSQL backend) is loaded for one study and you need to find
  all genes whose expression is highly correlated to a query gene, transcriptome-wide,
  within that study.
when_to_use_negative:
- The goal is cross-study (multi-study) correlation rather than correlation within
  a single study — Cellenium supports analysis within and across studies, but this
  skill covers the within-study co-expression view
- The goal is finding differentially expressed genes based on clusters of annotated
  cells — that is a separate Cellenium feature, not co-expression correlation
- The input is not a single-cell expression matrix (e.g. bulk RNA-seq or an already-computed
  feature/summary table); Cellenium imports scRNA expression data and cell annotations
  in H5AD format
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PostgreSQL
  role: Backend database that stores the imported single-cell expression matrix from
    which gene expression vectors are extracted
  repo: https://github.com/Bayer-Group/cellenium
- name: Python 3
  role: Server backend language whose stored procedures perform the transcriptome-wide
    within-study correlation computation
  repo: https://github.com/Bayer-Group/cellenium
- name: GraphQL
  role: API layer through which the correlation analysis and its results are queried
    from the PostgreSQL backend
  repo: https://github.com/Bayer-Group/cellenium
- name: ReactJS
  role: Frontend framework rendering the coexpression (scatter) plot view fed by the
    pairwise co-expression values
  repo: https://github.com/Bayer-Group/cellenium
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad349
    title: Cellenium—a scalable and interactive visual analytics app for exploring
      multimodal single-cell data
  source_content_hash: sha256:89f239a5770ca31fb1df891e7e4d1cda62785f3919238fd8afc665d9afa94975
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Bayer-Group/cellenium
  tools:
  - GraphQL
  - OnClass
  - PostgreSQL
  - Python
  - Python 3
  - ReactJS
  - TensorFlow
  - git
  - pip
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# gene-expression-correlation-analysis

## Summary

Within a single single-cell study loaded in Cellenium, compute a transcriptome-wide correlation of every gene's expression vector against a user-selected query gene across all cells, rank genes by correlation coefficient, and expose the pairwise values that drive the gene-pair coexpression (scatter) plot view.

## When to use

Use when a normalized single-cell expression matrix (imported from H5AD into Cellenium's PostgreSQL backend) is loaded for one study and you need to find all genes whose expression is highly correlated to a query gene, transcriptome-wide, within that study.

## When NOT to use

- The goal is cross-study (multi-study) correlation rather than correlation within a single study — Cellenium supports analysis within and across studies, but this skill covers the within-study co-expression view
- The goal is finding differentially expressed genes based on clusters of annotated cells — that is a separate Cellenium feature, not co-expression correlation
- The input is not a single-cell expression matrix (e.g. bulk RNA-seq or an already-computed feature/summary table); Cellenium imports scRNA expression data and cell annotations in H5AD format

## Inputs

- Single-cell RNA-seq expression matrix for one study, imported in H5AD format and stored in the Cellenium PostgreSQL database (e.g. the public blood_covid demo study)
- A query gene of interest (gene name/symbol) selected within the study
- Expression vectors of all other genes in the study across the same cells

## Outputs

- coexpression_ranked_table.csv: one row per analyzed gene with a correlation-coefficient column, ranked in descending order of correlation with the query gene
- Pairwise co-expression values feeding the gene-pair coexpression (scatter) plot view

## How to apply

Load the single-cell expression matrix for the study (e.g. the public blood_covid demo study imported from H5AD) from the Cellenium PostgreSQL backend via the Python 3 server backend, which is exposed through a GraphQL API. Select the query gene and extract its expression vector across all cells of the study together with the expression vectors of every other gene in the study. Compute a within-study correlation of each gene's expression vector against the query gene's vector across cells — Cellenium implements this server-side using Python stored procedures behind the GraphQL API. Rank all genes by their correlation coefficient with the query gene in descending order and write the ranked gene-by-correlation table to coexpression_ranked_table.csv, retaining the pairwise co-expression values that also feed the gene-pair coexpression (scatter) plot view.

## Related tools

- **PostgreSQL** (Backend database that stores the imported single-cell expression matrix from which gene expression vectors are extracted) — https://github.com/Bayer-Group/cellenium
- **Python 3** (Server backend language whose stored procedures perform the transcriptome-wide within-study correlation computation) — https://github.com/Bayer-Group/cellenium
- **GraphQL** (API layer through which the correlation analysis and its results are queried from the PostgreSQL backend) — https://github.com/Bayer-Group/cellenium
- **ReactJS** (Frontend framework rendering the coexpression (scatter) plot view fed by the pairwise co-expression values) — https://github.com/Bayer-Group/cellenium

## Evaluation signals

- The output file coexpression_ranked_table.csv exists and contains one row per analyzed gene
- The table includes a correlation-coefficient column and is ranked in descending order of correlation with the query gene
- All correlation coefficients fall within the mathematically bounded range [-1, 1]
- The ranked table retains pairwise co-expression values consistent with what the gene-pair coexpression (scatter) plot view displays
- The queried study is present in the PostgreSQL backend (e.g. the blood_covid demo study was imported) before running the analysis

## Limitations

- This is a within-study (single-study) analysis; correlated genes are computed only against cells of the same study, not across studies
- Study data must be pre-processed and imported first: H5AD expression data, normalization, differentially expressed genes, and UMAP projection are prepared via jupyter notebooks, and the full setup can take a couple of hours
- Correlation values alone do not establish causality or correct for confounding cell-type composition; results should be interpreted alongside cell-type annotations available in Cellenium

## Evidence

- [readme] Co-expression feature is a documented Cellenium capability: "find genes which expression is highly correlated to a query gene"
- [readme] Pairwise coexpression plots are part of the same feature set: "draw coexpression plots for pairs of genes, explore the cell types contained in the plots"
- [abstract] Backend technology stack implementing the analysis: "The server backend is implemented in PostgreSQL, Python 3, and GraphQL"
- [readme] Required input format and data import path: [evidence span withheld — anchor status: external_doc]
- [readme] Public demo study used for the within-study co-expression analysis: "UMAP projection cell type plot of the public study example blood_covid.ipynb"
