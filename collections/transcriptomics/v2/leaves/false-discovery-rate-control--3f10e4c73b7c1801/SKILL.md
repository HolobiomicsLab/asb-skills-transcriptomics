---
name: false-discovery-rate-control
description: Use when you have per-gene Wald test p-values from per-feature Tweedie
  (CPLM/ZICP) GLM fits of scRNA-seq UMI or read counts and must convert them into
  FDR-adjusted q-values for DE calls at α = 0.05.
when_to_use_negative:
- Input is a single pre-specified hypothesis with no multiple testing across genes,
  where raw p-values already control error
- Input is continuous transformed abundance data with no per-feature inferential p-values
  (e.g., unsupervised clustering or WGCNA-style co-expression outputs)
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Tweedieverse
  role: R/Bioconductor package that fits per-gene CPLM/ZICP models, computes Wald
    p-values, and returns multiplicity-adjusted q-values
  repo: https://github.com/himelmallick/Tweedieverse
- name: BenchmarkSingleCell
  role: Hosted benchmarking code whose evaluator produces tables of adjusted p-values
    (q-values after FDR correction) for every method and scenario
  repo: https://github.com/himelmallick/BenchmarkSingleCell
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1002/sim.9430
    title: Differential expression of single‐cell RNA‐seq data using Tweedie models
  source_content_hash: sha256:76063fba32acb2dbba088eeb3f86ca7c0264e98c47e88df5f529e541461e57b2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/himelmallick/Tweedieverse
  tools:
  - BenchmarkSingleCell
  - Bioconductor
  - R
  - Tweedieverse
  - cplm
  - cplm (R package)
  - statmod
  - statmod (R package)
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# false-discovery-rate-control

## Summary

Adjusts per-gene Wald test p-values from the Tweedieverse CPLM/ZICP models to multiplicity-adjusted q-values so that genes are called differentially expressed only when the FDR-adjusted q-value falls below a pre-specified α = 0.05 Type 1 error threshold.

## When to use

Use when you have per-gene Wald test p-values from per-feature Tweedie (CPLM/ZICP) GLM fits of scRNA-seq UMI or read counts and must convert them into FDR-adjusted q-values for DE calls at α = 0.05.

## When NOT to use

- Input is a single pre-specified hypothesis with no multiple testing across genes, where raw p-values already control error
- Input is continuous transformed abundance data with no per-feature inferential p-values (e.g., unsupervised clustering or WGCNA-style co-expression outputs)

## Inputs

- Per-gene Wald test p-values from CPLM or ZICP fits (χ² with 1 df under H0: β_g = 0)
- Per-gene coefficient estimates, standard errors, and estimated index parameter p̂
- Pre-specified Type 1 error threshold α (e.g., 0.05)

## Outputs

- FDR-adjusted q-value for every tested gene
- Binary DE call per gene (q-value < 0.05)
- Results table with coefficient, p-value, q-value, and p̂ columns per gene (e.g., zicp_de_results.csv, power_fdr_comparison_results.csv)

## How to apply

After fitting each gene's Tweedie or zero-inflated Tweedie model (via profile likelihood and Newton-Raphson, constrained to ϕ > 0 and p ∈ (1,2)) and computing the Wald statistic for H0: β_g = 0, obtain the p-value from the asymptotic χ²-distribution with 1 degree of freedom. Apply FDR adjustment across all genes' p-values to obtain q-values, then infer a gene to be differentially expressed if its q-value falls below the pre-specified threshold α = 0.05. In the benchmarking context, this adjustment is applied identically to every competing method (DESeq2, edgeR, MAST, DEsingle, etc.) so that statistical power and false discovery rate are aggregated on comparable q < 0.05 calls. The rationale is that thousands of genes are tested simultaneously per dataset, so raw Wald p-values would yield an uncontrolled false discovery rate.

## Related tools

- **Tweedieverse** (R/Bioconductor package that fits per-gene CPLM/ZICP models, computes Wald p-values, and returns multiplicity-adjusted q-values) — https://github.com/himelmallick/Tweedieverse
- **BenchmarkSingleCell** (Hosted benchmarking code whose evaluator produces tables of adjusted p-values (q-values after FDR correction) for every method and scenario) — https://github.com/himelmallick/BenchmarkSingleCell

## Examples

```
qvals <- p.adjust(wald_pvalues, method = "fdr"); de_genes <- names(qvals)[qvals < 0.05]
```

## Evaluation signals

- Output table contains coefficient, p-value, q-value, and p̂ columns for every tested gene
- All q-values lie within [0,1], and DE calls are defined by q < 0.05
- Number of q-values equals number of tested genes (genes retained after filtering, e.g., expressed in ≥20% of cells)
- When benchmarking, the same FDR adjustment is applied to all compared methods so aggregated false discovery rate across synthetic and published datasets is directly comparable

## Limitations

- Tweedieverse is not architected for the situation when zero-inflation is a property specific only to a subset of the genes, which can affect power in heterogeneously zero-inflated data
- The paper notes no borrowed-information / shrinkage estimates for improved DE gene ranking, so q-value ranking may be suboptimal relative to methods that pool information across genes

## Evidence

- [methods] infer a gene to be differentially expressed between two experimental conditions if the FDR-adjusted p-value (q-value) falls below a pre-specified Type 1 error threshold (e.g., α = 0.05): "infer a gene to be differentially expressed between two experimental conditions if the FDR-adjusted p-value (q-value) falls below a pre-specified Type 1 error threshold (e.g., α = 0.05)"
- [readme] Table of marginal p-values, adjusted p-values (q-values after FDR correction), fold change for each feature for every scenario foe every method: [evidence span withheld — anchor status: not_found]
