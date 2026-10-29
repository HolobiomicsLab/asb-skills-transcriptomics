---
name: isotonic-regression-pava
description: Use when per-dose toxicity (DLT) proportions from a dose-escalation trial
  violate the order constraint p1 ≤ p2 ≤ ... ≤ pJ and monotonicity-constrained dose-toxicity
  estimates are needed for dose-escalation/de-escalation decisions.
when_to_use_negative:
- The toxicity-efficacy outcomes require long follow-up to assess — the TSNP design
  (which consumes these isotonic estimates) cannot be applied directly in that case.
- Dose-toxicity data are already monotone and no pooling is needed beyond copying
  the raw proportions (the PAVA loop is a no-op).
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Implementation language for PAVA isotonic regression and the TSNP simulation/trial
    code that consumes the isotonic DLT estimates
  repo: https://github.com/yongzang2020/TSNP
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/pst.2075
    title: '<scp>TSNP</scp>: A <scp>two‐stage</scp> nonparametric phase I/<scp>II</scp>
      clinical trial design for immunotherapy'
  source_content_hash: sha256:39534990519cdb8f8b2687029b5b99ce6966b9b93bec60120d4670eb557af9f6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yongzang2020/TSNP
  tools:
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# isotonic-regression-pava

## Summary

Applies the pool-adjacent-violators algorithm (PAVA) to per-dose DLT proportions so that toxicity rate estimates satisfy the monotonic increasing order constraint across dose levels in the TSNP phase I/II design.

## When to use

Use when per-dose toxicity (DLT) proportions from a dose-escalation trial violate the order constraint p1 ≤ p2 ≤ ... ≤ pJ and monotonicity-constrained dose-toxicity estimates are needed for dose-escalation/de-escalation decisions.

## When NOT to use

- The toxicity-efficacy outcomes require long follow-up to assess — the TSNP design (which consumes these isotonic estimates) cannot be applied directly in that case.
- Dose-toxicity data are already monotone and no pooling is needed beyond copying the raw proportions (the PAVA loop is a no-op).

## Inputs

- Per-dose toxicity outcome data: number of patients n_j per dose level and DLT counts
- Raw per-dose DLT proportions p̃_j

## Outputs

- Isotonic DLT rate estimates p̂_j for all dose levels (monotonic increasing), e.g. written to isotonic_dlt_estimates.csv

## How to apply

Load the per-dose toxicity data (number of patients n_j per dose and DLT counts) and compute the raw per-dose DLT proportions p̃_j. Check whether the raw proportions satisfy the order constraint p1 ≤ p2 ≤ ... ≤ pJ; if so, set the isotonic estimates equal to the raw proportions. For any adjacent pair of doses violating the constraint, apply the pool-adjacent-violators algorithm: replace both estimates with the sample-size weighted average (n_j*p̃_j + n_{j+1}*p̃_{j+1})/(n_j + n_{j+1}), repeating the pooling until the monotonically increasing restriction holds for all doses. The resulting isotonic estimates p̂_j are equivalent to the MLE of p_j under the order constraint, which is the key justification for using PAVA in the TSNP design. Use p̂_j in the stage 1 escalation rule (escalate if p̂_j ≤ φ−τ, de-escalate if overly toxic, otherwise retain) and write the final estimates to isotonic_dlt_estimates.csv.

## Related tools

- **R** (Implementation language for PAVA isotonic regression and the TSNP simulation/trial code that consumes the isotonic DLT estimates) — https://github.com/yongzang2020/TSNP

## Examples

```
Rscript -e 'p <- c(0,1/3,1/6,1/3); p.hat <- isotone::gpava(seq_along(p), p)$x; write.csv(p.hat, "isotonic_dlt_estimates.csv")'
```

## Evaluation signals

- The output estimates p̂_j satisfy the order constraint p̂_1 ≤ p̂_2 ≤ ... ≤ p̂_J for all dose levels.
- When the raw proportions already satisfy the order constraint, the isotonic estimates equal the raw per-dose DLT proportions exactly.
- Each pooled value equals the sample-size weighted average (n_j*p̃_j + n_{j+1}*p̃_{j+1})/(n_j + n_{j+1}) of the merged adjacent doses.
- The isotonic estimates match the constrained MLE of p_j (PAVA-equivalence check), e.g. the example trial obtains p̂_4 = 1/3.
- The output file isotonic_dlt_estimates.csv contains one estimate per dose level j = 1, ..., J with no missing values.

## Limitations

- PAVA imposes only a monotonic increasing toxicity constraint; it does not model joint toxicity-efficacy probabilities by itself — those closed-form estimates are derived separately in the TSNP stage 2.
- Estimates are limited to the tried dose levels and depend on accumulated per-dose sample sizes; small cohorts produce noisy raw proportions that drive excessive pooling.

## Evidence
