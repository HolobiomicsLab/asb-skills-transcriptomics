---
name: gamma-normal-mixture-em-estimation
description: Use when estimating gene co-expression strength from sparse, zero-inflated
  single-cell count matrices where excessive zero counts make ordinary Pearson/Spearman
  correlations unreliable, and you need to separate low-confidence zeros from genuine
  expression.
when_to_use_negative:
- The input is already bulk RNA-seq or imputed/low-zero expression data — scLink's
  zero-filtering targets excess zeros in single-cell data, and imputed data may still
  be sparser than bulk data so bulk network methods or plain correlations may suffice
  instead
- The goal is directed or typed regulatory edges — scLink's edges are undirected statistical
  dependencies and edge types cannot be distinguished from expression data alone
- The gene list is very large and unfiltered — the article recommends a gene filtering
  step by detection rate or mean expression (or filter.genes=TRUE for top-n average
  expression) and possibly module-wise analysis before applying scLink
edam_operation: http://edamontology.org/operation_3899
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scLink
  role: 'Provides sclink_norm, sclink_cor, and sclink_net: sclink_cor fits the per-gene
    Gamma-Normal mixture by EM, estimates d_ij, applies the d_ij < 0.5 confident-cells
    rule, and computes the robust correlation matrix'
  repo: https://github.com/Vivianstats/scLink
- name: R
  role: Implementation environment; scLink is distributed as an R package on CRAN
    and GitHub
  repo: https://github.com/Vivianstats/scLink
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2020.11.006
    title: 'scLink: Inferring Sparse Gene Co-Expression Networks from Single-Cell
      Expression Data'
  source_content_hash: sha256:058e6cb83b4dca5427775471305307699e64befd0d7aeb8fc78dfe2b8a296d2b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scLink
  tools:
  - R
  - scLink
  edam_operation: http://edamontology.org/operation_3899
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gamma-normal-mixture-em-estimation

## Summary

Fits scLink's per-gene Gamma-Normal mixture model via EM to estimate each cell's non-detection probability d_ij for a gene, so that confidently measured expression values can be distinguished from unreliable zeros before computing robust co-expression. It underpins scLink's robust correlation estimator for sparse, zero-inflated single-cell RNA-seq data.

## When to use

Use when estimating gene co-expression strength from sparse, zero-inflated single-cell count matrices where excessive zero counts make ordinary Pearson/Spearman correlations unreliable, and you need to separate low-confidence zeros from genuine expression.

## When NOT to use

- The input is already bulk RNA-seq or imputed/low-zero expression data — scLink's zero-filtering targets excess zeros in single-cell data, and imputed data may still be sparser than bulk data so bulk network methods or plain correlations may suffice instead
- The goal is directed or typed regulatory edges — scLink's edges are undirected statistical dependencies and edge types cannot be distinguished from expression data alone
- The gene list is very large and unfiltered — the article recommends a gene filtering step by detection rate or mean expression (or filter.genes=TRUE for top-n average expression) and possibly module-wise analysis before applying scLink

## Inputs

- Raw single-cell read count matrix (genes x cells), e.g. GSE114727 B cells with 109 cells
- Gene-filtered count matrix (genes detected in >=10% of cells; ~410 genes, ~71.2% zeros in the exemplar dataset)
- Normalized expression matrix produced by sclink_norm (scale.factor = 1e6, log10 scale)

## Outputs

- Per-gene, per-cell non-detection probabilities d_ij from the EM-fitted Gamma-Normal mixture
- Symmetric G x G robust correlation matrix with unit diagonal and off-diagonal entries in [-1, 1] (e.g. written to robust_correlation_matrix.csv)

## How to apply

Normalize the raw read count matrix with scLink's sclink_norm (library size 10^6, log10 scale) after filtering genes by detection rate (e.g. keep genes detected in at least ~10% of cells). Run sclink_cor, which fits a Gamma-Normal mixture model to each gene's expression values by EM and outputs each cell's non-detection probability d_ij for that gene. Apply the confident-cells criterion: values with d_ij < t (t = 0.5) are treated as confidently measured, while d_ij >= t entries are treated as missing. Compute pairwise-complete Pearson correlations over confident cells only, with a Pearson fallback on the remaining confident cells when fewer than nthre = 10 remain for a gene pair. Assemble the symmetric G x G robust correlation matrix and clip entries to [-1, 1] for downstream network inference with sclink_net.

## Related tools

- **scLink** (Provides sclink_norm, sclink_cor, and sclink_net: sclink_cor fits the per-gene Gamma-Normal mixture by EM, estimates d_ij, applies the d_ij < 0.5 confident-cells rule, and computes the robust correlation matrix) — https://github.com/Vivianstats/scLink
- **R** (Implementation environment; scLink is distributed as an R package on CRAN and GitHub) — https://github.com/Vivianstats/scLink

## Examples

```
library(scLink); cor.mat <- sclink_cor(counts, genes = genes, normalize = TRUE, scale.factor = 1e6, model = "gamma")
```

## Evaluation signals

- Output correlation matrix is symmetric G x G (e.g. 410 x 410 for the filtered GSE114727 B-cell data) with unit diagonal
- All off-diagonal correlation entries fall within [-1, 1] after clipping
- Cells flagged with d_ij >= 0.5 for a gene are excluded from that gene's pairwise-complete correlation, and the Pearson fallback is triggered only when fewer than 10 (nthre = 10) confident cells remain for a pair
- Normalization preceded estimation: count matrix was scaled to library size 1e6 and log10-transformed via sclink_norm before EM fitting
- Robust correlations can be compared against plain Pearson correlations to confirm the mixture filter attenuates correlations driven by shared zero-inflation

## Limitations

- No scLink R package software version is specified in the article or README, so exact numeric reproducibility is not guaranteed
- The EM-based confidence filtering requires a minimum number of confident cells; when fewer than nthre = 10 confident cells remain, a fallback Pearson on the remaining confident cells is used, which weakens the zero-filtering rationale
- Robust correlations capture undirected statistical dependencies only; regulatory vs. co-regulatory edge types cannot be distinguished from expression data alone
- Imputation-based alternatives are time-consuming with many cells, but scLink's approach still presumes a gene filtering step (detection rate or mean expression) to keep gene numbers tractable

## Evidence

- [intro] it proposes a robust estimator for measuring gene co-expression strength, built upon our previous work on improving the quality of single-cell gene expression data: "it proposes a robust estimator for measuring gene co-expression strength, built upon our previous work on improving the quality of single-cell gene expression data"
- [methods] we suggest that the users first use `sclink_cor` to investigate the correlation structures among these genes: "we suggest that the users first use `sclink_cor` to investigate the correlation structures among these genes"
- [methods] normalize the read count matrix with a library size of $10^6$ and only keep the 500 genes in `genes` for downstream analysis: "normalize the read count matrix with a library size of $10^6$ and only keep the 500 genes in `genes` for downstream analysis"
- [readme] `sclink_cor` for calculating the co-expression matrix by scLink: [evidence span withheld — anchor status: external_doc]
- [methods] use the function `sclink_net` to calculate the robust correlation matrix and identifed sparse co-expression network: "use the function `sclink_net` to calculate the robust correlation matrix and identifed sparse co-expression network"
