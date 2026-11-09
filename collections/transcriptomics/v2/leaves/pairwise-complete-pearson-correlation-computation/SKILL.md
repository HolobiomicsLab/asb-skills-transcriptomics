---
name: pairwise-complete-pearson-correlation-computation
description: Use when computing gene-gene co-expression correlations from sparse,
  zero-inflated scRNA-seq count matrices where excess zeros make ordinary Pearson/Spearman
  correlations unreliable, and a confidence-filtered pairwise-complete estimate is
  needed.
when_to_use_negative:
- Input is bulk RNA-seq or already-imputed expression data with few zeros — scLink's
  discussion notes imputed data may remain sparse, but the mixture-based confidence
  filtering targets excess zeros in raw single-cell counts
- The goal is directed or regulatory edge types — scLink's edges are undirected statistical
  dependencies and regulatory/co-regulatory types cannot be distinguished from expression
  data alone
- A very large gene set needs full network inference at once — the article suggests
  first dividing genes into modules based on scLink's correlation and applying scLink
  separately to each module
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0128
tools:
- name: scLink
  role: R package providing sclink_norm for normalization, sclink_cor for the Gamma-Normal
    mixture EM fitting and robust correlation matrix, and sclink_net for downstream
    network construction
  repo: https://github.com/Vivianstats/scLink
- name: R
  role: Runtime environment for the scLink package and the pairwise-complete Pearson
    correlation computation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2020.11.006
    title: 'scLink: Inferring Sparse Gene Co-Expression Networks from Single-Cell
      Expression Data'
  source_content_hash: sha256:bf4977db0261f6dd9fd07b7ca8f7a467a4091ae5969f537840a57c049caa722f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scLink
  tools:
  - R
  - scLink
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# pairwise-complete-pearson-correlation-computation

## Summary

Reconstructs scLink's robust co-expression estimator, which measures gene-gene correlation from sparse, zero-inflated single-cell data by treating low-confidence expression values as missing and computing pairwise-complete Pearson correlations over the remaining confidently measured cells.

## When to use

Use when computing gene-gene co-expression correlations from sparse, zero-inflated scRNA-seq count matrices where excess zeros make ordinary Pearson/Spearman correlations unreliable, and a confidence-filtered pairwise-complete estimate is needed.

## When NOT to use

- Input is bulk RNA-seq or already-imputed expression data with few zeros — scLink's discussion notes imputed data may remain sparse, but the mixture-based confidence filtering targets excess zeros in raw single-cell counts
- The goal is directed or regulatory edge types — scLink's edges are undirected statistical dependencies and regulatory/co-regulatory types cannot be distinguished from expression data alone
- A very large gene set needs full network inference at once — the article suggests first dividing genes into modules based on scLink's correlation and applying scLink separately to each module

## Inputs

- Raw single-cell read count matrix (genes x cells), e.g. GSE114727 B-cell subset (109 cells)
- Gene filter: genes detected in >=10% of cells (~410 genes)
- Normalized expression matrix from sclink_norm (library size 10^6, log10 scale)
- Per-cell non-detection probabilities d_ij from the EM-fitted Gamma-Normal mixture model

## Outputs

- Symmetric G x G robust co-expression matrix (e.g. robust_correlation_matrix.csv, 410 x 410) with unit diagonal and entries in [-1, 1]

## How to apply

Start from a filtered count matrix (e.g. GSE114727 B cells; 109 cells; genes detected in >=10% of cells, ~410 genes, ~71.2% zeros) and normalize with scLink's sclink_norm (scale.factor = 1e6, log10 scale). Fit the per-gene Gamma-Normal mixture model by EM via sclink_cor to estimate each cell's non-detection probability d_ij for every gene. Apply the confidence criterion: cells with d_ij < 0.5 are kept as confidently measured, entries with d_ij >= 0.5 are treated as missing. Compute the Pearson correlation for each gene pair using only the cells where both genes are confidently measured; when fewer than 10 confident cells (nthre = 10) remain for a pair, fall back to Pearson on the remaining confident cells. Assemble the symmetric G x G correlation matrix, clip entries to [-1, 1], and validate symmetry, unit diagonal, and bounded off-diagonal values.

## Related tools

- **scLink** (R package providing sclink_norm for normalization, sclink_cor for the Gamma-Normal mixture EM fitting and robust correlation matrix, and sclink_net for downstream network construction) — https://github.com/Vivianstats/scLink
- **R** (Runtime environment for the scLink package and the pairwise-complete Pearson correlation computation)

## Examples

```
library(scLink); counts.norm <- sclink_norm(counts); cor.mat <- sclink_cor(counts.norm)
```

## Evaluation signals

- Output CSV is a symmetric 410 x 410 matrix (dimension matches number of filtered genes)
- Diagonal entries equal 1 and all off-diagonal values lie within [-1, 1] (clip entries to [-1, 1])
- Every correlation is computed over only cells with d_ij < 0.5 for both genes; pairs with fewer than 10 confident cells (nthre = 10) use the Pearson fallback on the remaining confident cells
- Normalization precedes correlation: read counts scaled with library size 10^6 and log10-transformed via sclink_norm
- Robust correlation matrix reproduces scLink's correlation structure suitable for downstream sclink_net sparse-network inference

## Limitations

- No software version specified for the scLink R package in the article or README
- Edges identified by scLink are based on statistical dependencies and do not have directions; regulatory, co-regulatory, and functionally coordinated edge types cannot be distinguished from expression data alone
- The Pearson fallback triggers when fewer than 10 confident cells (nthre = 10) remain, which may yield less reliable estimates for gene pairs with many low-confidence values

## Evidence

- [intro] it proposes a robust estimator for measuring gene co-expression strength: "it proposes a robust estimator for measuring gene co-expression strength, built upon our previous work on improving the quality of single-cell gene expression data"
- [methods] sclink_norm usage: "We use the function `sclink_norm` to process single cell read count for application of the sclink method"
- [methods] sclink_cor usage: "we suggest that the users first use `sclink_cor` to investigate the correlation structures among these genes"
- [methods] normalize with library size 10^6: "normalize the read count matrix with a library size of $10^6$ and only keep the 500 genes in `genes` for downstream analysis"
- [discussion] edges are undirected: "the edges identified by scLink are based on statistical dependencies and do not have directions"
- [readme] README main functions: [evidence span withheld — anchor status: external_doc]
