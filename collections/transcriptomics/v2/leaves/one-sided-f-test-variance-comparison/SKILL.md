---
name: one-sided-f-test-variance-comparison
description: Use when two quantification methods have been benchmarked against a common
  ground truth (e.g., long-read TE counts) and you must show one method's residuals
  are significantly smaller, not just descriptively better.
when_to_use_negative:
- When the comparison is two-directional (no prior hypothesis that MATES outperforms
  the baseline) — a two-sided variance test is then the appropriate choice
- When only locus-level rather than subfamily-level TE counts are being validated,
  since the reported reference values (e.g., 0.7531 vs 0.6499 vs 0.6841) are subfamily-level
- When no common ground truth exists to define residual variances for both methods
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: MATES
  role: Method under test whose quantification residuals are compared against baselines
    via the F statistic
  repo: https://github.com/mcgilldinglab/MATES
- name: scTE
  role: Baseline single-cell TE quantifier whose residual variance is tested against
    MATES
- name: SoloTE
  role: Baseline single-cell TE quantifier whose residual variance is tested against
    MATES
- name: TE_quantifier_LongRead
  role: MATES module for extracting long-read ground-truth TE counts used to compute
    the residuals being tested
  repo: https://github.com/mcgilldinglab/MATES
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:eb66c939763146280b80748258dcb8b20c39e9a9a7c752b211c00a6ae03d7067
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mcgilldinglab/MATES
  repo_url: https://github.com/mcgilldinglab/MATES
  tools:
  - MATES
  - SoloTE
  - TE_quantifier
  - TE_quantifier_LongRead
  - scTE
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# One-sided F-test variance comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies a one-sided F-test on residual variances to attach statistical significance (p < 0.05) to differences in short-read TE quantification accuracy between methods (MATES vs scTE and SoloTE), complementing R2 comparison against long-read ground truth.

## When to use

Use when two quantification methods have been benchmarked against a common ground truth (e.g., long-read TE counts) and you must show one method's residuals are significantly smaller, not just descriptively better.

## When NOT to use

- When the comparison is two-directional (no prior hypothesis that MATES outperforms the baseline) — a two-sided variance test is then the appropriate choice
- When only locus-level rather than subfamily-level TE counts are being validated, since the reported reference values (e.g., 0.7531 vs 0.6499 vs 0.6841) are subfamily-level
- When no common ground truth exists to define residual variances for both methods

## Inputs

- Subfamily-level TE count matrices for each method (MATES, scTE, SoloTE) on matched short-read single-cell data
- Ground-truth subfamily-level TE counts from long-read sequencing (scNanoRNAseq nanopore melanoma brain metastasis data and PacBio postnatal mouse brain data)
- Computed R2 residual vectors per method per platform (single-cell nanopore, pseudo-bulk nanopore, PacBio)

## Outputs

- te_quantification_benchmark_r2.csv with columns: method, platform (nanopore single-cell, nanopore pseudo-bulk, PacBio), R2, p-value
- One-sided F-test p-values supporting MATES improvement over each baseline method per platform

## How to apply

First compute subfamily-level R2 values for each method (MATES, scTE, SoloTE) against long-read ground truth separately for each setting — single-cell nanopore, pseudo-bulk nanopore, and PacBio. For each comparison setting, take the residuals of each baseline method's quantification against ground truth, form the F statistic as the ratio of the baseline method's residual variance to MATES's residual variance, and compute a one-sided F-test p-value with the appropriate residual degrees of freedom. This one-sided formulation is chosen because the hypothesis is directional (MATES is more accurate), aligning with the validation requirement that MATES > scTE and MATES > SoloTE in all three settings. Reproduced R2 values must match reported references within a tolerance of 0.01 (e.g., MATES 0.7531 vs scTE 0.6499 vs SoloTE 0.6841 for single-cell nanopore), and significance is declared at F-test p < 0.05. Write all R2 values and F-test p-values to te_quantification_benchmark_r2.csv with columns method, platform, R2, and p-value.

## Related tools

- **MATES** (Method under test whose quantification residuals are compared against baselines via the F statistic) — https://github.com/mcgilldinglab/MATES
- **scTE** (Baseline single-cell TE quantifier whose residual variance is tested against MATES)
- **SoloTE** (Baseline single-cell TE quantifier whose residual variance is tested against MATES)
- **TE_quantifier_LongRead** (MATES module for extracting long-read ground-truth TE counts used to compute the residuals being tested) — https://github.com/mcgilldinglab/MATES

## Examples

```
Rscript -e 'f <- var(res_scte)/var(res_mates); p <- pf(f, df1, df2, lower.tail=FALSE); write.csv(data.frame(method="scTE",platform=p,R2=r2_scte,p=p), "te_quantification_benchmark_r2.csv", append=TRUE)'
```

## Evaluation signals

- Each reproduced R2 matches the reported reference value within a tolerance of 0.01 (single-cell nanopore: MATES 0.7531, scTE 0.6499, SoloTE 0.6841; pseudo-bulk nanopore: 0.9276/0.8591/0.8654; PacBio: 0.5096/0.3537/0.4199)
- MATES R2 > scTE R2 and MATES R2 > SoloTE R2 in all three settings
- One-sided F-test p < 0.05 for the MATES improvement over each baseline in each setting
- te_quantification_benchmark_r2.csv contains exactly the columns method, platform, R2, p-value with rows covering all three platforms

## Limitations

- MATES requires more computational resources such as GPUs than its counterparts scTE and SoloTE, which may limit rerunning the benchmark for reproducibility checks
- Long-read ground truth itself may misestimate TEs longer than the 2001 bp coverage vector span, inflating residual variance for all methods
- MATES has limited ability to quantify long autonomously expressed TEs such as L1HS spanning several thousand bases, which can dominate residuals for those subfamilies

## Evidence

- [other] One-sided F-test on residual variances: [evidence span withheld — anchor status: not_found]
- [other] R2 reference values and tolerance: [evidence span withheld — anchor status: not_found]
- [results] MATES superiority finding: [evidence span withheld — anchor status: not_found]
