---
name: ppin-network-activity-computation
description: Use when you have a gene-by-cell scRNA-seq count matrix and need per-cell
  pluripotency/stemness scores to identify stem or progenitor cells as trajectory
  origins.
when_to_use_negative:
- When you need a general-purpose cell-type annotation or differential expression
  method rather than a pluripotency/stemness score
- When the PPIN must cover a biological process other than differentiation and you
  are using base ORIGINS rather than ORIGINS2, which supports custom and non-differentiation
  PPINs
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0128
tools:
- name: ORIGINS
  role: R package providing activity() to compute the differentiation PPIN activity
    score per cell from the expression matrix
  repo: https://github.com/danielasenraoka/ORIGINS
- name: ORIGINS2
  role: Extended version computing PPIN activity for 18 biological processes and custom
    PPINs via build_ppin()
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: PyOrigins
  role: Python/Jupyter implementation of the PPIN activity score computation
  repo: https://github.com/danielasenraoka/PyOrigins
- name: remotes
  role: Used to install ORIGINS from GitHub via remotes::install_github
- name: dplyr
  role: Imported dependency used to aggregate per-cell scores into per-cell-type averages
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.mex.2022.101778
    title: 'ORIGINS: A protein network-based approach to quantify cell pluripotency
      from scRNA-seq data'
  source_content_hash: sha256:85ecdf34ac473582b97993e494d494af459e66a8a10bffa09639eb853acca227
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/danielasenraoka/ORIGINS
  tools:
  - ORIGINS
  - ORIGINS2
  - PyOrigins
  - R
  - dplyr
  - remotes
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# ppin-network-activity-computation

## Summary

Computes a per-cell differentiation-activity score from a scRNA-seq expression matrix using a differentiation-associated Protein-protein Interaction Network (PPIN) with the ORIGINS R package, to quantify cell pluripotency/stemness.

## When to use

Use when you have a gene-by-cell scRNA-seq count matrix and need per-cell pluripotency/stemness scores to identify stem or progenitor cells as trajectory origins.

## When NOT to use

- When you need a general-purpose cell-type annotation or differential expression method rather than a pluripotency/stemness score
- When the PPIN must cover a biological process other than differentiation and you are using base ORIGINS rather than ORIGINS2, which supports custom and non-differentiation PPINs

## Inputs

- Normalized scRNA-seq expression matrix or data frame (rows = genes, columns = cells)
- Differentiation PPIN edge data frame (differentiation_edges, bundled within the ORIGINS package)

## Outputs

- Per-cell differentiation activity score vector (diff_activity)
- Per-cell-type average pluripotency scores (e.g. colon_stemness_scores.csv)

## How to apply

Load the ORIGINS R package (installed via remotes::install_github('danielasenraoka/ORIGINS')), which ships the differentiation PPIN (differentiation_edges) in its data directory. Provide an expression matrix or data frame where rows are genes and columns are cells, and call activity(expression_matrix, differentiation_edges) to obtain a per-cell differentiation activity vector (diff_activity). The score aggregates expression over PPIN edges, normalized by edge count and average expression, so it is comparable across samples and networks. Aggregate per-cell scores into per-cell-type averages (e.g. with dplyr) to rank cell types such as stem, transit-amplifying (TA), and differentiated cells. For extended use, ORIGINS2 provides PPINs for 18 biological processes and build_ppin() for custom gene lists, and PyOrigins offers a Python implementation.

## Related tools

- **ORIGINS** (R package providing activity() to compute the differentiation PPIN activity score per cell from the expression matrix) — https://github.com/danielasenraoka/ORIGINS
- **ORIGINS2** (Extended version computing PPIN activity for 18 biological processes and custom PPINs via build_ppin()) — https://github.com/danielasenraoka/ORIGINS2
- **PyOrigins** (Python/Jupyter implementation of the PPIN activity score computation) — https://github.com/danielasenraoka/PyOrigins
- **remotes** (Used to install ORIGINS from GitHub via remotes::install_github)
- **dplyr** (Imported dependency used to aggregate per-cell scores into per-cell-type averages)

## Examples

```
remotes::install_github("danielasenraoka/ORIGINS"); library(ORIGINS); diff_activity <- activity(expression_matrix, differentiation_edges)
```

## Evaluation signals

- The activity() call returns a named per-cell score vector with one value per column of the input expression matrix
- On the GSE125970 colon (Colon-2) benchmark, per-type averages should reproduce the reported result: transit-amplifying (TA) cells, not stem cells, rank first by average score for ORIGINS, LandSCENT, and CytoTRACE
- Scores are normalized by the number of PPIN edges and per-cell mean expression, enabling comparison across samples and PPINs
- In ORIGINS2, setting normalization=FALSE returns the raw sum of PPIN edge contributions, while TRUE normalizes by edge count — check the flag matches the intended comparison

## Limitations

- On the colon dataset (GSE125970, Colon-2), ORIGINS and the benchmarked methods (LandSCENT, CytoTRACE) were limited in identifying true stem cells: TA cells scored highest on average
- ORIGINS requires less RAM than LandSCENT and CytoTRACE and is more efficient than LandSCENT, but very large matrices may still be constrained
- Base ORIGINS is limited to the differentiation PPIN; other biological processes require the ORIGINS2 extension

## Evidence

- [readme] R package that provides a function to compute activity of a Protein-protein Interaction Network (PPIN) asociated with differentiation for scRNA-seq data.: "R package that provides a function to compute activity of a Protein-protein Interaction Network (PPIN) asociated with differentiation for scRNA-seq data."
- [methods] activity call: "diff_activity <- activity(expression_matrix, differentiation_edges)"
- [readme] expression matrix definition: "expression_matrix is a matrix or data frame, where each row is a gene and each column is a cell."
- [intro] PPIN bundled in package: "The differentiation PPIN is loaded within the package in the data directory."
- [readme] normalization rationale: [evidence span withheld — anchor status: external_doc]
- [supplementary] benchmark result on colon: "GSE125970 for colon epithelial cells"
