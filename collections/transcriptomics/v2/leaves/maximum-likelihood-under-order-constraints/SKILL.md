---
name: maximum-likelihood-under-order-constraints
description: Use when per-dose toxicity (or other monotone dose-response) proportions
  violate the increasing order constraint and you need monotonicity-constrained MLEs,
  e.g. isotonic DLT rate estimates for TSNP dose escalation decisions.
when_to_use_negative:
- When the raw per-dose proportions already satisfy the monotonic increasing constraint
  — no pooling is needed and raw proportions are the constrained MLEs.
- When dose-response need not be monotone (the constraint p1 ≤ p2 ≤ ... ≤ pJ is a
  design assumption of TSNP; violating it structurally invalidates isotonic fitting).
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: R
  role: R code implements PAVA-based isotonic estimation of toxicity rates and the
    surrounding TSNP simulation and trial implementation
  repo: https://github.com/yongzang2020/TSNP
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pst.2075
    title: '<scp>TSNP</scp>: A <scp>two‐stage</scp> nonparametric phase I/<scp>II</scp>
      clinical trial design for immunotherapy'
  source_content_hash: sha256:bf47cadbd96a0753432f42fc804ef46e37c0e165e6cc98bfe249ba581c909c72
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yongzang2020/TSNP
  tools:
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# maximum-likelihood-under-order-constraints

## Summary

Computes order-constrained maximum likelihood estimates of per-dose DLT rates in the TSNP phase I/II design by applying the pool-adjacent-violators algorithm (PAVA) to raw toxicity proportions, yielding isotonic estimates p̂_j that satisfy p1 ≤ p2 ≤ ... ≤ pJ.

## When to use

Use when per-dose toxicity (or other monotone dose-response) proportions violate the increasing order constraint and you need monotonicity-constrained MLEs, e.g. isotonic DLT rate estimates for TSNP dose escalation decisions.

## When NOT to use

- When the raw per-dose proportions already satisfy the monotonic increasing constraint — no pooling is needed and raw proportions are the constrained MLEs.
- When dose-response need not be monotone (the constraint p1 ≤ p2 ≤ ... ≤ pJ is a design assumption of TSNP; violating it structurally invalidates isotonic fitting).

## Inputs

- per-dose toxicity outcome data: number of patients n_j per dose and DLT counts (dose levels j = 1..J)
- raw per-dose DLT proportions p̃_j

## Outputs

- isotonic DLT rate estimates p̂_j for all dose levels (e.g. written to isotonic_dlt_estimates.csv)
- pooled block assignments from PAVA indicating which adjacent doses were merged

## How to apply

Load per-dose counts (number of patients n_j and DLT counts per dose) and compute raw proportions p̃_j. Check whether the order constraint p1 ≤ p2 ≤ ... ≤ pJ holds; if so, the isotonic estimates equal the raw proportions. For any adjacent pair violating the constraint, apply PAVA: replace both estimates with the sample-size weighted average (n_j*p̃_j + n_{j+1}*p̃_{j+1})/(n_j + n_{j+1}), and repeat pooling until the sequence is monotonically increasing across all doses. The result is statistically justified because the isotonic estimates p̂_j are equivalent to the MLE of p_j under the order constraint; these p̂_j then feed TSNP dose-escalation decisions (e.g. compare p̂_j against φ−τ to escalate, de-escalate, or retain).

## Related tools

- **R** (R code implements PAVA-based isotonic estimation of toxicity rates and the surrounding TSNP simulation and trial implementation) — https://github.com/yongzang2020/TSNP

## Evaluation signals

- The final isotonic estimates p̂_j satisfy p1 ≤ p2 ≤ ... ≤ pJ for all J doses.
- Each pooled estimate equals the sample-size weighted average of the raw proportions within its pooled block (e.g. (n_j*p̃_j + n_{j+1}*p̃_{j+1})/(n_j + n_{j+1}) for a merged adjacent pair).
- If no adjacent violators exist, the output equals the raw per-dose DLT proportions p̃_j unchanged.
- Within each pooled block, total expected DLTs (sum of n_j * p̂_j) equals the observed total DLT count, and block-level rates equal the block pooled proportion.
- Illustrative check: an example trial yields p̂_4 = 1/3 as an isotonic estimate consistent with pooling of adjacent violators.

## Limitations

- PAVA forces monotonicity by averaging, so an apparent violation at a low dose can pull the estimates of higher doses downward; small per-dose sample sizes m amplify this pooling effect.
- The isotonic estimate is a constrained (shrunken) MLE — it is not the unconstrained per-dose proportion, so downstream thresholds (e.g. φ−τ for escalation/de-escalation) operate on pooled, not raw, values.

## Evidence
