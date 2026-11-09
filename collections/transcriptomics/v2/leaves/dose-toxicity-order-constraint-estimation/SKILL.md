---
name: dose-toxicity-order-constraint-estimation
description: Use when per-dose DLT proportions from a dose-escalation trial violate
  the monotone increasing toxicity constraint and monotone (isotonic) toxicity estimates
  are needed for dose-escalation/de-escalation decisions.
when_to_use_negative:
- The response outcome requires long follow-up to assess or is not binary and quickly
  observable — the TSNP framework is limited to binary, shortly observable outcomes
- Raw per-dose DLT proportions already satisfy the monotone increasing order constraint
  — no pooling is needed and estimates equal the raw proportions
- The goal is to model joint toxicity-efficacy probabilities parametrically rather
  than obtain nonparametric closed-form toxicity estimates
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Implementation language for the PAVA isotonic regression computation of per-dose
    DLT estimates within the TSNP design code
  repo: https://github.com/yongzang2020/TSNP
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pst.2075
    title: '<scp>TSNP</scp>: A <scp>two‐stage</scp> nonparametric phase I/<scp>II</scp>
      clinical trial design for immunotherapy'
  source_content_hash: sha256:35351452ebf66918ab132db76cb77090a5be5cd0fa2f107088d2e771a467898a
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
n_claims: 9
---

# dose-toxicity-order-constraint-estimation

## Summary

Computes monotonicity-constrained per-dose DLT rate estimates using the pool-adjacent-violators algorithm (PAVA), where the isotonic estimates are equivalent to the maximum likelihood estimates of the toxicity probabilities p_j under the order constraint p1 ≤ p2 ≤ ... ≤ pJ in the TSNP phase I/II design.

## When to use

Use when per-dose DLT proportions from a dose-escalation trial violate the monotone increasing toxicity constraint and monotone (isotonic) toxicity estimates are needed for dose-escalation/de-escalation decisions.

## When NOT to use

- The response outcome requires long follow-up to assess or is not binary and quickly observable — the TSNP framework is limited to binary, shortly observable outcomes
- Raw per-dose DLT proportions already satisfy the monotone increasing order constraint — no pooling is needed and estimates equal the raw proportions
- The goal is to model joint toxicity-efficacy probabilities parametrically rather than obtain nonparametric closed-form toxicity estimates

## Inputs

- Per-dose toxicity outcome data: number of patients n_j and DLT counts at each dose level j = 1..J
- Raw per-dose DLT proportions p̃_j (derived from the counts)

## Outputs

- Monotone increasing isotonic DLT rate estimates p̂_j for all J dose levels (e.g., isotonic_dlt_estimates.csv)

## How to apply

Load the per-dose toxicity data (n_j patients and DLT counts per dose) and compute raw proportions p̃_j. If the raw proportions already satisfy p1 ≤ p2 ≤ ... ≤ pJ, set the isotonic estimates equal to them; otherwise apply PAVA, replacing each adjacent violating pair with the sample-size weighted average (n_j*p̃_j + n_{j+1}*p̃_{j+1})/(n_j + n_{j+1}) and repeating until monotonicity holds for all doses. The resulting isotonic estimates p̂_j are the MLEs of p_j under the order constraint, so they are statistically optimal, not ad hoc smoothing. In the TSNP design, the p̂_j are then compared with the target toxicity φ−τ to escalate, de-escalate, or retain the current dose. Write the final p̂_j (e.g., to isotonic_dlt_estimates.csv) for downstream stage-2 utility computations.

## Related tools

- **R** (Implementation language for the PAVA isotonic regression computation of per-dose DLT estimates within the TSNP design code) — https://github.com/yongzang2020/TSNP

## Examples

```
Rscript -e 'p <- c(0.1, 0.25, 0.2, 0.3333); iso <- isotone::gpava(seq_along(p), p)$x; write.csv(iso, "isotonic_dlt_estimates.csv")'
```

## Evaluation signals

- The output vector p̂_j satisfies p̂_1 ≤ p̂_2 ≤ ... ≤ p̂_J (monotone increasing order constraint holds)
- Each pooled estimate equals the sample-size weighted average of the raw proportions of the pooled block, preserving the total observed DLT count (n_j*p̃_j sums are conserved)
- Where raw proportions already satisfy the constraint, isotonic estimates equal the raw proportions (PAVA is a no-op)
- Isotonic estimates coincide with the constrained MLEs of p_j, so a likelihood check under the order constraint matches the PAVA solution
- Worked example reproduces known values, e.g., the example trial's isotonic estimate p̂_4 = 1/3

## Limitations

- PAVA only enforces the order constraint; it does not address dose ordering uncertainty or latency in toxicity ascertainment
- Estimates are nonparametric and data-driven, so with small per-dose sample sizes early in stage 1 the estimates can be unstable until m patients have been treated at the current dose
- The skill addresses toxicity probabilities only; efficacy and joint toxicity-efficacy probabilities are handled in the stage-2 utility step of TSNP

## Evidence
