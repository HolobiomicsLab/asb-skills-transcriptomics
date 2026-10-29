---
name: tcga-alteration-feature-interpretation
description: Use when you have TCGA alteration data (CNV, mutations) for RBPs and
  need to quantify alteration burden (Zp) or test whether predicted circRNA regulators
  differ from non-regulators in alteration features.
when_to_use_negative:
- Input is already a computed Zp / Z-score feature table — skip recomputation and
  go directly to the group comparison
- The tissue set is not from TCGA cancer batches (e.g., non-cancer or neurodegenerative
  cohorts) — the dichotomized-alteration Z-score is calibrated to TCGA's 19 batches
  of 12 cancer types and CRIT's generalization elsewhere is untested
- You need RBP-circRNA binding evidence rather than alteration burden — use the C
  value feature from RNAct/catRAPID instead
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CRIT
  role: Pipeline that computes the Zp feature (GET_Prop_zscore.R), merges features
    into MATRIX.txt, and clusters candidate RBPs via NMF
  repo: https://github.com/BioinformaticsSTU/CRIT
- name: R
  role: Language used to run the two-sample Welch t tests per feature and Benjamini-Hochberg
    correction across 51 CRIT features
- name: survival
  role: R package used by the companion Zs log-rank survival analysis, whose workflow
    parallels the Zp alteration-proportion computation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:8d51a238d9c403f4073c835888c9f1dcbe02121b07345d65681d71c2cbd91494
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
  - survival
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# tcga-alteration-feature-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute and statistically interpret the CRIT Zp feature — the Z-score of gene alteration proportion derived from dichotomized TCGA data (19 batches, 12 cancer types) — and use it to test whether circRNA-regulating RBPs harbor more alterations than other RBPs.

## When to use

Use when you have TCGA alteration data (CNV, mutations) for RBPs and need to quantify alteration burden (Zp) or test whether predicted circRNA regulators differ from non-regulators in alteration features.

## When NOT to use

- Input is already a computed Zp / Z-score feature table — skip recomputation and go directly to the group comparison
- The tissue set is not from TCGA cancer batches (e.g., non-cancer or neurodegenerative cohorts) — the dichotomized-alteration Z-score is calibrated to TCGA's 19 batches of 12 cancer types and CRIT's generalization elsewhere is untested
- You need RBP-circRNA binding evidence rather than alteration burden — use the C value feature from RNAct/catRAPID instead

## Inputs

- TCGA per-cancer .txt files: data_mRNA_median_Zscores.txt (or data_RNA_Seq_v2_mRNA_median_Zscores.txt), data_CNA.txt, data_mutations_extended.txt, data_clinical_patient.txt
- Merged CRIT feature matrix (MATRIX.txt) containing Zp, Zs, Zg, ρm, and C value columns
- NMF cluster membership labels distinguishing the 90 predicted regulator RBPs from the remaining 916 RBPs

## Outputs

- XX_proportion_Z_Score.txt — feature Zp of each batch of every cancer type
- feature_ttest_results.csv — per-feature Welch t-test statistics, p-values, and BH-adjusted p-values
- Ranked list of discriminating features with Zp and Zg expected as top hits (adjusted p < 0.05)

## How to apply

For each cancer type or batch, dichotomize the raw TCGA data files (data_CNA.txt, data_mutations_extended.txt, data_mRNA_median_Zscores.txt) to obtain each gene's alteration status, then compute the proportion of samples altered per gene and convert it to a Z-score — CRIT implements this in GET_Prop_zscore.R, writing 'XX_proportion_Z_Score.txt'. Merge Zp with the other CRIT features (ρm, Zs, Zg, C value) into MATRIX.txt via GET_feauters_MATRIX.R. To interpret Zp's role, split genes into the predicted-regulator cluster (NMF labels, 90 RBPs) versus the remaining RBPs and run an independent two-sample Welch t test per feature in R. Apply Benjamini-Hochberg correction across all tested features and flag those with adjusted p < 0.05. The expected result, matching the paper, is that Zp (together with Zg, the GO:0000398 splicing-similarity Z-score) emerges as a top discriminator because authentic regulators harbor more alterations in the vast majority of cancers.

## Related tools

- **CRIT** (Pipeline that computes the Zp feature (GET_Prop_zscore.R), merges features into MATRIX.txt, and clusters candidate RBPs via NMF) — https://github.com/BioinformaticsSTU/CRIT
- **R** (Language used to run the two-sample Welch t tests per feature and Benjamini-Hochberg correction across 51 CRIT features)
- **survival** (R package used by the companion Zs log-rank survival analysis, whose workflow parallels the Zp alteration-proportion computation)

## Examples

```
M <- read.delim('MATRIX.txt'); grp <- read.delim('NMFgroup_labels.txt'); t.test(Zp ~ is_regulator, data = merge(M, grp))
```

## Evaluation signals

- Zp column in feature_ttest_results.csv shows adjusted p < 0.05 between predicted-regulator and non-regulator groups
- Zg and Zp rank as the top two significant features after Benjamini-Hochberg correction across the 51 features
- Recomputed XX_proportion_Z_Score.txt values are consistent between the regulator and non-regulator groups' directionality (regulators higher), matching the paper's finding
- Group sizes reconcile with NMF labels: 90 predicted RBPs vs the remaining 916 RBPs (1006 total genes)
- Per-batch Zp outputs exist for all 19 TCGA batches across the 12 cancer types

## Limitations

- CRIT's strategy, including the TCGA alteration-based Zp feature, was developed and validated only in cancers; generalization to other diseases (e.g., neurodegenerative disease) is untested
- The finding that regulators harbor more alterations holds 'in the vast majority of cancers', not necessarily every cancer type or batch
- CNV, mutation, and mRNA-expression dichotomization choices in TCGA files determine alteration proportion; mis-dichotomization propagates into Zp

## Evidence
