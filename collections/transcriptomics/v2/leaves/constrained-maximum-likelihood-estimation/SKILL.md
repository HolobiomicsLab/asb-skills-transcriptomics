---
name: constrained-maximum-likelihood-estimation
description: Use when binary joint toxicity-efficacy outcome counts (x_jkl) are available
  per dose and closed-form MLEs of π_jkl are needed subject to a monotonicity order
  constraint on toxicity probabilities.
when_to_use_negative:
- The response outcome requires long follow-up time to assess, since the TSNP design
  assumes binary, quickly observable response outcomes
- Toxicity probabilities are not expected to be monotonically increasing with dose,
  invalidating the order constraint (1) on which the PAVA and conditional MLE rely
- A Bayesian parametric joint toxicity-efficacy model with prior-based estimation
  (e.g., via MCMC) is specifically required; this skill is deliberately nonparametric
  and closed-form
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Implements the PAVA isotonic estimation and the closed-form conditional MLE
    computation without MCMC or numerical optimization
  repo: https://github.com/yongzang2020/TSNP
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pst.2075
    title: '<scp>TSNP</scp>: A <scp>two‐stage</scp> nonparametric phase I/<scp>II</scp>
      clinical trial design for immunotherapy'
  source_content_hash: sha256:0cf9be2c28e9048c54fc5320677bb26a88bec353849dc9cdd51aeda17324fda6
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

# constrained-maximum-likelihood-estimation

## Summary

Derives closed-form nonparametric estimates of joint toxicity-efficacy response probabilities (π_jkl) in the TSNP phase I/II design, by conditioning the likelihood on PAVA-isotonic DLT rate estimates under the monotonic increasing toxicity constraint, avoiding MCMC or numerical optimization.

## When to use

Use when binary joint toxicity-efficacy outcome counts (x_jkl) are available per dose and closed-form MLEs of π_jkl are needed subject to a monotonicity order constraint on toxicity probabilities.

## When NOT to use

- The response outcome requires long follow-up time to assess, since the TSNP design assumes binary, quickly observable response outcomes
- Toxicity probabilities are not expected to be monotonically increasing with dose, invalidating the order constraint (1) on which the PAVA and conditional MLE rely
- A Bayesian parametric joint toxicity-efficacy model with prior-based estimation (e.g., via MCMC) is specifically required; this skill is deliberately nonparametric and closed-form

## Inputs

- Per-dose joint toxicity-efficacy outcome counts x_jkl (k,l = 0,1) from patients treated at each dose d_j
- Isotonic DLT rate estimates p_j^ produced by the PAVA algorithm

## Outputs

- Per-dose closed-form estimates of joint response probabilities (π̂_j00, π̂_j01, π̂_j10, π̂_j11) in joint_probability_estimates.csv
- Feed-forward probability estimates used to compute utility values U(π̂_jkl, ω_kl) for OBD selection

## How to apply

First apply the pool-adjacent-violators algorithm (PAVA) to the observed DLT data to obtain isotonic estimates p_j^ that satisfy the monotonic increasing order constraint (1); these are equivalent to the MLEs of p_j under that constraint. Then, for each dose, condition the likelihood on p_j = p_j^ and apply the closed-form conditional MLE relations: π̂_j00 = (1−p_j^)·x_j00/(x_j00+x_j01), π̂_j01 = (1−p_j^)·x_j01/(x_j00+x_j01), π̂_j10 = p_j^·x_j10/(x_j10+x_j11), π̂_j11 = p_j^·x_j11/(x_j10+x_j11). Implement in R (consistent with the TSNP codebase) with no MCMC or numerical optimization. Verify per dose that π̂_j00 + π̂_j01 = 1−p_j^ and π̂_j10 + π̂_j11 = p_j^, confirming equivalence to the MLEs under the order constraint, and write the per-dose estimates to joint_probability_estimates.csv for downstream utility calculation.

## Related tools

- **R** (Implements the PAVA isotonic estimation and the closed-form conditional MLE computation without MCMC or numerical optimization) — https://github.com/yongzang2020/TSNP

## Examples

```
Rscript tsnp_conditional_mle.R --counts x_jkl.csv --isotonic p_hat_pava.csv --out joint_probability_estimates.csv
```

## Evaluation signals

- Verify the row-wise identities π̂_j00 + π̂_j01 = 1−p_j^ and π̂_j10 + π̂_j11 = p_j^ hold for every dose
- Check the isotonic PAVA estimates p_j^ satisfy the monotonic increasing order constraint (1) across dose levels
- Replicate the worked example at dose 4 (x400=2, x401=2, x410=0, x411=2) yielding π̂400 = 1/3, π̂401 = 1/3, π̂410 = 0, π̂411 = 1/3
- Confirm no numerical optimization or MCMC sampling is invoked — estimates are obtained purely in closed form
- Confirm the output joint_probability_estimates.csv contains per-dose probability rows consumable by the utility calculation

## Limitations

- Estimates are valid only under the monotonic increasing constraint for the toxicity outcomes; violation of the dose-toxicity ordering undermines the PAVA step
- Applicable only when the response outcome is binary and observable shortly after treatment initiation, not for long follow-up endpoints
- Extension to drug-combination trials is future work; the estimator applies to single-agent dose-finding only

## Evidence
