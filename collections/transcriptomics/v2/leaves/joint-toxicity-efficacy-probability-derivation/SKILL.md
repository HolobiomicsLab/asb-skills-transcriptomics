---
name: joint-toxicity-efficacy-probability-derivation
description: Use when dose-finding trial data contain per-dose bivariate toxicity-efficacy
  outcome counts (x_jkl) and monotone-constrained joint response probability estimates
  are needed to feed a utility-based OBD selection.
when_to_use_negative:
- The response outcome requires long follow-up time to be assessed — the TSNP design
  cannot be applied directly in this case
- The response outcome is not binary and observable shortly after treatment initiation
- The trial is a drug-combination study — extending TSNP to identify the optimal dose
  combination is future work, not yet done
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Implements the conditional closed-form MLE computation and the PAVA isotonic
    estimates in the TSNP R codebase
  repo: https://github.com/yongzang2020/TSNP
- name: crossratio()
  role: R function that generates the joint toxicity-efficacy response probabilities
    for simulation scenario configuration
  repo: https://github.com/yongzang2020/TSNP
- name: Gumbel()
  role: R function that generates the joint toxicity-efficacy response probabilities
    for simulation scenario configuration
  repo: https://github.com/yongzang2020/TSNP
provenance:
  source_task_ids:
  - task_007
  - task_002
  source_papers:
  - doi: 10.1002/pst.2075
    title: '<scp>TSNP</scp>: A <scp>two‐stage</scp> nonparametric phase I/<scp>II</scp>
      clinical trial design for immunotherapy'
  source_content_hash: sha256:35a0a40e25f55dcc5a61706ebfd6e7a6a0796654b1bbeb1b7fd5cce0f496051e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yongzang2020/TSNP
  tools:
  - Gumbel()
  - R
  - crossratio()
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# joint-toxicity-efficacy-probability-derivation

## Summary

Derives closed-form nonparametric estimates of the joint toxicity-efficacy response probabilities (π_j00, π_j01, π_j10, π_j11) at each dose in the TSNP phase I/II design, by conditioning the likelihood on the isotonic (PAVA) DLT rate estimate — no MCMC or numerical optimization required.

## When to use

Use when dose-finding trial data contain per-dose bivariate toxicity-efficacy outcome counts (x_jkl) and monotone-constrained joint response probability estimates are needed to feed a utility-based OBD selection.

## When NOT to use

- The response outcome requires long follow-up time to be assessed — the TSNP design cannot be applied directly in this case
- The response outcome is not binary and observable shortly after treatment initiation
- The trial is a drug-combination study — extending TSNP to identify the optimal dose combination is future work, not yet done

## Inputs

- Per-dose joint toxicity-efficacy outcome counts x_jkl (k,l = 0,1) from enrolled patients
- Isotonic DLT rate estimates p_j^ obtained via the pool-adjacent-violators algorithm (PAVA) under the monotonic increasing toxicity constraint (1)

## Outputs

- Closed-form estimates π̂_j00, π̂_j01, π̂_j10, π̂_j11 of the joint toxicity-efficacy response probabilities at each dose (e.g., written to joint_probability_estimates.csv)
- Per-dose probability estimates feeding the utility calculation U(π̂_jkl, ω_kl) for OBD selection

## How to apply

Load the observed joint toxicity-efficacy outcome counts x_jkl (k,l = 0,1) at each dose d_j and the isotonic DLT rate estimates p_j^ produced by the pool-adjacent-violators algorithm (PAVA), which are equivalent to the MLE of p_j under the monotonicity order constraint (1). For each dose, condition the likelihood on p_j = p_j^ and apply the closed-form conditional MLE relations: π̂_j00 = (1−p_j^)·x_j00/(x_j00+x_j01), π̂_j01 = (1−p_j^)·x_j01/(x_j00+x_j01), π̂_j10 = p_j^·x_j10/(x_j10+x_j11), π̂_j11 = p_j^·x_j11/(x_j10+x_j11). Implement in R (consistent with the TSNP codebase at github.com/yongzang2020/TSNP), avoiding Bayesian parametric modeling. The resulting per-dose probability estimates feed the utility calculation U(π̂_jkl, ω_kl) used in stage 2 of the TSNP design to identify the optimal biological dose (OBD).

## Related tools

- **R** (Implements the conditional closed-form MLE computation and the PAVA isotonic estimates in the TSNP R codebase) — https://github.com/yongzang2020/TSNP
- **crossratio()** (R function that generates the joint toxicity-efficacy response probabilities for simulation scenario configuration) — https://github.com/yongzang2020/TSNP
- **Gumbel()** (R function that generates the joint toxicity-efficacy response probabilities for simulation scenario configuration) — https://github.com/yongzang2020/TSNP

## Examples

```
p_hat <- 0.5; pi_00 <- (1-p_hat)*x00/(x00+x01); pi_01 <- (1-p_hat)*x01/(x00+x01); pi_10 <- p_hat*x10/(x10+x11); pi_11 <- p_hat*x11/(x10+x11)
```

## Evaluation signals

- Verify the constraint π̂_j00 + π̂_j01 = 1−p_j^ holds for every dose j
- Verify the constraint π̂_j10 + π̂_j11 = p_j^ holds for every dose j
- Confirm the estimates equal the MLEs under the monotonicity order constraint (1), since the isotonic PAVA estimates are equivalent to the constrained MLE of p_j
- Check a worked example: at dose 4 with x400 = 2, x401 = 2, x410 = 0, x411 = 2, the estimates must be π̂400 = 1/3, π̂401 = 1/3, π̂410 = 0, π̂411 = 1/3
- Confirm no MCMC or numerical optimization was used — computation must be closed-form

## Limitations

- Limited to trials with binary, quickly observable response outcomes
- Cannot be applied directly when the response outcome requires long follow-up time to assess
- Extension to phase I/II drug-combination trials is future work and not yet available

## Evidence
