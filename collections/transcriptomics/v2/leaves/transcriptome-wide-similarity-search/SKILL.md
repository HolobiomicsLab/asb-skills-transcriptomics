---
name: transcriptome-wide-similarity-search
description: Use when you have a single-cell expression matrix for one study (e.g.
  H5AD-imported study in Cellenium) and want to find, for a query gene, all genes
  with correlated expression across cells.
when_to_use_negative:
- You need co-expression across multiple studies rather than a within-study analysis
  — Cellenium also offers cross-study comparison, which is a different view
- You only need expression of one or a few genes in UMAP or violin plots, not transcriptome-wide
  correlated genes
- You are looking for differentially expressed genes between annotated cell clusters,
  not genes correlated with a query gene
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Python 3
  role: Server-backend language in which the within-study correlation of each gene
    against the query gene is computed over the expression vectors
  repo: https://github.com/Bayer-Group/cellenium
- name: PostgreSQL
  role: Backend database that stores the study expression matrix and serves gene expression
    vectors to the analysis
  repo: https://github.com/Bayer-Group/cellenium
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad349
    title: Cellenium—a scalable and interactive visual analytics app for exploring
      multimodal single-cell data
  source_content_hash: sha256:bcb448f84039726e49ae8f9727d2782c2f488670d96c156e5404a4748264d82f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Bayer-Group/cellenium
  tools:
  - PostgreSQL
  - Python 3
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# transcriptome-wide-similarity-search

## Summary

Identifies genes whose single-cell expression profiles are correlated with a query gene of interest within one study, as enabled by Cellenium's single-study co-expression view, and returns a transcriptome-wide ranked gene-by-correlation table.

## When to use

Use when you have a single-cell expression matrix for one study (e.g. H5AD-imported study in Cellenium) and want to find, for a query gene, all genes with correlated expression across cells.

## When NOT to use

- You need co-expression across multiple studies rather than a within-study analysis — Cellenium also offers cross-study comparison, which is a different view
- You only need expression of one or a few genes in UMAP or violin plots, not transcriptome-wide correlated genes
- You are looking for differentially expressed genes between annotated cell clusters, not genes correlated with a query gene

## Inputs

- Single-cell expression matrix for one study (H5AD-imported, served from the Cellenium PostgreSQL backend)
- A query gene of interest selected from the study's gene set

## Outputs

- coexpression_ranked_table.csv — ranked gene-by-correlation table (one row per analyzed gene, descending correlation coefficient)
- Pairwise co-expression values feeding the gene-pair coexpression (scatter) plot view

## How to apply

Load the single-cell expression matrix for the study (e.g. the public blood_covid demo study) from the Cellenium PostgreSQL backend via the Python 3 server backend. Select the query gene and extract its expression vector across all cells together with the expression vectors of every other gene in the study. Compute, transcriptome-wide, the within-study correlation of each gene's vector against the query gene's vector across cells, then rank all genes by correlation coefficient in descending order. Write the ranked gene-by-correlation table to coexpression_ranked_table.csv; the pairwise values also feed the gene-pair coexpression (scatter) plot view. Validate that the output has one row per analyzed gene, contains a correlation-coefficient column, and that all coefficients fall within [-1, 1].

## Related tools

- **Python 3** (Server-backend language in which the within-study correlation of each gene against the query gene is computed over the expression vectors) — https://github.com/Bayer-Group/cellenium
- **PostgreSQL** (Backend database that stores the study expression matrix and serves gene expression vectors to the analysis) — https://github.com/Bayer-Group/cellenium

## Examples

```
from scipy.stats import pearsonr; ranked = sorted(((g, pearsonr(query_vec, vec)[0]) for g, vec in gene_vectors.items()), key=lambda t: -t[1])
```

## Evaluation signals

- coexpression_ranked_table.csv exists and contains one row per analyzed gene
- The table includes a correlation-coefficient column
- All correlation coefficients fall within the mathematically bounded range [-1, 1]
- Genes are ranked by correlation coefficient in descending order, with the query gene's most correlated partners at the top
- The retained pairwise values are consistent with the values shown in the gene-pair coexpression (scatter) plot view

## Limitations

- The correlation is computed within a single study only; it does not by itself establish cross-study generality
- The setup requires the study's precomputed study data (e.g. differentially expressed genes, UMAP projection) to have been imported; initial study processing via the provided notebooks can take a couple of hours for full-size studies
- Initial docker-based setup shows error messages about the missing 'postgraphile' user until the database reset step is run; these can be ignored but may confuse first-time users

## Evidence

- [readme] README quote on correlated-gene feature: "find genes which expression is highly correlated to a query gene"
- [readme] README quote on coexpression plots: "draw coexpression plots for pairs of genes, explore the cell types contained in the plots"
- [abstract] Abstract quote on backend implementation: "The server backend is implemented in PostgreSQL, Python 3, and GraphQL"
- [readme] README quote on data access architecture: [evidence span withheld — anchor status: external_doc]
- [readme] README quote on public example study: "the public study example blood_covid.ipynb"
