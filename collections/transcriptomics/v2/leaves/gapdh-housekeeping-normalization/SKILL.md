---
name: gapdh-housekeeping-normalization
description: Use when comparing single-cell expression distributions (e.g., Gini coefficients)
  between platforms like Drop-seq and RNA FISH, and a shared housekeeping gene (GAPDH)
  is available to control for cell-level sampling depth differences.
when_to_use_negative:
- The dataset lacks GAPDH or another reliably expressed housekeeping gene measured
  on both platforms being compared
- A library-size normalization approach (e.g., Seurat library/log normalization) is
  already being applied and platform-comparable per-cell scaling is handled differently
- The analysis goal is differential expression between cell types, where housekeeping
  normalization could obscure biology rather than control technical variation
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: ALRA
  role: Consumes the GAPDH-normalized Drop-seq matrix as input for rank-k imputation
    (k=14) in the downstream Gini-consistency analysis
  repo: https://github.com/KlugerLab/ALRA
- name: rsvd
  role: Randomized SVD package used by ALRA on the normalized matrix; prerequisite
    for ALRA installation
  repo: https://github.com/KlugerLab/ALRA
- name: SAVER
  role: Baseline imputation method applied to the same normalized data for comparison
    against ALRA (Gini correlation 0.6 vs 0.8)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:38aad6be173cab278c4e21299e6bcd955d664467e07322b89ec496e9986c6a6a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KlugerLab/ALRA
  tools:
  - ALRA
  - SAVER
  - rsvd
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gapdh-housekeeping-normalization

## Summary

Normalizes single-cell RNA-seq gene expression by GAPDH housekeeping expression after removing cells in the bottom and top tenth percentiles of GAPDH expression. This enables apples-to-apples comparison of expression distributions (e.g., Gini coefficients) between Drop-seq and RNA FISH measurements of the same cell line.

## When to use

Use when comparing single-cell expression distributions (e.g., Gini coefficients) between platforms like Drop-seq and RNA FISH, and a shared housekeeping gene (GAPDH) is available to control for cell-level sampling depth differences.

## When NOT to use

- The dataset lacks GAPDH or another reliably expressed housekeeping gene measured on both platforms being compared
- A library-size normalization approach (e.g., Seurat library/log normalization) is already being applied and platform-comparable per-cell scaling is handled differently
- The analysis goal is differential expression between cell types, where housekeeping normalization could obscure biology rather than control technical variation

## Inputs

- Filtered gene-by-cell expression matrix from Drop-seq (Torre et al. 2018 melanoma data: 8498 cells, 12,241 genes after filtering)
- RNA FISH expression data for the same cell line (26 genes, GAPDH included)
- Per-cell GAPDH expression values for percentile-based outlier removal

## Outputs

- GAPDH-normalized Drop-seq expression matrix with outlier cells removed
- GAPDH-normalized RNA FISH expression values
- Normalized matrices suitable for per-gene Gini coefficient computation (all genes except GAPDH)

## How to apply

After filtering the Drop-seq matrix (drop cells with library size >20,000 or <500 and genes with mean expression <0.1, yielding 8498 cells and 12,241 genes with 16 genes shared with the 26-gene RNA FISH panel), remove cells falling in the bottom and top tenth percentiles of GAPDH expression to exclude potential outliers and doublets. Then divide every gene's expression value in each remaining cell by that cell's GAPDH expression, so all downstream statistics are expressed relative to a housekeeping baseline. Apply the identical GAPDH normalization to the RNA FISH data before computing per-gene statistics such as the Gini coefficient. GAPDH itself is excluded from downstream per-gene comparisons (15 of the 16 shared genes remain). This normalization is a prerequisite for the Gini-coefficient consistency comparison that validated ALRA (correlation 0.8 vs. 0.3 raw and 0.6 for SAVER).

## Related tools

- **ALRA** (Consumes the GAPDH-normalized Drop-seq matrix as input for rank-k imputation (k=14) in the downstream Gini-consistency analysis) — https://github.com/KlugerLab/ALRA
- **rsvd** (Randomized SVD package used by ALRA on the normalized matrix; prerequisite for ALRA installation) — https://github.com/KlugerLab/ALRA
- **SAVER** (Baseline imputation method applied to the same normalized data for comparison against ALRA (Gini correlation 0.6 vs 0.8))

## Examples

```
cells_keep <- !(gapdh_expr <= quantile(gapdh_expr, 0.10) | gapdh_expr >= quantile(gapdh_expr, 0.90)); expr_norm <- sweep(expr[cells_keep, ], 1, gapdh_expr[cells_keep], "/")
```

## Evaluation signals

- After normalization, 15 shared genes (GAPDH excluded) remain available for FISH-vs-Drop-seq comparison out of the 16 originally shared
- Cells in the bottom and top tenth percentiles of GAPDH expression must be absent from the normalized output matrix
- Per-gene Gini coefficients computed on normalized Drop-seq data should yield a FISH correlation of approximately 0.3 (raw), 0.6 (SAVER), and 0.8 (ALRA-imputed)
- All correlation values for the 15 genes are written to gini_fish_correlation.csv and are finite Pearson correlations in [-1, 1]

## Limitations

- Percentile-based GAPDH outlier removal (bottom/top 10%) is an aggressive trim that discards ~20% of cells; reproducibility depends on replicating this exact step before imputation
- GAPDH-normalization assumes GAPDH expression is stable across cells; cell-cycle or stress-driven variation in GAPDH would bias per-cell scaling
- The Gini-consistency validation used k=14 rank for ALRA with rsvd; other choices of k may change downstream correlations

## Evidence

- [other] Filter cells with library size greater than 20,000 or less than 500 and genes with mean expression less than 0.1, yielding 8498 cells, 12,241 genes, and 16 genes shared between Drop-seq and FISH.: [evidence span withheld — anchor status: not_found]
- [other] Remove cells in the bottom and top tenth percentiles of GAPDH expression and normalize all genes by GAPDH expression.: [evidence span withheld — anchor status: not_found]
- [other] Compute the Gini coefficient of each of the other 15 genes (excluding GAPDH) in the FISH data, the raw Drop-seq data, and the ALRA-imputed Drop-seq data.: [evidence span withheld — anchor status: not_found]
- [other] ALRA outperforms all other imputation methods on the Gini coefficient consistency metric between FISH and Drop-seq data, obtaining a correlation of 0.8 (vs. 0.3 for raw Drop-seq and 0.6 for SAVER).: [evidence span withheld — anchor status: not_found]
- [readme] Please be sure to pass ALRA a matrix where the cells are rows and genes are columns.: "Please be sure to pass ALRA a matrix where the cells are rows and genes are columns"
