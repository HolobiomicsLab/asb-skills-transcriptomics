---
name: phase-i-dlt-rate-estimation
description: Use when dose-escalation decisions in a phase I/II trial require per-dose
  DLT probability estimates that are guaranteed monotonically non-decreasing across
  dose levels, from raw cohort DLT counts.
when_to_use_negative:
- The goal is joint toxicity-efficacy (bivariate) estimation in stage 2 of TSNP rather
  than univariate per-dose DLT rates
- Response/toxicity outcomes are not binary and quickly observable — the TSNP design
  cannot be directly applied when the response outcome requires long follow-up time
- Only one dose level has data, so no ordering constraint across doses can be violated
  and pooling is unnecessary
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Implementation language for the TSNP design's simulation and trial code, including
    the PAVA-based isotonic estimation step
  repo: https://github.com/yongzang2020/TSNP
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pst.2075
    title: '<scp>TSNP</scp>: A <scp>two‐stage</scp> nonparametric phase I/<scp>II</scp>
      clinical trial design for immunotherapy'
  source_content_hash: sha256:28c72d169bad9e1f3473ebed314b0923dbcef321dea6652c031bfb60a1f935b4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yongzang2020/TSNP
  tools:
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# phase-i-dlt-rate-estimation

## Summary

Computes per-dose dose-limiting toxicity (DLT) rate estimates in the TSNP phase I/II design by applying the pool-adjacent-violators algorithm (PAVA) to observed per-dose toxicity proportions, yielding isotonic estimates that satisfy the monotonic increasing order constraint on toxicity rates and are equivalent to the constrained MLEs.

## When to use

Use when dose-escalation decisions in a phase I/II trial require per-dose DLT probability estimates that are guaranteed monotonically non-decreasing across dose levels, from raw cohort DLT counts.

## When NOT to use

- The goal is joint toxicity-efficacy (bivariate) estimation in stage 2 of TSNP rather than univariate per-dose DLT rates
- Response/toxicity outcomes are not binary and quickly observable — the TSNP design cannot be directly applied when the response outcome requires long follow-up time
- Only one dose level has data, so no ordering constraint across doses can be violated and pooling is unnecessary

## Inputs

- Per-dose cohort toxicity outcome data: number of patients n_j treated at each dose level j and observed DLT counts
- Target toxicity probability φ and escalation/de-escalation offset τ (TSNP stage 1 parameters)

## Outputs

- Isotonic (monotonicity-constrained) DLT rate estimates p̂_j for all dose levels, e.g. written to isotonic_dlt_estimates.csv
- Updated admissible set Ω = {j: rj ≥ cT} via Fisher's exact test p-values rj against the toxicity cutoff cT

## How to apply

Load the per-dose toxicity data (number of patients n_j and DLT counts per dose j) and compute the raw proportions p~_j. If the raw proportions already satisfy the order constraint p1 ≤ p2 ≤ ... ≤ pJ, adopt them directly as the isotonic estimates p̂_j. Whenever adjacent doses violate the constraint, apply PAVA: replace the violating pair with the sample-size weighted average (n_j*p~_j + n_{j+1}*p~_{j+1})/(n_j + n_{j+1}) and repeat pooling until the monotonically increasing restriction holds for all doses. The resulting isotonic estimates p̂_j are then used downstream in the TSNP design to drive escalation/de-escalation (comparing p̂_j against φ−τ) and the toxicity admissibility test (Fisher's exact test of H0: pj ≤ φ). Write the final estimates, e.g. to isotonic_dlt_estimates.csv; in the article's example trial this yields p̂_4 = 1/3 as an isotonic estimate.

## Related tools

- **R** (Implementation language for the TSNP design's simulation and trial code, including the PAVA-based isotonic estimation step) — https://github.com/yongzang2020/TSNP

## Examples

```
Rscript TSNP_simulation.R  # run the TSNP R code from github.com/yongzang2020/TSNP; PAVA yields isotonic DLT estimates p̂_j satisfying p1 ≤ ... ≤ pJ (example: p̂_4 = 1/3)
```

## Evaluation signals

- The output vector p̂_j satisfies the monotonic increasing constraint p̂_1 ≤ p̂_2 ≤ ... ≤ p̂_J exactly
- Each isotonic estimate equals the sample-size weighted mean of the raw proportions of all doses pooled together, preserving total DLTs (sum n_j*p̂_j = total observed DLTs)
- Where raw proportions already satisfy the order constraint, the isotonic estimates are identical to the raw per-dose DLT proportions
- The isotonic estimates reproduce the constrained MLEs — equivalence to MLE of pj under the order constraint (1)
- Reproducible example: the article's example trial yields p̂_4 = 1/3 as an isotonic estimate

## Limitations

- The TSNP design, including this estimation step, is appropriate only for trials where the response outcome is binary and observable shortly after treatment initiation
- Extension to drug-combination trials (dose combinations requiring a different ordering/pooling structure) is future work and not supported
- No named software package or function is documented in the article's methods section; users must rely on the R code in the GitHub repository

## Evidence
