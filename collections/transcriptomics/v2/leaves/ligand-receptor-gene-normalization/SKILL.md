---
name: ligand-receptor-gene-normalization
description: Use when preparing a gene-by-cell-population expression matrix (microarray,
  bulk RNA-seq, or scRNA-seq) for ICELLNET communication scoring, so that highly expressed
  molecules like chemokines do not dominate interaction scores.
when_to_use_negative:
- 'The expression data are non-human: the ICELLNET ligand-receptor database is curated
  exclusively from human studies'
- The data have already been max-scaled to the 0-10 range; reapplying the scaling
  would distort values
- The goal is differential expression or clustering rather than ligand-receptor communication
  scoring, where max-scaling would erase absolute-expression information
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ICELLNET
  role: Framework whose scoring step requires max-scaled ligand/receptor expression;
    installed via devtools::install_github from soumelis-lab/ICELLNET (subdir 'icellnet')
  repo: https://github.com/soumelis-lab/ICELLNET
- name: R
  role: Implementation environment; ICELLNET v2.0.0 requires R < 4.1, latest R versions
    use ICELLNET v2.2.x
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-21244-x
    title: Dissection of intercellular communication using the transcriptome-based
      framework ICELLNET
  source_content_hash: sha256:c5459675d214d9f677c3b60d28fd0da7dac100fce2155e240713db61f576bf68
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soumelis-lab/ICELLNET
  tools:
  - ICELLNET
  - R
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# ligand-receptor-gene-normalization

## Summary

ICELLNET's scaling procedure for ligand/receptor gene expression: each gene is divided by its maximum value (computed as the mean of the top 5% highest expression values) and multiplied by 10, so values fall in a bounded 0-10 range before computing ligand-receptor interaction and global communication scores.

## When to use

Use when preparing a gene-by-cell-population expression matrix (microarray, bulk RNA-seq, or scRNA-seq) for ICELLNET communication scoring, so that highly expressed molecules like chemokines do not dominate interaction scores.

## When NOT to use

- The expression data are non-human: the ICELLNET ligand-receptor database is curated exclusively from human studies
- The data have already been max-scaled to the 0-10 range; reapplying the scaling would distort values
- The goal is differential expression or clustering rather than ligand-receptor communication scoring, where max-scaling would erase absolute-expression information

## Inputs

- Gene expression matrix (rows = ligand/receptor genes, columns = cell populations) from scRNA-seq, bulk RNA-seq, microarray, or sorted-cell-population transcriptomics

## Outputs

- Scaled expression matrix with all values non-negative and bounded at 10 (e.g., scaled_expression.csv)
- Normalized input for ICELLNET communication score quantification

## How to apply

In R, load the input matrix with rows as ligand/receptor genes and columns as cell populations, following the ICELLNET scRNAseq tutorial layout. For each gene, compute the maximum as the mean of the 5% highest expression values, then divide the gene's expression values by this maximum and multiply by 10 so the reference gene reaches a scaled value of 10. Cap any values exceeding 10 at 10 to bound the scaled scores. The rationale is to prevent the communication score from being dominated by a few highly expressed molecules (e.g., chemokines) while still detecting important low-level molecules (e.g., IL-12, IL-4). Write the result (e.g., to scaled_expression.csv) and validate before feeding it into ICELLNET score computation.

## Related tools

- **ICELLNET** (Framework whose scoring step requires max-scaled ligand/receptor expression; installed via devtools::install_github from soumelis-lab/ICELLNET (subdir 'icellnet')) — https://github.com/soumelis-lab/ICELLNET
- **R** (Implementation environment; ICELLNET v2.0.0 requires R < 4.1, latest R versions use ICELLNET v2.2.x)

## Examples

```
scaled <- t(apply(expr, 1, function(g) pmin(10 * g / mean(head(sort(g, decreasing = TRUE), 5)), 10))); write.csv(scaled, "scaled_expression.csv")
```

## Evaluation signals

- scaled_expression.csv exists and all values are non-negative
- No scaled value exceeds the cap of 10
- For each gene, the mean of its top 5% expression values maps to approximately 10 after scaling
- Scaled matrix preserves the original gene x population dimensions of the input matrix
- Low-level molecules (e.g., IL-12, IL-4) remain detectable rather than being swamped by highly expressed chemokines

## Limitations

- Scaling to a bounded 0-10 range discards absolute-expression magnitude, which matters only within ICELLNET's interaction-score framework
- The 5%-highest-value maximum estimate is sensitive to the input data layout; it must be computed per gene across the populations in the matrix
- ICELLNET v2.0.0 requires R version < 4.1; users on R > 4.1.x must use ICELLNET v2.2.x

## Evidence

- [other] scaling rationale: [evidence span withheld — anchor status: not_found]
- [readme] installation: "install_github("soumelis-lab/ICELLNET",ref="master", subdir="icellnet")"
- [readme] scRNAseq tutorial reference: "ICELLNET analysis on scRNAseq data: [Tutorial](https://github.com/soumelis-lab/ICELLNET/blob/master/exemples_tutorials/Exemple2_scRNAseq.md)"
- [methods] R version constraint: "ICELLNET v2.0.0 requires R version < 4.1. For latest R versions (> 4.1.x), please use ICELLNET v2.2.x."
