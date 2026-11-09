---
name: dropout-non-detection-probability-estimation
description: Use when computing gene-gene co-expression correlations from sparse,
  zero-inflated single-cell RNA-seq count matrices (e.g. ~70% zeros) where naive Pearson/Spearman
  correlations are distorted by dropout zeros.
when_to_use_negative:
- Input data are bulk RNA-seq or otherwise not zero-inflated single-cell counts —
  the dropout-mixture filtering adds no benefit and discards information
- The goal is a sparse network of direct gene associations rather than pairwise co-expression;
  use scLink's sclink_net (penalized Gaussian graphical model) instead of correlation-only
  output
- Data have already been imputed (e.g. with scImpute) — the article notes imputed
  data may still be much sparser than bulk data and imputation can be time-consuming,
  so the estimator's confidence filtering is intended for the observed count matrix
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2229
tools:
- name: scLink
  role: R package providing sclink_norm (count preprocessing), sclink_cor (EM Gamma-Normal
    mixture fit and robust correlation matrix), and sclink_net (sparse network construction)
  repo: https://github.com/Vivianstats/scLink
- name: R
  role: Runtime environment in which the scLink package functions are executed to
    filter genes, estimate d_ij, and compute pairwise-complete Pearson correlations
  repo: https://github.com/Vivianstats/scLink
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2020.11.006
    title: 'scLink: Inferring Sparse Gene Co-Expression Networks from Single-Cell
      Expression Data'
  source_content_hash: sha256:779a5cdb3a8fd94cb88da41df8aa2a759276c9aa09ed9be982794fb95955c851
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scLink
  tools:
  - R
  - scLink
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# dropout-non-detection-probability-estimation

## Summary

Reconstructs scLink's robust co-expression estimator, which fits a per-gene Gamma-Normal mixture model by EM to estimate each cell's non-detection (dropout) probability d_ij, then computes pairwise-complete Pearson correlations using only confidently measured cells (d_ij < 0.5).

## When to use

Use when computing gene-gene co-expression correlations from sparse, zero-inflated single-cell RNA-seq count matrices (e.g. ~70% zeros) where naive Pearson/Spearman correlations are distorted by dropout zeros.

## When NOT to use

- Input data are bulk RNA-seq or otherwise not zero-inflated single-cell counts — the dropout-mixture filtering adds no benefit and discards information
- The goal is a sparse network of direct gene associations rather than pairwise co-expression; use scLink's sclink_net (penalized Gaussian graphical model) instead of correlation-only output
- Data have already been imputed (e.g. with scImpute) — the article notes imputed data may still be much sparser than bulk data and imputation can be time-consuming, so the estimator's confidence filtering is intended for the observed count matrix

## Inputs

- Raw single-cell read count matrix (genes x cells), e.g. GSE114727 B-cell counts
- Gene detection-rate filter threshold (keep genes detected in >=10% of cells)
- Non-detection probability threshold t = 0.5
- Minimum confident-cell count nthre = 10 for Pearson correlation

## Outputs

- Symmetric G x G robust correlation matrix with unit diagonal and off-diagonal values in [-1, 1] (e.g. robust_correlation_matrix.csv, 410 x 410)
- Per-cell per-gene non-detection probability estimates d_ij from the EM-fitted Gamma-Normal mixture

## How to apply

Load the raw single-cell count matrix (e.g. GSE114727 B cells, 109 cells) and filter genes by detection rate, keeping genes detected in >=10% of cells (~410 genes, ~71.2% zeros). Normalize the filtered counts with scLink's sclink_norm (scale.factor = 1e6, log10 scale). Fit the per-gene Gamma-Normal mixture model by EM, as implemented in sclink_cor, to estimate each cell's non-detection probability d_ij for every gene. Treat cells with d_ij < 0.5 as confidently measured and d_ij >= 0.5 as missing, then compute pairwise-complete Pearson correlations over the confident cells, falling back to the available confident cells when fewer than nthre = 10 remain for a gene pair. Assemble the symmetric G x G robust correlation matrix, clip entries to [-1, 1], and write it out (e.g. robust_correlation_matrix.csv).

## Related tools

- **scLink** (R package providing sclink_norm (count preprocessing), sclink_cor (EM Gamma-Normal mixture fit and robust correlation matrix), and sclink_net (sparse network construction)) — https://github.com/Vivianstats/scLink
- **R** (Runtime environment in which the scLink package functions are executed to filter genes, estimate d_ij, and compute pairwise-complete Pearson correlations) — https://github.com/Vivianstats/scLink

## Examples

```
install.packages("scLink"); counts.norm <- sclink_norm(counts); cor.mat <- sclink_cor(counts.norm)
```

## Evaluation signals

- Output CSV is a symmetric 410 x 410 (genes x genes) matrix matching the filtered gene set
- Diagonal of the correlation matrix equals 1 for all genes
- All off-diagonal entries lie within [-1, 1] after clipping
- Every correlation is computed from cells with d_ij < 0.5; gene pairs with fewer than 10 confident cells use the Pearson fallback on the remaining confident cells
- Genes entering the computation are exactly those passing the >=10% detection-rate filter from the raw count matrix

## Limitations

- Estimates of non-detection probability depend on the EM-fitted Gamma-Normal mixture; the scLink R package version is not specified in the article, complicating exact reproduction
- When fewer than nthre = 10 confident cells remain for a gene pair, the correlation relies on a small-sample Pearson fallback and may be unstable
- Imputed expression data may remain much sparser than bulk data, and imputation can be time-consuming depending on cell number
- scLink-identified edges are based on statistical dependencies and carry no direction; regulatory, co-regulatory, and functionally coordinated edge types cannot be distinguished from expression data alone

## Evidence

- [intro] it proposes a robust estimator for measuring gene co-expression strength, built upon our previous work on improving the quality of single-cell gene expression data: "it proposes a robust estimator for measuring gene co-expression strength, built upon our previous work on improving the quality of single-cell gene expression data"
- [methods] normalize the read count matrix with a library size of $10^6$ and only keep the 500 genes in `genes` for downstream analysis: "normalize the read count matrix with a library size of $10^6$ and only keep the 500 genes in `genes` for downstream analysis"
- [methods] use the function `sclink_cor` to investigate the correlation structures among these genes: "we suggest that the users first use `sclink_cor` to investigate the correlation structures among these genes"
- [readme] The package is available on CRAN. For installation please use the following codes in `R` install.packages("scLink"): "The package is available on CRAN. For installation please use the following codes in `R`

 r
install.packages("scLink")
"
- [readme] sclink_cor for calculating the co-expression matrix by scLink: [evidence span withheld — anchor status: external_doc]
