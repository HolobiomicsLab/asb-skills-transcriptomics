---
name: geo-count-matrix-loading
description: Use when a downstream tool requires a raw/normalized scRNA-seq count
  matrix from GEO (genes as rows, cells as columns) as its expression_matrix input,
  e.g. before running ORIGINS activity or benchmarking against LandSCENT/CytoTRACE.
when_to_use_negative:
- Input is already a per-cell score vector, cluster assignment, or trajectory pseudotime
  rather than a gene-by-cell count/expression matrix.
- The target analysis uses ORIGINS2's build_ppin() workflow on a custom gene list
  rather than loading a GEO dataset for differentiation activity.
- Data are not from a GEO-deposited scRNA-seq study or lack gene-level identifiers
  needed to align rows with the differentiation PPIN.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_3934
tools:
- name: ORIGINS
  role: Consumes the loaded expression_matrix together with the bundled differentiation_edges
    PPIN to compute per-cell differentiation activity
  repo: https://github.com/danielasenraoka/ORIGINS
- name: R
  role: Environment in which GEO count data are loaded as a matrix/data frame and
    ORIGINS is installed and run
- name: remotes
  role: R package used to install ORIGINS from GitHub before loading the data
- name: ORIGINS2
  role: Extended version that accepts a normalized expression matrix and includes
    PPINs for 18 biological processes
  repo: https://github.com/danielasenraoka/ORIGINS2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.mex.2022.101778
    title: 'ORIGINS: A protein network-based approach to quantify cell pluripotency
      from scRNA-seq data'
  source_content_hash: sha256:2992b7dc20acb05d346463d809cd60c4856f852e15df7e6a97aa97d470071cb8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/danielasenraoka/ORIGINS
  tools:
  - ORIGINS
  - ORIGINS2
  - R
  - remotes
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_3934
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# geo-count-matrix-loading

## Summary

Load GEO-deposited scRNA-seq count matrices (e.g., GSE113197 breast, GSE125970 colon, GSE117498 hematopoietic, GSE161383 lung) as R expression matrices with genes as rows and cells as columns, ready for ORIGINS activity() computation.

## When to use

Use when a downstream tool requires a raw/normalized scRNA-seq count matrix from GEO (genes as rows, cells as columns) as its expression_matrix input, e.g. before running ORIGINS activity or benchmarking against LandSCENT/CytoTRACE.

## When NOT to use

- Input is already a per-cell score vector, cluster assignment, or trajectory pseudotime rather than a gene-by-cell count/expression matrix.
- The target analysis uses ORIGINS2's build_ppin() workflow on a custom gene list rather than loading a GEO dataset for differentiation activity.
- Data are not from a GEO-deposited scRNA-seq study or lack gene-level identifiers needed to align rows with the differentiation PPIN.

## Inputs

- GEO accession count matrices: GSE113197 (breast epithelial cells), GSE125970 (colon epithelial cells), GSE117498 (hematopoietic bone marrow cells), GSE161383 (lung cells)
- expression_matrix: R matrix or data frame with genes as rows and cells as columns (normalized matrix for ORIGINS2)

## Outputs

- expression_matrix object in R ready to pass as the first argument of activity(expression_matrix, differentiation_edges)
- diff_activity: per-cell differentiation activity score vector (e.g., 26.6 Kb activity vector for the breast dataset)

## How to apply

Download the four GEO datasets by accession (GSE113197 breast epithelial cells, GSE125970 colon epithelial cells, GSE117498 hematopoietic bone marrow cells, GSE161383 lung cells) and load each into R as an expression_matrix where each row is a gene and each column is a cell, matching the orientation required by the ORIGINS activity() function. Ensure the remotes package is installed (install.packages("remotes")) so that remotes::install_github("danielasenraoka/ORIGINS") succeeds and the differentiation_edges PPIN bundled in the package data directory can be paired with the loaded matrix. For ORIGINS2, supply a normalized expression matrix and optionally toggle the normalization parameter (FALSE returns the sum of all PPIN edges, TRUE normalizes by the number of edges). Verify orientation and gene identifier consistency (gene rows) before calling diff_activity <- activity(expression_matrix, differentiation_edges).

## Related tools

- **ORIGINS** (Consumes the loaded expression_matrix together with the bundled differentiation_edges PPIN to compute per-cell differentiation activity) — https://github.com/danielasenraoka/ORIGINS
- **R** (Environment in which GEO count data are loaded as a matrix/data frame and ORIGINS is installed and run)
- **remotes** (R package used to install ORIGINS from GitHub before loading the data)
- **ORIGINS2** (Extended version that accepts a normalized expression matrix and includes PPINs for 18 biological processes) — https://github.com/danielasenraoka/ORIGINS2

## Examples

```
install.packages("remotes"); remotes::install_github("danielasenraoka/ORIGINS"); library(ORIGINS); diff_activity <- activity(expression_matrix, differentiation_edges)
```

## Evaluation signals

- Matrix orientation check: number of rows equals number of genes and number of columns equals number of cells, matching the activity() convention 'each row is a gene and each column is a cell'.
- Downstream call succeeds: diff_activity <- activity(expression_matrix, differentiation_edges) returns a per-cell score vector without gene-identifier alignment errors.
- Output magnitude sanity: measured ORIGINS activity values are within the same order of magnitude as the reported benchmark values (breast runtime 3.43 h, extra RAM 6.4 Gb for LandSCENT vs 26.6 Kb activity vector for ORIGINS).
- Correct accession-to-tissue mapping: GSE113197 = breast, GSE125970 = colon, GSE117498 = hematopoietic, GSE161383 = lung.
- For ORIGINS2, the normalization parameter behaves as documented: FALSE yields the raw sum of all PPIN edges, TRUE yields activity normalized by the number of PPIN edges.

## Limitations

- ORIGINS (original package) is specific to the differentiation PPIN; other biological processes require the extended ORIGINS2 package or PyOrigins.
- ORIGINS2 expects a normalized expression matrix; supplying raw counts changes whether edge-sum vs edge-normalized activity is returned (normalization parameter FALSE/TRUE).
- Gene rows must be compatible with the differentiation PPIN bundled in the package data directory; mismatched gene identifiers will prevent correct activity computation.

## Evidence

- [supplementary] available on the GEO database under the accession codes: GSE113197 for breast epithelial cells: "available on the GEO database under the accession codes: GSE113197 for breast epithelial cells"
- [readme] expression_matrix is a matrix or data frame, where each row is a gene and each column is a cell: "expression_matrix is a matrix or data frame, where each row is a gene and each column is a cell."
- [methods] diff_activity <- activity(expression_matrix, differentiation_edges): "diff_activity <- activity(expression_matrix, differentiation_edges)"
- [methods] The remotes package is needed, if you don't have it type: install.packages("remotes"): "The remotes package is needed, if you don't have it type: install.packages("remotes")"
- [readme] expression_matrix is a matrix or data frame containing the normalized expression matrix of the data set, where each row is a gene and each column is a cell: [evidence span withheld — anchor status: not_found]
