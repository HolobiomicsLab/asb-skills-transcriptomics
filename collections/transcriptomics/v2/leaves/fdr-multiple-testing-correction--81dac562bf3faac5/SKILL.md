---
name: fdr-multiple-testing-correction
description: Use when a per-feature hypothesis test (e.g., Wald test on Tweedie/CPLM/ZICP
  model coefficients) produces many p-values that must be FDR-adjusted to make differential-expression
  calls across genes or taxa.
when_to_use_negative:
- When the analysis goal is family-wise error rate control (e.g., Bonferroni-level
  guarantees) rather than FDR control
- When p-values come from a single pre-specified hypothesis rather than a high-dimensional
  multi-feature screen
- When zero-inflation affects only a subset of genes and per-gene model assumptions
  are violated — the article notes Tweedieverse 'is not architected for the situation
  when zero-inflation is a property specific only to a subset of the genes', undermining
  downstream calls
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Tweedieverse
  role: Fits per-feature Tweedie/CPLM/ZICP models, computes Wald test p-values, and
    applies FDR adjustment to call differentially expressed features
  repo: https://github.com/himelmallick/Tweedieverse
- name: R
  role: Implementation language for the Tweedieverse package and its FDR-correction
    workflow
  repo: https://github.com/himelmallick/Tweedieverse
- name: cplm (R package)
  role: Fits the Compound Poisson linear models whose Wald test p-values are FDR-adjusted
  repo: https://github.com/himelmallick/Tweedieverse
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/sim.9430
    title: Differential expression of single‐cell RNA‐seq data using Tweedie models
  source_content_hash: sha256:c19d7a9bfa96b7cc378eb5211eec088b757ef0b0c9796e489a843ea6560e0311
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/himelmallick/Tweedieverse
  tools:
  - R
  - Tweedieverse
  - cplm (R package)
  - statmod (R package)
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# fdr-multiple-testing-correction

## Summary

False discovery rate correction of per-feature Wald test p-values from Tweedieverse differential analysis, calling features differentially expressed when the FDR-adjusted q-value falls below alpha = 0.05.

## When to use

Use when a per-feature hypothesis test (e.g., Wald test on Tweedie/CPLM/ZICP model coefficients) produces many p-values that must be FDR-adjusted to make differential-expression calls across genes or taxa.

## When NOT to use

- When the analysis goal is family-wise error rate control (e.g., Bonferroni-level guarantees) rather than FDR control
- When p-values come from a single pre-specified hypothesis rather than a high-dimensional multi-feature screen
- When zero-inflation affects only a subset of genes and per-gene model assumptions are violated — the article notes Tweedieverse 'is not architected for the situation when zero-inflation is a property specific only to a subset of the genes', undermining downstream calls

## Inputs

- Per-feature Wald test p-values from Tweedie/CPLM/ZICM (ZICP) model fits
- Per-feature coefficient (effect size) estimates from the Compound Poisson linear model
- Number of tested features (genes, taxa, metabolites) for FDR adjustment

## Outputs

- FDR-adjusted p-values (q-values) per feature
- Differential expression calls: features with q-value < alpha = 0.05
- Table of feature-level coefficients, p-values, q-values, and estimated index parameters (e.g., de_results.csv)

## How to apply

For each feature, test H0: beta_g = 0 using the Wald test statistic, which under the null follows an asymptotic chi-squared distribution with 1 degree of freedom (as implemented in the Tweedieverse framework via the cplm, tweedie, and statmod R packages). Collect the marginal per-feature p-values, then adjust them for the false discovery rate to obtain q-values. Call a feature differentially expressed between two experimental conditions if the FDR-adjusted p-value (q-value) falls below the pre-specified Type 1 error threshold, e.g., alpha = 0.05. Output the coefficient, raw p-value, q-value, and estimated index parameter per feature (e.g., to de_results.csv) so calls and effect sizes are auditable together.

## Related tools

- **Tweedieverse** (Fits per-feature Tweedie/CPLM/ZICP models, computes Wald test p-values, and applies FDR adjustment to call differentially expressed features) — https://github.com/himelmallick/Tweedieverse
- **R** (Implementation language for the Tweedieverse package and its FDR-correction workflow) — https://github.com/himelmallick/Tweedieverse
- **cplm (R package)** (Fits the Compound Poisson linear models whose Wald test p-values are FDR-adjusted) — https://github.com/himelmallick/Tweedieverse

## Examples

```
Tweedieverse(feature_count, meta_data, test = "Wald", fdr = "BH")
```

## Evaluation signals

- Output table contains coefficient, p-value, q-value, and index-parameter columns for every feature
- q-values are no smaller than the raw p-values (monotonicity invariant of FDR adjustment)
- Significant calls are exactly those features with q-value < 0.05
- Raw p-values are present alongside q-values so the adjustment is traceable
- BenchmarkSingleCell-style output includes marginal p-values, adjusted p-values (q-values after FDR correction), and fold change for each feature for every method

## Limitations

- The alpha = 0.05 threshold is a pre-specified choice; power and FDR control were benchmarked for Tweedieverse against >10 DE methods, but the extension to other modalities (metagenomics, metatranscriptomics, spatial transcriptomics) 'remains a conjecture, not validated'
- Tweedieverse is not architected for gene-specific zero-inflation affecting only a subset of genes, which can distort per-feature p-values feeding the FDR step
- No borrowed-information/shrinkage estimates are used for improved DE gene ranking, so q-value ranking may not reflect optimal gene prioritization

## Evidence

- [readme] Benchmark output table: [evidence span withheld — anchor status: not_found]
