---
name: multiple-testing-correction
description: Use when performing many parallel statistical tests (e.g., one t test
  per feature across a multi-feature matrix) and you need adjusted p-values to identify
  which features genuinely differ between groups.
when_to_use_negative:
- When only one or a few pre-specified hypotheses are tested — a single uncorrected
  test does not require BH correction
- When the input is not yet a numeric feature matrix split into defined groups (e.g.,
  raw TCGA or MiOncoCirc expression data) — compute the CRIT features and NMF clustering
  first
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3517
tools:
- name: R
  role: Environment used to run the per-feature two-sample Welch t tests and apply
    the Benjamini-Hochberg correction across the 51 features
- name: CRIT
  role: Produces the 51-feature matrix (MATRIX.txt) and NMF cluster labels that supply
    the input groups for the multiple testing
  repo: https://github.com/BioinformaticsSTU/CRIT
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:3f0d01bab9dad428af6475a9d78eec6d834e439001cd918ce117ef133117d11d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BioinformaticsSTU/CRIT
  repo_url: https://github.com/BioinformaticsSTU/CRIT
  tools:
  - CRIT
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# multiple-testing-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Control the false discovery rate when testing many features at once (e.g., 51 CRIT features with per-feature Welch t tests) by applying a Benjamini-Hochberg correction and ranking features by adjusted significance.

## When to use

Use when performing many parallel statistical tests (e.g., one t test per feature across a multi-feature matrix) and you need adjusted p-values to identify which features genuinely differ between groups.

## When NOT to use

- When only one or a few pre-specified hypotheses are tested — a single uncorrected test does not require BH correction
- When the input is not yet a numeric feature matrix split into defined groups (e.g., raw TCGA or MiOncoCirc expression data) — compute the CRIT features and NMF clustering first

## Inputs

- MATRIX.txt: merged candidate-regulator feature matrix with 51 features (ρm, Zp, Zs, Zg, C value columns) for the 1006 RBPs
- NMF cluster membership labels distinguishing the 90 predicted RBPs from the remaining 916 RBPs

## Outputs

- feature_ttest_results.csv: per-feature t-test table with test statistics, raw p-values, and Benjamini-Hochberg adjusted p-values
- Ranked list of features by adjusted significance (expected top hits: Zg and Zp)

## How to apply

Run an independent two-sample Welch t test for each of the 51 CRIT features (ρm, Zp, Zs, Zg, C value) between the predicted-regulator group (NMF cluster membership) and the non-predicted RBPs. Collect the raw p-values for all 51 features and apply the Benjamini-Hochberg correction across the family of tests rather than interpreting any single test in isolation. Rank features by adjusted significance (adjusted p < 0.05 as the acceptance threshold) and write all test statistics, raw p-values, and BH-adjusted p-values to feature_ttest_results.csv. Validation: the output is accepted when Zg (GO:0000398 splicing similarity) and Zp (alteration proportion Z-score) emerge as significantly different between groups (adjusted p < 0.05), consistent with the finding that authentic circRNA regulators are more likely core splicing proteins and peripheral factors harboring more alterations.

## Related tools

- **R** (Environment used to run the per-feature two-sample Welch t tests and apply the Benjamini-Hochberg correction across the 51 features)
- **CRIT** (Produces the 51-feature matrix (MATRIX.txt) and NMF cluster labels that supply the input groups for the multiple testing) — https://github.com/BioinformaticsSTU/CRIT

## Examples

```
res <- t.test(Zg ~ group, data = feat); pvals <- sapply(featCols, function(f) t.test(feat[[f]] ~ feat$group)$p.value); padj <- p.adjust(pvals, method = "BH")
```

## Evaluation signals

- Adjusted p-values (BH) for all 51 features are present and monotone-consistent with raw p-values (adjusted ≥ raw)
- Zg and Zp show adjusted p < 0.05 between the predicted-regulator and non-predicted groups
- The table contains one row per CRIT feature (51 rows) with test statistic, raw p-value, and adjusted p-value columns
- Directionality checks: predicted regulators show higher Zg (splicing similarity) and higher Zp (alteration proportion) than non-predicted RBPs
- Group sizes match the NMF cluster labels (90 predicted vs 916 remaining RBPs)

## Limitations

- Welch t tests assume approximate normality of feature distributions; highly skewed CRIT features may need transformation or non-parametric alternatives
- The corrected result is specific to the 90-vs-916 NMF grouping; other cluster resolutions change the tested hypothesis family
- Benjamini-Hochberg controls FDR across the 51 tests only; findings do not generalize to other diseases (e.g., neurodegenerative disease) as noted as untested in the discussion

## Evidence

- [other] In R, run a two-sample Welch t test independently for each of the 51 features: [evidence span withheld — anchor status: not_found]
- [other] Apply Benjamini-Hochberg correction across the 51 features: [evidence span withheld — anchor status: not_found]
- [other] accept the output when the per-feature t-test table shows Zg and Zp significantly different between groups (adjusted p < 0.05): [evidence span withheld — anchor status: not_found]
