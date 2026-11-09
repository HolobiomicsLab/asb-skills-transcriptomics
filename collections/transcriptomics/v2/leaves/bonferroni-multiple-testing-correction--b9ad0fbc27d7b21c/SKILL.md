---
name: bonferroni-multiple-testing-correction
description: Use when a per-gene model (e.g., the negative binomial GLM with Time,
  Group, and Time-by-Group interaction) yields thousands of raw p-values that must
  be Bonferroni-adjusted before calling tDEGs at p.cutoff = 0.05.
when_to_use_negative:
- When the analysis goal is expected-discovery (FDR) control rather than family-wise
  error control — Bonferroni is the strictest adjustment and CASi applies it specifically
  to support the p.cutoff = 0.05 tDEG criterion
- When there are no repeated statistical tests to correct (a single model fit), since
  Bonferroni adjustment requires a defined family of tests
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CASi
  role: R package whose FindTemporalDEGs function fits the per-gene negative binomial
    GLM and applies the Bonferroni correction across tested genes to identify tDEGs
  repo: github.com/yizhuo-wang/CASi
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-024-58566-x
    title: 'CASi: A framework for cross-timepoint analysis of single-cell RNA sequencing
      data'
  source_content_hash: sha256:29f724dd3c40ce12a0aad8440a0bd906af0e07c6d3ea565477cb58d7180f4d21
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CASi
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# bonferroni-multiple-testing-correction

## Summary

Family-wise multiple-testing correction applied to per-gene p-values from CASi's negative binomial GLM in FindTemporalDEGs, so that temporal differentially expressed genes (tDEGs) are called with p values smaller than the p.cutoff of 0.05 after adjustment across all tested genes.

## When to use

Use when a per-gene model (e.g., the negative binomial GLM with Time, Group, and Time-by-Group interaction) yields thousands of raw p-values that must be Bonferroni-adjusted before calling tDEGs at p.cutoff = 0.05.

## When NOT to use

- When the analysis goal is expected-discovery (FDR) control rather than family-wise error control — Bonferroni is the strictest adjustment and CASi applies it specifically to support the p.cutoff = 0.05 tDEG criterion
- When there are no repeated statistical tests to correct (a single model fit), since Bonferroni adjustment requires a defined family of tests

## Inputs

- Vector of raw per-gene p-values from negative binomial GLM fits (time and interaction coefficients)
- List of filtered genes retained by the detection-rate cutoff (pct.cutoff = 0.3)
- Significance threshold p.cutoff (default 0.05)

## Outputs

- Vector of Bonferroni-adjusted p-values, one per tested gene
- tDEG calls (genes with adjusted p-value < 0.05)
- tdeg_results.csv with columns: gene, coefficients, raw p-value, adjusted p-value, tDEG call

## How to apply

Fit the negative binomial GLM with log link for each gene passing the detection-rate filter (pct.cutoff = 0.3 of cells), extracting the raw p-value for the time/interaction coefficient. Collect raw p-values across all tested genes and apply the Bonferroni correction, i.e., multiply each raw p-value by the number of tested genes (equivalently p.adjust(method='bonferroni')), capping at 1. Call tDEGs as genes whose adjusted p-value is smaller than p.cutoff = 0.05, matching CASi's FindTemporalDEGs criterion. Write the per-gene table (gene, coefficients, raw p-value, adjusted p-value, tDEG call) to tdeg_results.csv so raw and adjusted values can be audited side by side. The correction is what enforces the family-wise error rate over the gene set, since individual gene-level GLM p-values are not valid for genome-wide calling on their own.

## Related tools

- **CASi** (R package whose FindTemporalDEGs function fits the per-gene negative binomial GLM and applies the Bonferroni correction across tested genes to identify tDEGs) — github.com/yizhuo-wang/CASi

## Examples

```
p_adj <- p.adjust(p_values, method = "bonferroni"); tdegs <- names(p_adj)[p_adj < 0.05]
```

## Evaluation signals

- Every adjusted p-value is >= its corresponding raw p-value (Bonferroni monotonicity)
- Number of adjusted p-values equals the number of genes passing the pct.cutoff = 0.3 detection filter
- All genes flagged as tDEGs have p values smaller than 0.05 after correction, per the FindTemporalDEGs criterion
- Bonferroni adjustment reproduces min(raw_p * n_tests, 1) for each gene when recomputed independently
- tdeg_results.csv exists and contains one row per filtered gene with both a raw p-value field and an adjusted p-value field

## Limitations

- The underlying GLM can only detect linear effects, i.e., the linear change of gene expression by time or the interaction of time and condition, so correction only covers linear time/treatment effects
- The FindTemporalDEGs criterion (p < 0.05 after Bonferroni) is strict family-wise control; genes with modest but consistent temporal divergence may not pass

## Evidence

- [methods] FindTemporalDEGs function parameters: "Below are the input parameters of the FindTemporalDEGs function"
- [readme] tDEG identification is a CASi capability: "(4) identification of temporal differentially expressed genes (tDEGs)."
- [discussion] GLM limitation on detectable effects: "the current GLM framework can only detect linear effect, i.e., the linear change of gene expression by time or the interaction of time and condition"
- [abstract] CASi multi-timepoint scope: "we develop CASi, a comprehensive framework for analyzing multiple timepoints' scRNA-seq data"
