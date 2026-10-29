---
name: gene-expression-scaling
description: Use when preparing a gene expression matrix (rows = genes, columns =
  cell populations) as input to ICELLNET interaction scoring, to bound per-gene influence
  before computing ligand–receptor and global communication scores.
when_to_use_negative:
- The expression matrix has already been scaled/normalized for ICELLNET scoring —
  rescaling again would distort the 0–10 range
- The goal is a method other than transcriptome-based ligand–receptor communication
  scoring (e.g. downstream signaling or effector analysis), which ICELLNET explicitly
  does not model
- Input is raw per-cell counts that have not been aggregated per cell population,
  since the 5%-highest-value maximum is defined per gene across the dataset
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: ICELLNET
  role: Framework whose interaction-scoring step consumes the scaled expression matrix;
    scaling follows its scRNAseq tutorial data layout
  repo: github.com/soumelis-lab/ICELLNET
- name: R
  role: Implementation language for the row-wise scaling (divide by gene maximum,
    multiply by 10, cap at 10)
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-21244-x
    title: Dissection of intercellular communication using the transcriptome-based
      framework ICELLNET
  source_content_hash: sha256:8f4cac3e1b269e5f4dd2a12b5f034ddae3a58861f226c655a34cab0392628f76
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ICELLNET
  - R
  - devtools
  - readxl
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# gene-expression-scaling

## Summary

ICELLNET-specific preprocessing that scales each ligand/receptor gene's expression by its dataset maximum (mean of the 5% highest values) to a 0–10 range, preventing highly expressed molecules (e.g. chemokines) from dominating communication scores and enabling detection of low-level molecules (e.g. IL-12, IL-4).

## When to use

Use when preparing a gene expression matrix (rows = genes, columns = cell populations) as input to ICELLNET interaction scoring, to bound per-gene influence before computing ligand–receptor and global communication scores.

## When NOT to use

- The expression matrix has already been scaled/normalized for ICELLNET scoring — rescaling again would distort the 0–10 range
- The goal is a method other than transcriptome-based ligand–receptor communication scoring (e.g. downstream signaling or effector analysis), which ICELLNET explicitly does not model
- Input is raw per-cell counts that have not been aggregated per cell population, since the 5%-highest-value maximum is defined per gene across the dataset

## Inputs

- Gene expression matrix (rows = genes, columns = cell populations) from microarray, bulk RNAseq, scRNAseq, or spatial transcriptomics, in the ICELLNET scRNAseq tutorial data layout

## Outputs

- Scaled expression matrix (e.g. scaled_expression.csv) with values in [0, 10]

## How to apply

Load the gene expression matrix following the ICELLNET scRNAseq tutorial layout (rows = genes, columns = cell populations). For each ligand/receptor gene, compute the maximum as the mean of the 5% highest expression values for that gene, then divide the gene's expression values by this maximum and multiply by 10 so the reference gene reaches a scaled value of 10. Cap any outlier values exceeding 10 at 10 to bound the scaled scores. This scaling is what prevents the global communication score from being dominated by a small number of highly expressed molecules such as chemokines, allowing detection of important low-level molecules like IL-12 and IL-4. Write the result (e.g. scaled_expression.csv) and feed it into the individual ligand–receptor interaction score computation, which multiplies scaled ligand expression in the central cell by receptor expression in the partner cell.

## Related tools

- **ICELLNET** (Framework whose interaction-scoring step consumes the scaled expression matrix; scaling follows its scRNAseq tutorial data layout) — github.com/soumelis-lab/ICELLNET
- **R** (Implementation language for the row-wise scaling (divide by gene maximum, multiply by 10, cap at 10))

## Examples

```
scaled <- sweep(expr_matrix, 1, gene_max, "/") * 10; scaled[scaled > 10] <- 10
```

## Evaluation signals

- All scaled values are non-negative
- No scaled value exceeds the cap of 10
- For each gene, the reference maximum maps to a scaled value of 10
- Gene maxima are computed as the mean of the 5% highest expression values per gene, not as absolute maxima
- Downstream ICELLNET run completes with finite, non-negative pairwise communication scores, indicating the scaled matrix was well-formed

## Limitations

- ICELLNET v2.0.0 requires R < 4.1; use ICELLNET v2.2.x for R > 4.1.x, so the R environment constrains which package version consumes the scaled matrix
- The cap at 10 bounds outliers, so extreme expression outliers are truncated rather than preserved
- Scaling bounds the influence of highly expressed molecules by design, so absolute expression magnitude information is not retained in the scaled matrix

## Evidence

- [other] ICELLNET scales each gene's expression by its maximum value in the dataset, which prevents the communication score from being dominated by a small number of highly expressed molecules: [evidence span withheld — anchor status: not_found]
- [other] Divide each gene's expression values by this computed maximum and multiply by 10 so the reference gene reaches a scaled value of 10: [evidence span withheld — anchor status: not_found]
- [other] Load the gene expression matrix (rows = genes, columns = cell populations) following the ICELLNET scRNAseq tutorial data layout: [evidence span withheld — anchor status: not_found]
- [readme] ICELLNET v2.0.0 requires R version < 4.1. For latest R versions (> 4.1.x), please use ICELLNET v2.2.x.: "ICELLNET v2.0.0 requires R version < 4.1. For latest R versions (> 4.1.x), please use ICELLNET v2.2.x."
