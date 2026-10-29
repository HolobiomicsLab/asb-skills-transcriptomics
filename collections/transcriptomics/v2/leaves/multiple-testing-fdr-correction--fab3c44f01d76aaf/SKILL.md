---
name: multiple-testing-fdr-correction
description: Use when running many parallel tests in the PreCanCell workflow — e.
when_to_use_negative:
- A single pre-specified hypothesis test is being run (no multiplicity to correct)
- Ranking genes purely by effect size or log2FC without any significance claim — correction
  is unnecessary
- The input is already adjusted P-values or a prefiltered significant-gene list; re-correcting
  would double-penalize
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Provides p.adjust()/p.adjust.methods for Benjamini-Hochberg FDR and Bonferroni
    correction of t-test, Spearman, and Wilcoxon P-value families in the PreCanCell
    downstream analyses
  repo: https://github.com/WangX-Lab/PreCanCell
- name: PreCanCell
  role: The R package whose upstream DEG and downstream TCGA/GDSC analyses generate
    the P-value families that require correction
  repo: https://github.com/WangX-Lab/PreCanCell
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.009
    title: 'PreCanCell: An ensemble learning algorithm for predicting cancer and non-cancer
      cells from single-cell transcriptomes'
  source_content_hash: sha256:c0fcf805552e3611e68c1fe8d79122bb580fb451dc1ac534ddeb810a7615591a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WangX-Lab/PreCanCell
  tools:
  - ESTIMATE
  - PreCanCell
  - R
  - Student's t-test
  - ssGSEA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# multiple-testing-fdr-correction

## Summary

Adjust P-values for the many simultaneous hypothesis tests run in the PreCanCell downstream analyses (tumor-vs-normal t-tests across 30 cancer types, GDSC IC50 correlations across 265 compounds) using Benjamini-Hochberg FDR, plus Bonferroni correction for the malignant-vs-non-malignant Wilcoxon DEG tests, so that reported significance (FDR < 0.05) controls false discoveries.

## When to use

Use when running many parallel tests in the PreCanCell workflow — e.g. Student's t-tests of TMG expression in 30 TCGA cancer types, Spearman correlations of TMG/NMG scores with IC50 for 265 GDSC compounds, or Wilcoxon tests over all genes — and per-gene/per-compound P-values need family-wise or FDR adjustment before declaring significance.

## When NOT to use

- A single pre-specified hypothesis test is being run (no multiplicity to correct)
- Ranking genes purely by effect size or log2FC without any significance claim — correction is unnecessary
- The input is already adjusted P-values or a prefiltered significant-gene list; re-correcting would double-penalize

## Inputs

- Vector or table of raw P-values from the family of tests (per-cancer-type t-tests, per-compound Spearman correlations, or per-gene Wilcoxon tests)
- Choice of correction method: 'BH' for FDR or 'bonferroni' for the DEG Wilcoxon tests
- Significance threshold (0.05 on the adjusted scale)

## Outputs

- Adjusted P-values (BH FDR or Bonferroni) aligned one-to-one with the input tests
- Per-cancer-type table of TMG vs normal t-test results with FDR significance flags
- Per-compound table of GDSC Spearman rho, P-value, and FDR significance for TMG and NMG IC50 correlations

## How to apply

Collect the raw P-value vector from the family of tests being run, then apply the correction matched to the analysis: use the Benjamini-Hochberg FDR method (e.g. p.adjust(method='BH') in R) for the TCGA tumor-vs-normal Student's t-tests and for the GDSC per-compound Spearman correlations, and Bonferroni correction for the Seurat FindMarkers() Wilcoxon rank-sum tests that define the 73 TMGs and 186 NMGs. Declare significance only when the adjusted P-value passes the threshold used by the paper (adjusted P < 0.05 / FDR < 0.05; Bonferroni-adjusted Wilcoxon P < 0.05 for DEGs). Tabulate per-item correlation coefficients, raw P-values, and adjusted significance together so direction and coverage can be validated against reported counts. The rationale is that thousands of genes/compounds are tested at once, so raw thresholds would inflate false positives.

## Related tools

- **R** (Provides p.adjust()/p.adjust.methods for Benjamini-Hochberg FDR and Bonferroni correction of t-test, Spearman, and Wilcoxon P-value families in the PreCanCell downstream analyses) — https://github.com/WangX-Lab/PreCanCell
- **PreCanCell** (The R package whose upstream DEG and downstream TCGA/GDSC analyses generate the P-value families that require correction) — https://github.com/WangX-Lab/PreCanCell

## Examples

```
padj <- p.adjust(pvals, method = "BH"); sig <- padj < 0.05
```

## Evaluation signals

- TMG pan-cancer elevation remains significant after BH adjustment: P = 2.22e-39 overall, and FDR < 0.05 in 25 of 30 individual TCGA cancer types with >= 5 normal samples
- GDSC correlation coverage reproduces the reported counts after BH correction: TMG expression significantly positively correlated with IC50 in 212 of 265 compounds and NMG expression significantly negatively correlated in 232 of 265 compounds (FDR < 0.05)
- Adjustment is one-to-one: output adjusted P-value vector has the same length and ordering as the input raw P-value vector
- DEG calling uses Bonferroni-adjusted Wilcoxon rank-sum P < 0.05, and the resulting TMG/NMG intersection yields exactly 73 TMGs and 186 NMGs
- Adjusted P-values are monotonically consistent with raw P-values (no adjusted P smaller than its raw P; BH adjusted values non-decreasing with raw values)

## Limitations

- BH FDR controls expected false discovery proportion, not family-wise error; per-test false-positive risk differs from Bonferroni
- Adjustment strength depends on family definition — correcting across all 265 GDSC compounds differs from correcting within a single compound's test, changing which correlations pass FDR < 0.05
- Reported predictive performance in the PreCanCell training set may be overestimated because the predicted sets were involved in feature selection — correction of downstream P-values does not fix this training-set bias

## Evidence

- [results] TMG pan-cancer and 25/30 cancer-type significance after FDR correction: "TMGs displayed significantly higher expression levels in TCGA pan-cancer (P = 2.22 × 10−39) and in 25 of the 30 individual cancer types with normal sample’s size not less than five (FDR < 0.05)"
- [methods] Bonferroni correction for DEG Wilcoxon tests: "the gene expression difference between malignant and non-malignant cells was statistically significant if the adjusted Wilcoxon rank-sum test P-value < 0.05 by the Bonferroni correction"
- [other] BH FDR applied to GDSC per-compound correlations: [evidence span withheld — anchor status: not_found]
- [other] BH FDR applied to TCGA tumor-vs-normal t-tests: [evidence span withheld — anchor status: not_found]
