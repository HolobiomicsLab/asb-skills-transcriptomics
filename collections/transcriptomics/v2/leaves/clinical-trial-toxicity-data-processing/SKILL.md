---
name: clinical-trial-toxicity-data-processing
description: Use when you have per-dose phase I/II toxicity data (patient counts n_j
  and DLT counts) whose raw proportions p̃_j violate the order constraint p1 ≤ p2
  ≤ ... ≤ pJ and you need monotonicity-constrained DLT rate estimates for dose-escalation
  decisions or utility calculations.
when_to_use_negative:
- The trial outcome requires long follow-up time to assess — the TSNP nonparametric
  approach 'cannot be applied directly' to such cases
- The response outcome is not binary and quickly observable after treatment initiation,
  or the trial involves drug combinations (an untested extension in the TSNP design)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0274
tools:
- name: R
  role: Programming language used to implement PAVA and the TSNP design; R code for
    simulation and trial implementation is provided in the TSNP repository
  repo: https://github.com/yongzang2020/TSNP
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pst.2075
    title: '<scp>TSNP</scp>: A <scp>two‐stage</scp> nonparametric phase I/<scp>II</scp>
      clinical trial design for immunotherapy'
  source_content_hash: sha256:5e49e1631d2ec41b468b259671244a670b2ecc8982537fd7d7c77ab886d5d7ca
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yongzang2020/TSNP
  tools:
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0274
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# clinical-trial-toxicity-data-processing

## Summary

Computes per-dose DLT rate estimates that satisfy the monotonic increasing (order) constraint across dose levels by applying the pool-adjacent-violators algorithm (PAVA) to observed per-dose toxicity data. The resulting isotonic estimates are equivalent to the constrained maximum likelihood estimates of the DLT probabilities used by the TSNP phase I/II design.

## When to use

Use when you have per-dose phase I/II toxicity data (patient counts n_j and DLT counts) whose raw proportions p̃_j violate the order constraint p1 ≤ p2 ≤ ... ≤ pJ and you need monotonicity-constrained DLT rate estimates for dose-escalation decisions or utility calculations.

## When NOT to use

- The trial outcome requires long follow-up time to assess — the TSNP nonparametric approach 'cannot be applied directly' to such cases
- The response outcome is not binary and quickly observable after treatment initiation, or the trial involves drug combinations (an untested extension in the TSNP design)

## Inputs

- Per-dose cohort data: number of patients treated n_j and DLT counts at each dose level j = 1,...,J
- Raw per-dose DLT proportions p̃_j computed from the toxicity data

## Outputs

- Monotonicity-constrained isotonic DLT rate estimates p̂_j for all dose levels (e.g., written to isotonic_dlt_estimates.csv)

## How to apply

Load the per-dose toxicity outcome data (number of patients n_j and DLT counts per dose) and compute the raw per-dose DLT proportions p̃_j. Check whether the raw proportions already satisfy the order constraint p1 ≤ p2 ≤ ... ≤ pJ; if so, set the isotonic estimates equal to the raw proportions. For any adjacent pair of doses violating the constraint, apply PAVA: replace both estimates with the sample-size weighted average (n_j*p̃_j + n_{j+1}*p̃_{j+1})/(n_j + n_{j+1}), and repeat pooling until the monotonically increasing restriction holds for all doses. The TSNP design then uses these isotonic estimates p̂_j in stage-1 dose-escalation decisions (escalate if p̂_j ≤ φ−τ, de-escalate if overly toxic, otherwise retain) and as inputs to the stage-2 utility function. Rationale: the isotonic estimates are the constrained MLEs of the DLT probabilities (e.g., the example trial yields p4^ = 1/3).

## Related tools

- **R** (Programming language used to implement PAVA and the TSNP design; R code for simulation and trial implementation is provided in the TSNP repository) — https://github.com/yongzang2020/TSNP

## Evaluation signals

- The final estimates p̂_1, ..., p̂_J satisfy p1 ≤ p2 ≤ ... ≤ pJ (the order constraint holds for all dose levels)
- Each isotonic estimate equals a sample-size weighted average of the raw proportions within its pooled block, so total observed DLT counts are preserved
- If raw proportions already satisfy the order constraint, PAVA leaves them unchanged (estimates equal p̃_j)
- Each p̂_j lies within [0,1] and matches the constrained MLE of p_j under the order constraint
- Escalation decisions derived from p̂_j match the rule: escalate if p̂_j ≤ φ−τ, de-escalate if overly toxic per Fisher's exact test p-value rj, otherwise retain

## Limitations

- The design is appropriate only for trials with binary, quickly observable response outcomes
- Cannot be applied directly when the response outcome requires long follow-up to assess
- Extension to drug-combination trials is future work, not yet done

## Evidence

- [methods] PAVA derives isotonic estimates: "We first use the pool-adjacent-violators algorithm (PAVA) to derive the isotonic estimates"
- [methods] Isotonic estimates are constrained MLEs: "the isotonic estimates pȷ^ are equivalent to the MLE of pj under the order constraint (1)"
