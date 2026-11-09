---
name: isotonic-dlt-rate-estimation
description: Use when a phase I/II dose-finding analysis needs toxicity probabilities
  at each dose that satisfy a monotonic order constraint, and when joint toxicity-efficacy
  probability estimates must be computed in closed form without Bayesian modeling,
  MCMC, or numerical optimization.
when_to_use_negative:
- The response (efficacy) outcome is not binary or requires long follow-up to assess
  — the TSNP design is appropriate only when the response outcome is binary and observable
  shortly after treatment initiation
- A drug-combination trial where the target is an optimal dose combination — extension
  of TSNP to combination trials is explicitly future work
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Implementation language for PAVA isotonic estimation and the closed-form conditional
    MLE computation in the TSNP codebase
  repo: https://github.com/yongzang2020/TSNP
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pst.2075
    title: '<scp>TSNP</scp>: A <scp>two‐stage</scp> nonparametric phase I/<scp>II</scp>
      clinical trial design for immunotherapy'
  source_content_hash: sha256:90c8aaaff384d3e440b953209aa36bca55f5c021acef0f7e1ea85fe3c36c626c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yongzang2020/TSNP
  tools:
  - R
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# isotonic-dlt-rate-estimation

## Summary

Nonparametric estimation of dose-limiting-toxicity (DLT) rates via the pool-adjacent-violators algorithm (PAVA) under a monotonic increasing toxicity constraint, as used in the TSNP phase I/II design for immunotherapy, with closed-form conditional MLEs of joint toxicity-efficacy probabilities derived from the isotonic DLT estimate.

## When to use

Use when a phase I/II dose-finding analysis needs toxicity probabilities at each dose that satisfy a monotonic order constraint, and when joint toxicity-efficacy probability estimates must be computed in closed form without Bayesian modeling, MCMC, or numerical optimization.

## When NOT to use

- The response (efficacy) outcome is not binary or requires long follow-up to assess — the TSNP design is appropriate only when the response outcome is binary and observable shortly after treatment initiation
- A drug-combination trial where the target is an optimal dose combination — extension of TSNP to combination trials is explicitly future work

## Inputs

- Per-dose joint toxicity-efficacy outcome counts x_jkl (k,l = 0,1) for each dose level d_j (e.g., x400 = 2, x401 = 2, x410 = 0, x411 = 2 at dose 4)
- Observed DLT (toxicity) indicator per patient by dose level

## Outputs

- Isotonic DLT rate estimates p_j^ satisfying the monotonic order constraint (1)
- Per-dose joint toxicity-efficacy probability estimates pi_jkl^ written to joint_probability_estimates.csv

## How to apply

Load the joint toxicity-efficacy outcome counts x_jkl (k,l = 0,1) at each dose level d_j. Apply the pool-adjacent-violators algorithm (PAVA) to the observed DLT rates to obtain isotonic estimates p_j^ that satisfy the monotonic increasing order constraint (1); these isotonic estimates are equivalent to the MLEs of p_j under that constraint. Then, for each dose, condition the likelihood on p_j = p_j^ and apply the closed-form conditional MLE relations: pi_j00^ = (1-p_j^) * x_j00/(x_j00+x_j01), pi_j01^ = (1-p_j^) * x_j01/(x_j00+x_j01), pi_j10^ = p_j^ * x_j10/(x_j10+x_j11), pi_j11^ = p_j^ * x_j11/(x_j10+x_j11). Implement in R consistent with the TSNP codebase (github.com/yongzang2020/TSNP); no MCMC or numerical optimization is required. Write the per-dose estimates to joint_probability_estimates.csv; these feed the subsequent utility calculation U(pi_jkl^, omega_kl) for OBD selection.

## Related tools

- **R** (Implementation language for PAVA isotonic estimation and the closed-form conditional MLE computation in the TSNP codebase) — https://github.com/yongzang2020/TSNP

## Evaluation signals

- Verify the isotonicity invariant: estimated DLT rates p_j^ are non-decreasing across dose levels j = 1, ..., J
- Verify the conditional probability invariants: pi_j00^ + pi_j01^ = 1 - p_j^ and pi_j10^ + pi_j11^ = p_j^ for every dose
- Check that PAVA output matches the constrained MLE — the article states the isotonic estimates are equivalent to the MLE of p_j under the order constraint (1)
- Reproduce the worked example: at dose 4 with x400 = 2, x401 = 2, x410 = 0, x411 = 2, the estimates are pi_400^ = 1/3, pi_401^ = 1/3, pi_410^ = 0, pi_411^ = 1/3
- Confirm no MCMC or numerical optimization routines are invoked — the estimation is fully closed-form conditional on p_j^

## Limitations

- Appropriate only for trials with binary, quickly observable response outcomes; cannot be applied directly when the response outcome requires long follow-up time to assess
- Derived for single-agent dose finding; extension to phase I/II drug-combination trials is warranted future work, not yet done
- PAVA estimation is conditioned on the monotonic increasing toxicity constraint (1); it does not impose monotonicity on the efficacy outcomes

## Evidence
