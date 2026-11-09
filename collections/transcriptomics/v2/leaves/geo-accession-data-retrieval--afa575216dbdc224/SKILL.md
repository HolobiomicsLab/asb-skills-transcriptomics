---
name: geo-accession-data-retrieval
description: Use when a benchmark or reproduction task names a published dataset only
  by its GEO accession code (e.g., GSE125970 for colon epithelial cells) and the original
  count matrix must be obtained before scoring cells.
when_to_use_negative:
- The count matrix is already available locally in the genes-by-cells format expected
  by ORIGINS' activity() function
- The study of interest does not deposit its data under a GEO accession code (the
  article's datasets are all GEO-hosted)
- A raw-read reprocessing is required and only the authors' precomputed matrix is
  needed otherwise
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: R
  role: Environment in which the retrieved GEO count matrix is loaded as an expression
    matrix (rows = genes, columns = cells) prior to scoring
- name: ORIGINS
  role: Downstream consumer of the retrieved expression matrix, computing the differentiation-activity
    score with activity(expression_matrix, differentiation_edges)
  repo: https://github.com/danielasenraoka/ORIGINS
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.mex.2022.101778
    title: 'ORIGINS: A protein network-based approach to quantify cell pluripotency
      from scRNA-seq data'
  source_content_hash: sha256:05772bb8c4e252ea3a4cb321d8784831b66336cae7a60f3167ed1075292c617e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/danielasenraoka/ORIGINS
  tools:
  - ORIGINS
  - R
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# geo-accession-data-retrieval

## Summary

Locate and download published scRNA-seq count matrices from the GEO database using accession codes cited in a paper (e.g., GSE125970 for colon epithelial cells), so that downstream tools like ORIGINS, LandSCENT and CytoTRACE can be run on the original data.

## When to use

Use when a benchmark or reproduction task names a published dataset only by its GEO accession code (e.g., GSE125970 for colon epithelial cells) and the original count matrix must be obtained before scoring cells.

## When NOT to use

- The count matrix is already available locally in the genes-by-cells format expected by ORIGINS' activity() function
- The study of interest does not deposit its data under a GEO accession code (the article's datasets are all GEO-hosted)
- A raw-read reprocessing is required and only the authors' precomputed matrix is needed otherwise

## Inputs

- GEO accession code (e.g., GSE125970)
- Downloaded scRNA-seq count/supplementary matrix from the GEO record
- Cell-type annotation metadata supplied with the GEO deposit

## Outputs

- R expression matrix or data frame with rows = genes and columns = cells
- Per-cell annotations usable for aggregating scores by cell type

## How to apply

Extract the GEO accession codes from the paper's supplementary section, which maps each code to its tissue (GSE113197 breast epithelial cells, GSE125970 colon epithelial cells, GSE117498 hematopoietic bone marrow cells, GSE161383 lung cells). Retrieve the corresponding count matrix from GEO and load it into R as an expression matrix where each row is a gene and each column is a cell, matching the format expected by the ORIGINS activity() function. Confirm that per-cell annotations (e.g., stem cells, transit-amplifying cells, differentiated lineages for Colon-2) accompany the matrix so per-type averages can be computed later. The retrieved matrix then feeds directly into remotes::install_github("danielasenraoka/ORIGINS")-installed tooling for differentiation-activity scoring.

## Related tools

- **R** (Environment in which the retrieved GEO count matrix is loaded as an expression matrix (rows = genes, columns = cells) prior to scoring)
- **ORIGINS** (Downstream consumer of the retrieved expression matrix, computing the differentiation-activity score with activity(expression_matrix, differentiation_edges)) — https://github.com/danielasenraoka/ORIGINS

## Evaluation signals

- The accession code resolves to the correct study on the GEO database (e.g., GSE125970 returns the colon epithelial cells dataset)
- The loaded object is a matrix/data frame with rows = genes and columns = cells, matching the ORIGINS input convention
- Cell-type annotations are present so that average scores per annotated type (stem, TA, differentiated) can be aggregated with dplyr
- Downstream ORIGINS activity() accepts the matrix without dimension or identifier errors

## Limitations

- The accession-to-tissue mapping is only stated in the paper's supplementary section; misreading it retrieves the wrong tissue dataset
- The article benchmarks only four healthy human datasets (breast, colon, hematopoietic, lung), so accession-based reproduction outside these tissues is not validated by this paper
- GEO supplementary matrices may need reformatting before they conform to the genes-by-cells layout the tools expect

## Evidence

- [supplementary] available on the GEO database under the accession codes: GSE113197 for breast epithelial cells: "available on the GEO database under the accession codes: GSE113197 for breast epithelial cells"
- [supplementary] GSE125970 for colon epithelial cells: "GSE125970 for colon epithelial cells"
- [readme] expression_matrix is a matrix or data frame, where each row is a gene and each column is a cell.: "expression_matrix is a matrix or data frame, where each row is a gene and each column is a cell."
