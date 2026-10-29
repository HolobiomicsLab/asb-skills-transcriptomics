---
name: marker-gene-percent-expressing-aggregation
description: 'Use when validating scRNA-seq imputation on marker genes: you have a
  cell-by-gene normalized matrix (and optionally its imputed counterpart), cell-type
  labels, and need per-type percent-expressing fractions for genes like NCAM1 or CR2.'
when_to_use_negative:
- The input matrix is raw counts that have not been library/log normalized — percent-expressing
  comparisons against ALRA output require matching normalized scales
- The goal is differential expression testing between cell types rather than verification
  that biological zeros were preserved/completed by imputation
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ALRA
  role: Provides the b_nk_example and labels_example data, the normalize_data/alra
    functions whose input and output matrices are aggregated, and the aggregate-based
    percent-expressing checking recipe
  repo: https://github.com/KlugerLab/ALRA
- name: rsvd
  role: Randomized SVD prerequisite package used by ALRA's choose_k/alra to produce
    the completed matrix whose marker percentages are aggregated
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:6b143bb31ba5640fae2b2a390aa82e9e7928e24e39cd8873d8b17bfc8c87c1ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KlugerLab/ALRA
  tools:
  - ALRA
  - rsvd
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# marker-gene-percent-expressing-aggregation

## Summary

Quantifies, for chosen marker genes (e.g. NCAM1, CR2) in a scRNA-seq matrix, the percentage of cells with non-zero expression per cell type, before and after imputation, to verify that ALRA recovers biological-zero marker expression in the B/NK PBMC data.

## When to use

Use when validating scRNA-seq imputation on marker genes: you have a cell-by-gene normalized matrix (and optionally its imputed counterpart), cell-type labels, and need per-type percent-expressing fractions for genes like NCAM1 or CR2.

## When NOT to use

- The input matrix is raw counts that have not been library/log normalized — percent-expressing comparisons against ALRA output require matching normalized scales
- The goal is differential expression testing between cell types rather than verification that biological zeros were preserved/completed by imputation

## Inputs

- Normalized scRNA-seq expression matrix with cells as rows and genes as columns (e.g. A_norm from normalize_data on b_nk_example)
- ALRA-completed matrix A_norm_completed (result.completed[[3]] from alra)
- Cell-type labels vector (labels_example from the ALRA package)
- Marker gene column names (e.g. 'NCAM1', 'CR2')

## Outputs

- Per cell-type x gene percent-expressing table for the original normalized matrix
- Per cell-type x gene percent-expressing table for the imputed matrix
- Combined before/after table written to marker_expression_fractions.csv

## How to apply

Given a normalized expression matrix (cells as rows, genes as columns) such as ALRA's A_norm, select the marker-gene columns (NCAM1 for NK cells, CR2 for B cells) and compute percent = 100 * sum(x > 0) / length(x) within each cell type using R's aggregate with by=list(" "=labels_example). Run the aggregation twice: once on the normalized matrix and once on the imputed completed matrix A_norm_completed (result.completed[[3]] from alra). Combine the before/after percentages into one table and write it to marker_expression_fractions.csv. The rationale is that biological zeros should stay at zero while technical zeros are completed, so a correct ALRA run should raise NCAM1 expression in NK cells from ~4% and CR2 in B cells from ~1% to substantially higher levels. Round percentages to one decimal for the reported table.

## Related tools

- **ALRA** (Provides the b_nk_example and labels_example data, the normalize_data/alra functions whose input and output matrices are aggregated, and the aggregate-based percent-expressing checking recipe) — https://github.com/KlugerLab/ALRA
- **rsvd** (Randomized SVD prerequisite package used by ALRA's choose_k/alra to produce the completed matrix whose marker percentages are aggregated)

## Examples

```
aggregate(A_norm_completed[,c('NCAM1','CR2')], by=list(" "=labels_example), FUN=function(x) round(c(percent=100*sum(x>0)/length(x)),1))
```

## Evaluation signals

- The CSV marker_expression_fractions.csv exists and contains the required fields covering four cell-type x gene percentage groups
- Post-imputation NCAM1 percent-expressing in NK cells increases substantially from the original 4%
- Post-imputation CR2 percent-expressing in B cells increases substantially from the original 1%
- All reported percentages fall in the expected range (0–100, with post-imputation marker percentages in the expected high range)
- Percentages are computed as 100 * sum(x > 0) / length(x) within each label group and rounded to one decimal

## Limitations

- Results improve when using the entire dataset rather than the bundled b_nk_example subset (per the ALRA README)
- The automatic k choice from choose_k may warrant manual inspection of the singular-value spectrum; a poor k choice degrades the completed matrix being evaluated
- Percent-expressing alone does not distinguish biological zeros correctly preserved from over-imputation; non-marker genes must also stay at zero

## Evidence

- [readme] README usage of aggregate on marker columns: "print(aggregate(A_norm_completed[,c('NCAM1','CR2')], by=list(" "=labels_example),FUN=function(x) round(c(percent=100*sum(x>0)/length(x)),1)))"
- [readme] Expected baseline expression levels: "NCAM1 should be expressed in all NK cells, but is only expressed in 4% of of NK cells in the original data."
- [abstract] Zero-preserving claim motivating the check: "We present a method based on low-rank matrix approximation which imputes these values while preserving biologically non-expressed genes (true biological zeros) at zero expression levels."
- [intro] Thresholding rationale: "the last step of ALRA is to restore the biological zeros in the matrix by thresholding its entries"
