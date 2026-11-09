---
name: r-package-data-handling
description: Use when you need to compute per-cell biological-process activity from
  a normalized scRNA-seq expression matrix using the ORIGINS2 R package's activity()
  or build_ppin() functions.
when_to_use_negative:
- The expression matrix is not normalized, since activity() expects a normalized expression
  matrix as input
- The goal is bulk-RNA-seq or non-single-cell pathway scoring; the method and package
  are designed for scRNA-seq data
- No gene-level PPIN edges exist for the biological process of interest and full_human_ppin
  cannot be used to build one
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: ORIGINS2
  role: R package providing activity() to compute PPIN activity per cell and build_ppin()
    to construct custom PPINs from a gene ID list
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: remotes
  role: Used to install ORIGINS2 from GitHub via remotes::install_github()
- name: dplyr
  role: Package dependency (Imports) that must be attached when loading ORIGINS2
- name: magrittr
  role: Package dependency (Imports) providing the pipe operator used with ORIGINS2
    workflows
- name: R
  role: Runtime environment in which ORIGINS2 is installed and executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.mex.2023.102179
    title: 'Cell annotation using scRNA-seq data: A protein-protein interaction network
      approach'
  source_content_hash: sha256:ff9caadf3474d283002e8c75dea9c86966098fd7b9bfdf9e7775d5959f430058
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/danielasenraoka/ORIGINS2
  repo_url: https://github.com/danielasenraoka/ORIGINS2
  tools:
  - ORIGINS2
  - R
  - dplyr
  - magrittr
  - remotes
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# r-package-data-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Install, load, and correctly feed data into an open-source R package (ORIGINS2) that computes protein-protein interaction network (PPIN) activity scores for scRNA-seq data, ensuring the normalized expression matrix and PPIN adjacency edges are supplied in the shapes the package expects.

## When to use

Use when you need to compute per-cell biological-process activity from a normalized scRNA-seq expression matrix using the ORIGINS2 R package's activity() or build_ppin() functions.

## When NOT to use

- The expression matrix is not normalized, since activity() expects a normalized expression matrix as input
- The goal is bulk-RNA-seq or non-single-cell pathway scoring; the method and package are designed for scRNA-seq data
- No gene-level PPIN edges exist for the biological process of interest and full_human_ppin cannot be used to build one

## Inputs

- Normalized scRNA-seq expression matrix or data frame (rows = genes, columns = cells)
- PPIN adjacency-edge data frame for the biological process of interest (e.g. differentiation_edges, cell_cycle_edges)
- Gene ID list (genes_bp) for a custom biological process, plus the full_human_ppin adjacency list when calling build_ppin

## Outputs

- Per-cell activity vector (sum of all PPIN edges per cell when normalization=FALSE, or edge-count-normalized activity when normalization=TRUE)
- Custom PPIN adjacency-edge data frame from build_ppin

## How to apply

First install the remotes package (install.packages("remotes")) and then install ORIGINS2 from GitHub via remotes::install_github("danielasenraoka/ORIGINS2"); load it with library(ORIGINS2) and attach dplyr and magrittr (package Imports). Prepare a normalized expression matrix or data frame where each row is a gene and each column is a cell. For each biological process, pass a PPIN adjacency-edge data frame (17-18 BP edge sets such as differentiation_edges, cell_cycle_edges, dna_repair_edges are shipped in the package) to activity(expression_matrix, adjacency_edges, FALSE); the logical normalization flag set to FALSE returns the sum of all PPIN edges per cell, while TRUE normalizes by the number of PPIN edges. To score a custom biological process, build its PPIN with build_ppin(genes_bp, ppi_edges_complete = full_human_ppin), where full_human_ppin is the complete Human PPIN adjacency list included in the package. Correct application means the output is a per-cell activity vector whose length equals the number of columns (cells) of the expression matrix.

## Related tools

- **ORIGINS2** (R package providing activity() to compute PPIN activity per cell and build_ppin() to construct custom PPINs from a gene ID list) — https://github.com/danielasenraoka/ORIGINS2
- **remotes** (Used to install ORIGINS2 from GitHub via remotes::install_github())
- **dplyr** (Package dependency (Imports) that must be attached when loading ORIGINS2)
- **magrittr** (Package dependency (Imports) providing the pipe operator used with ORIGINS2 workflows)
- **R** (Runtime environment in which ORIGINS2 is installed and executed)

## Examples

```
install.packages("remotes"); remotes::install_github("danielasenraoka/ORIGINS2"); library(ORIGINS2); diff_activity <- activity(expression_matrix, differentiation_edges, FALSE)
```

## Evaluation signals

- library(ORIGINS2) loads without error after remotes::install_github("danielasenraoka/ORIGINS2") succeeds
- activity() returns a vector whose length equals the number of columns (cells) of the expression matrix
- With normalization=FALSE the output equals the sum of all PPIN edges; with normalization=TRUE values are rescaled by the number of edges of the PPIN
- build_ppin() returns an adjacency-edge data frame restricted to the supplied gene ID list, derived from full_human_ppin
- Package-shipped edge sets (e.g. differentiation_edges, cell_cycle_edges, full_human_ppin) resolve without manual loading

## Limitations

- The expression_matrix must be normalized and oriented as genes in rows, cells in columns; other orientations will fail or produce meaningless activity
- Activity with normalization=FALSE scales with the number of PPIN edges, so raw sums are not directly comparable across BP gene sets of different sizes (e.g. Ngenes>200 subset)
- Only 17-18 preloaded biological-process PPIN edge sets ship with the package; other processes require build_ppin and depend on coverage of full_human_ppin

## Evidence
