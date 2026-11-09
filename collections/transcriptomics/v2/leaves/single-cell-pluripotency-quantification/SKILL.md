---
name: single-cell-pluripotency-quantification
description: Use when you need per-cell stemness/pluripotency scores from a normalized
  scRNA-seq count matrix (genes x cells) to identify stem/progenitor cells as trajectory
  origins, or to compare ORIGINS, LandSCENT, and CytoTRACE scores.
when_to_use_negative:
- ORIGINS assumes the input matrix is a normalized expression matrix with genes as
  rows and cells as columns; a raw count matrix or a cell-by-gene matrix is the wrong
  input orientation/format
- The task is annotating cells by arbitrary biological processes other than differentiation
  — use ORIGINS2 or build a custom PPIN with build_ppin() instead of the differentiation
  network
- Only a Python environment is available without R — use PyOrigins rather than expecting
  the R activity() workflow verbatim
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: ORIGINS
  role: Primary R package that computes the differentiation PPIN activity score per
    cell from the expression matrix
  repo: https://github.com/danielasenraoka/ORIGINS
- name: ORIGINS2
  role: Extended version computing PPIN activity for any biological process and building
    custom PPINs with build_ppin()
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: PyOrigins
  role: Python/Jupyter implementation of the PPIN activity score with normalization
    factor e * mu_i
  repo: https://github.com/danielasenraoka/PyOrigins
- name: LandSCENT
  role: Benchmark tool providing SR pluripotency scores compared against ORIGINS activity
    via Pearson correlation
- name: CytoTRACE
  role: Benchmark tool providing CytoTRACE pluripotency scores compared against ORIGINS
    activity via Pearson correlation
- name: dplyr
  role: dplyr summarization to average correlation coefficients across the four datasets
    (ORIGINS Imports)
- name: remotes
  role: Used to install ORIGINS from GitHub (remotes::install_github)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.mex.2022.101778
    title: 'ORIGINS: A protein network-based approach to quantify cell pluripotency
      from scRNA-seq data'
  source_content_hash: sha256:cc558c93db267d0eea3055f25f593a3c981b10a6c78a2916a262a90b677006d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/danielasenraoka/ORIGINS
  tools:
  - CytoTRACE
  - LandSCENT
  - ORIGINS
  - ORIGINS2
  - PyOrigins
  - R
  - dplyr
  - magrittr
  - remotes
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6842105263157895
n_claims: 19
---

# single-cell-pluripotency-quantification

## Summary

Quantify per-cell pluripotency/stemness from scRNA-seq expression matrices using ORIGINS (differentiation PPIN activity), benchmarked against LandSCENT SR and CytoTRACE scores on four healthy human datasets (breast GSE113197, colon GSE125970, hematopoietic GSE117498, lung GSE161383).

## When to use

Use when you need per-cell stemness/pluripotency scores from a normalized scRNA-seq count matrix (genes x cells) to identify stem/progenitor cells as trajectory origins, or to compare ORIGINS, LandSCENT, and CytoTRACE scores.

## When NOT to use

- ORIGINS assumes the input matrix is a normalized expression matrix with genes as rows and cells as columns; a raw count matrix or a cell-by-gene matrix is the wrong input orientation/format
- The task is annotating cells by arbitrary biological processes other than differentiation — use ORIGINS2 or build a custom PPIN with build_ppin() instead of the differentiation network
- Only a Python environment is available without R — use PyOrigins rather than expecting the R activity() workflow verbatim

## Inputs

- Normalized scRNA-seq expression matrix or data frame (rows = genes, columns = cells)
- Differentiation PPIN edges data frame (differentiation_edges, bundled within the ORIGINS package data directory)
- GEO datasets: GSE113197 (breast epithelial), GSE125970 (colon epithelial), GSE117498 (hematopoietic bone marrow), GSE161383 (lung)
- Per-cell score tables from ORIGINS activity, activity-HVF, LandSCENT SR, and CytoTRACE for correlation benchmarking

## Outputs

