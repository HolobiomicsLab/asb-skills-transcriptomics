---
name: clinical-trial-binary-outcome-count-analysis
description: Use when phase I/II trial data contain per-dose binary toxicity (DLT)
  and efficacy outcome counts (x_jkl) plus isotonic DLT rate estimates from PAVA,
  and closed-form joint response probability estimates are needed to feed a utility-based
  OBD selection.
when_to_use_negative:
- The response outcome requires long follow-up to assess — the article states the
  design 'cannot be applied directly to cases in which the response outcome requires
  a long follow-up time to be assessed'
- The efficacy outcome is not binary and quickly observable after treatment initiation
- Drug-combination trials — extending TSNP to identify optimal dose combinations is
  explicitly future work not yet done
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Language for implementing the closed-form conditional MLE computation and
    the TSNP simulation/trial pipeline
  repo: https://github.com/yongzang2020/TSNP
- name: crossratio()
  role: R function that generates joint toxicity-efficacy response probabilities for
    simulation of the TSNP design
  repo: https://github.com/yongzang2020/TSNP
- name: Gumbel()
  role: R function that generates joint toxicity-efficacy response probabilities for
    simulation of the TSNP design
  repo: https://github.com/yongzang2020/TSNP
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pst.2075
    title: '<scp>TSNP</scp>: A <scp>two‐stage</scp> nonparametric phase I/<scp>II</scp>
      clinical trial design for immunotherapy'
  source_content_hash: sha256:4ab1a4522764519b43a11ebcbee22a2d4f4fa9be1f3ad3b5f16107d1311e053f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yongzang2020/TSNP
  tools:
  - Gumbel()
  - R
  - crossratio()
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# clinical-trial-binary-outcome-count-analysis

## Summary

Computes closed-form estimates of the joint toxicity-efficacy response probabilities (π_jkl for k,l = 0,1) at each dose of the TSNP phase I/II design, by conditioning the likelihood on the PAVA isotonic DLT rate estimate — no MCMC or numerical optimization required.

## When to use

Use when phase I/II trial data contain per-dose binary toxicity (DLT) and efficacy outcome counts (x_jkl) plus isotonic DLT rate estimates from PAVA, and closed-form joint response probability estimates are needed to feed a utility-based OBD selection.

## When NOT to use

- The response outcome requires long follow-up to assess — the article states the design 'cannot be applied directly to cases in which the response outcome requires a long follow-up time to be assessed'
- The efficacy outcome is not binary and quickly observable after treatment initiation
- Drug-combination trials — extending TSNP to identify optimal dose combinations is explicitly future work not yet done

## Inputs

- Per-dose joint outcome counts x_jkl (k,l = 0,1: no-toxicity/no-efficacy, no-toxicity/efficacy, toxicity/no-efficacy, toxicity/efficacy) for each dose d_j
- Isotonic DLT rate estimates p_j^ from the pool-adjacent-violators algorithm (PAVA) under the monotonic increasing toxicity constraint

## Outputs

- Per-dose closed-form joint probability estimates π̂_jkl (π̂_j00, π̂_j01, π̂_j10, π̂_j11)
- joint_probability_estimates.csv with per-dose estimates feeding the utility calculation U(π̂_jkl, ω_kl) for OBD selection

## How to apply

Load the observed joint toxicity-efficacy outcome counts (x_jkl for k,l = 0,1 at each dose d_j) and the isotonic DLT rate estimates p_j^ produced by the pool-adjacent-violators algorithm (PAVA), which are equivalent to the MLE of p_j under the monotonic order constraint (1). For each dose, condition the likelihood on p_j = p_j^ and apply the closed-form conditional MLE relations: π̂_j00 = (1−p_j^)·x_j00/(x_j00+x_j01), π̂_j01 = (1−p_j^)·x_j01/(x_j00+x_j01), π̂_j10 = p_j^·x_j10/(x_j10+x_j11), π̂_j11 = p_j^·x_j11/(x_j10+x_j11). Implement the computation in R (consistent with the TSNP R codebase at github.com/yongzang2020/TSNP), avoiding MCMC or numerical optimization — the article notes crossratio() and Gumbel() generate joint toxicity-efficacy response probabilities for simulations. Verify the marginal constraints π̂_j00 + π̂_j01 = 1−p_j^ and π̂_j10 + π̂_j11 = p_j^ for every dose, then write the per-dose estimates to joint_probability_estimates.csv for downstream utility calculation U(π̂_jkl, ω_kl).

## Related tools

- **R** (Language for implementing the closed-form conditional MLE computation and the TSNP simulation/trial pipeline) — https://github.com/yongzang2020/TSNP
- **crossratio()** (R function that generates joint toxicity-efficacy response probabilities for simulation of the TSNP design) — https://github.com/yongzang2020/TSNP
- **Gumbel()** (R function that generates joint toxicity-efficacy response probabilities for simulation of the TSNP design) — https://github.com/yongzang2020/TSNP

## Examples

```
p_hat <- pava(dlt_counts); pi_11 <- p_hat * x11 / (x10 + x11)  # conditional MLE given isotonic p_j^
```

## Evaluation signals

- Marginal constraints hold for every dose: π̂_j00 + π̂_j01 = 1−p_j^ and π̂_j10 + π̂_j11 = p_j^
- Estimates are equivalent to the MLEs under the monotonicity order constraint (1) on toxicity
- Worked check: at dose 4 with x400 = 2, x401 = 2, x410 = 0, x411 = 2, expect π̂400 = 1/3, π̂401 = 1/3, π̂410 = 0, π̂411 = 1/3
- No MCMC or numerical optimization is used — estimates must be computable in closed form from counts alone
- Output file joint_probability_estimates.csv contains exactly four probability estimates per dose, all in [0,1]

## Limitations

- Limited to trials with binary, quickly observable response outcomes
- Cannot be applied directly when the response outcome requires long follow-up time to assess
- Extension to drug-combination trials is future work, not yet implemented

## Evidence

- [methods] PAVA isotonic estimates: "We first use the pool-adjacent-violators algorithm (PAVA) to derive the isotonic estimates pȷ^ satisfying the order constraint (1)."
- [methods] MLE equivalence: "the isotonic estimates pȷ^ are equivalent to the MLE of pj under the order constraint (1)"
- [results] R probability generators: "crossratio () and Gumbel() generate the joint toxicity-efficacy response probabilities"
- [readme] R code availability: "R code for simulation and trial implementation of the TSNP design"
