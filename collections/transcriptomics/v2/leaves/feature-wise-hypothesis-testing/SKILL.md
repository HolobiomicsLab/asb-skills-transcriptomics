---
name: feature-wise-hypothesis-testing
description: Use when you have a merged feature matrix (e.g. CRIT's MATRIX.txt) plus
  group labels (e.g. NMF cluster membership separating 90 predicted regulators from
  916 other RBPs) and need to identify which features distinguish the two groups.
when_to_use_negative:
- The goal is to generate the features themselves (Zp, Zs, Zg, ρm, C value) rather
  than test which features separate groups — use CRIT's GET_*.R scripts instead
- The input is a raw feature-matrix candidate list without group labels — you first
  need NMF clustering output to define the regulator vs non-regulator split
- Features are not approximately continuous/normal within groups, where a non-parametric
  alternative (e.g. Wilcoxon rank-sum) would be more defensible than a Welch t test
edam_operation: http://edamontology.org/operation_2234
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0622
tools:
- name: R
  role: Implements the per-feature two-sample Welch t tests and Benjamini-Hochberg
    correction across the 51 CRIT features
- name: CRIT
  role: Produces the feature matrix (MATRIX.txt) and NMF cluster labels that define
    the two comparison groups
  repo: https://github.com/BioinformaticsSTU/CRIT
- name: GoSemSim
  role: Computes the Zg feature (Z-score of GO similarity to GO:0000398 mRNA splicing,
    via spliceosome) that is tested between groups
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:2bd53e4ba8be79fa07f7e948fa165b40dfec1a57ded381857d6e7fee6d6149fb
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
  - GoSemSim
  - R
  edam_operation: http://edamontology.org/operation_2234
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# Feature-wise hypothesis testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs independent two-sample Welch t tests for each of the 51 CRIT features (ρm, Zp, Zs, Zg, C value) between predicted circRNA-regulating RBPs and other RBPs, then applies Benjamini-Hochberg correction to rank features by adjusted significance.

## When to use

Use when you have a merged feature matrix (e.g. CRIT's MATRIX.txt) plus group labels (e.g. NMF cluster membership separating 90 predicted regulators from 916 other RBPs) and need to identify which features distinguish the two groups.

## When NOT to use

- The goal is to generate the features themselves (Zp, Zs, Zg, ρm, C value) rather than test which features separate groups — use CRIT's GET_*.R scripts instead
- The input is a raw feature-matrix candidate list without group labels — you first need NMF clustering output to define the regulator vs non-regulator split
- Features are not approximately continuous/normal within groups, where a non-parametric alternative (e.g. Wilcoxon rank-sum) would be more defensible than a Welch t test

## Inputs

- MATRIX.txt — merged candidate-regulator feature matrix with 51 CRIT feature columns (ρm, Zp, Zs, Zg, C value)
- NMF cluster membership labels (e.g. NMFgroup_XXclusterYYrun.txt) distinguishing predicted regulators (90 RBPs) from non-predicted RBPs (916)

## Outputs

- feature_ttest_results.csv — per-feature Welch t test statistics, raw p-values, and Benjamini-Hochberg adjusted p-values
- Ranked feature list with Zg and Zp expected at top (adjusted p < 0.05)

## How to apply

Load the merged candidate-regulator feature matrix (MATRIX.txt) containing the 51 CRIT feature columns (ρm, Zp, Zs, Zg, C value) and the NMF cluster membership labels that distinguish the 90 predicted RBPs from the remaining 916 RBPs. Split rows into predicted-regulator and non-predicted groups using the cluster labels. In R, run a two-sample Welch t test independently for each of the 51 features between the two groups (unpaired, no equal-variance assumption). Apply Benjamini-Hochberg correction across all 51 features and rank them by adjusted p-value. Verify that Zg (GO:0000398 mRNA splicing similarity, computed via GoSemSim) and Zp (TCGA alteration proportion Z-score) emerge as top distinguishing features with adjusted p < 0.05, and write all test statistics, p-values, and adjusted p-values to feature_ttest_results.csv.

## Related tools

- **R** (Implements the per-feature two-sample Welch t tests and Benjamini-Hochberg correction across the 51 CRIT features)
- **CRIT** (Produces the feature matrix (MATRIX.txt) and NMF cluster labels that define the two comparison groups) — https://github.com/BioinformaticsSTU/CRIT
- **GoSemSim** (Computes the Zg feature (Z-score of GO similarity to GO:0000398 mRNA splicing, via spliceosome) that is tested between groups)

## Examples

```
mtx <- read.delim('MATRIX.txt'); res <- apply(mtx[, feature_cols], 2, function(f) t.test(f ~ mtx$predicted_regulator)); p.adjust(sapply(res, function(r) r$p.value), method='BH')
```

## Evaluation signals

- Zg and Zp show significant between-group differences with Benjamini-Hochberg adjusted p < 0.05
- feature_ttest_results.csv contains one row per feature (51 rows) with test statistic, raw p-value, and adjusted p-value
- Group split reproduces the NMF result: 90 predicted RBPs vs 916 remaining RBPs
- Interpretation matches the article: true regulators are more likely core splicing proteins and peripheral factors (high Zg) and harbor more alterations in most cancer datasets (high Zp)

## Limitations

- Welch t tests assume approximate normality per group; features are Z-scored but tail behavior across only 90 regulators may be unstable
- Multiple-testing correction spans 51 correlated features, so BH-adjusted significance may understate features correlated with Zg/Zp
- The distinguishing features reflect cancer-derived data (TCGA alterations, MiOncoCirc correlations); generalization to other diseases such as neurodegenerative disease is untested per the article's discussion

## Evidence