- Per-cell differentiation activity score vector (diff_activity) from ORIGINS
- Pairwise Pearson correlation coefficients between ORIGINS activity, LandSCENT SR, CytoTRACE, and activity-HVF per dataset
- Mean correlation summary written to correlation_summary.csv

## How to apply

Install ORIGINS from GitHub with remotes::install_github("danielasenraoka/ORIGINS") and load it with library(ORIGINS). Provide a normalized expression matrix where each row is a gene and each column is a cell, together with the differentiation PPIN edge data frame (differentiation_edges) that ships inside the package data directory, and compute per-cell scores via diff_activity <- activity(expression_matrix, differentiation_edges). Optionally compute activity on highly variable genes (activity-HVF) and, for benchmarking, run LandSCENT to obtain SR scores and CytoTRACE on the same cells. Assess concordance by computing pairwise Pearson correlations in R with cor() between activity vs SR, activity vs CytoTRACE, SR vs CytoTRACE, and activity vs activity-HVF for each dataset, then average across the four datasets with dplyr summarization. Validate against the reported benchmark means: activity vs SR ~0.77, activity vs CytoTRACE ~0.44, SR vs CytoTRACE ~0.63, activity vs activity-HVF ~0.67.

## Related tools

- **ORIGINS** (Primary R package that computes the differentiation PPIN activity score per cell from the expression matrix) — https://github.com/danielasenraoka/ORIGINS
- **ORIGINS2** (Extended version computing PPIN activity for any biological process and building custom PPINs with build_ppin()) — https://github.com/danielasenraoka/ORIGINS2
- **PyOrigins** (Python/Jupyter implementation of the PPIN activity score with normalization factor e * mu_i) — https://github.com/danielasenraoka/PyOrigins
- **LandSCENT** (Benchmark tool providing SR pluripotency scores compared against ORIGINS activity via Pearson correlation)
- **CytoTRACE** (Benchmark tool providing CytoTRACE pluripotency scores compared against ORIGINS activity via Pearson correlation)
- **dplyr** (dplyr summarization to average correlation coefficients across the four datasets (ORIGINS Imports))
- **remotes** (Used to install ORIGINS from GitHub (remotes::install_github))

## Examples

```
remotes::install_github("danielasenraoka/ORIGINS"); library(ORIGINS); diff_activity <- activity(expression_matrix, differentiation_edges)
```

## Evaluation signals

- Mean Pearson correlation between ORIGINS activity and LandSCENT SR across the four datasets should approximate ~0.77
- Mean Pearson correlation between ORIGINS activity and CytoTRACE should approximate ~0.44, and SR vs CytoTRACE ~0.63
- Mean Pearson correlation between activity and activity-HVF should approximate ~0.67
- activity() returns one score per cell (length equals number of columns of the expression matrix)
- Workflow accepted if all computed means reproduce the reported benchmarks within small numerical tolerance and values are written to correlation_summary.csv

## Limitations

- ORIGINS is benchmarked on healthy human datasets (breast, colon, hematopoietic, lung); correlations with other stemness methods are moderate (e.g., activity vs CytoTRACE ~0.44), so scores are not interchangeable
- The differentiation PPIN is fixed within the package; the activity() call depends on the bundled differentiation_edges data frame rather than a user-supplied network
- ORIGINS requires the remotes package for installation from GitHub

## Evidence

- [abstract] This tool was implemented as an R package named ORIGINS.: "This tool was implemented as an R package named ORIGINS."
- [abstract] Benchmarking scope: "We benchmark the performance of our algorithm with two previously published tools, LandSCENT (Chen et al., 2019) and CytoTRACE (Gulati et al., 2020), for four healthy human data sets: breast, colon,"
- [methods] Installation command: "remotes::install_github("danielasenraoka/ORIGINS")"
- [methods] Core activity computation: "diff_activity <- activity(expression_matrix, differentiation_edges)"
- [readme] Input format specification: "expression_matrix is a matrix or data frame, where each row is a gene and each column is a cell."
- [readme] Bundled PPIN data: "The differentiation PPIN is loaded within the package in the data directory."
- [supplementary] GEO dataset accessions: "available on the GEO database under the accession codes: GSE113197 for breast epithelial cells"
