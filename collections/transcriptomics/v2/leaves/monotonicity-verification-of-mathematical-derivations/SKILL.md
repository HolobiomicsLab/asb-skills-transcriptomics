---
name: monotonicity-verification-of-mathematical-derivations
description: Use when you need to numerically verify a closed-form mathematical derivation
  (e.g., monotonicity between a likelihood/density objective and a stability measure)
  before relying on it in a pipeline.
when_to_use_negative:
- Input is empirical qPCR data requiring weight optimization (use calcWeight / run_experiment
  instead — this skill validates the derivation, not the weights)
- The stability measure of interest comes from a different distributional assumption
  (e.g., beta distribution), where the CV/SD equivalences derived here need not hold
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Environment used for the parameter sweep over delta, numerical evaluation
    of fZ(z=1), CV and SD-of-log-ratio, monotonicity checks, and plotting
- name: InterOpt
  role: R package whose geom(sd) weighting operationalizes the verified derivation
    (weights optimized on SD of logarithm of expression)
  repo: https://github.com/asalimih/InterOpt
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2023.107945
    title: 'InterOpt: Improved gene expression quantification in qPCR experiments
      using weighted aggregation of reference genes'
  source_content_hash: sha256:d9029cd02b089c3453b6204f96a79c76415f7884419805256163a979e8bb7b4b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/asalimih/InterOpt
  tools:
  - InterOpt
  - R
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# monotonicity-verification-of-mathematical-derivations

## Summary

Numerically reconstructs and validates the paper's theoretical claims that SD and CV are valid reference-gene stability measures: for a Gaussian ratio model Z = X1/X2 the argmax of the ratio density fZ(z=1) coincides with the argmin of the CV, and for a log-normal model it coincides with the argmin of the SD of the log-ratio.

## When to use

Use when you need to numerically verify a closed-form mathematical derivation (e.g., monotonicity between a likelihood/density objective and a stability measure) before relying on it in a pipeline.

## When NOT to use

- Input is empirical qPCR data requiring weight optimization (use calcWeight / run_experiment instead — this skill validates the derivation, not the weights)
- The stability measure of interest comes from a different distributional assumption (e.g., beta distribution), where the CV/SD equivalences derived here need not hold

## Inputs

- Parameter grid over the dispersion parameter delta of the two-gene ratio model
- Gaussian model specification for Z = X1/X2 (mean and delta-dependent variance)
- Log-normal model specification for Z with log-ratio SD as the stability measure

## Outputs

- Table of delta values with fZ(z=1), CV (Gaussian case), and SD of log-ratio (log-normal case)
- Boolean confirmation that argmax fZ(z=1) coincides with argmin of each stability measure
- Plots of fZ(z=1) versus CV and versus SD of log-ratio showing the monotone inverse relationship

## How to apply

In R, define a parameter sweep over the dispersion parameter delta of a two-gene model. For the Gaussian case, compute the ratio density fZ(z=1) for Z = X1/X2 at each delta and the CV of Z, then check that dfZ/ddelta is monotone so that argmax fZ(z=1) equals argmin CV. Repeat under the log-normal model, computing fZ(z=1) and the SD of the log-ratio log(Z), and confirm that maximizing fZ(z=1) is equivalent to minimizing the SD of the log-ratio — this is the theoretical basis for the paper's geom(sd) weighting, which optimizes weights on the SD of the logarithm of expression. Plot fZ(z=1) against each stability measure to display the monotone inverse relationship. The script must print boolean confirmations that the argmax of fZ(z=1) aligns with the argmin of CV (Gaussian) and the argmin of SD of log-ratio (log-normal) in both models.

## Related tools

- **R** (Environment used for the parameter sweep over delta, numerical evaluation of fZ(z=1), CV and SD-of-log-ratio, monotonicity checks, and plotting)
- **InterOpt** (R package whose geom(sd) weighting operationalizes the verified derivation (weights optimized on SD of logarithm of expression)) — https://github.com/asalimih/InterOpt

## Evaluation signals

- Printed boolean is TRUE: argmax of fZ(z=1) equals argmin of CV in the Gaussian two-gene model across the delta sweep
- Printed boolean is TRUE: argmax of fZ(z=1) equals argmin of the SD of log(Z) in the log-normal model
- The derivative dfZ/ddelta over delta is sign-monotone (single sign change), supporting the claimed monotone inverse relationship
- Plots of fZ(z=1) vs. CV and vs. SD of log-ratio show a strictly monotone inverse relationship with no interior extrema inconsistent with the derivation

## Limitations

- The equivalences hold only under the assumed Gaussian and log-normal distributional models; the paper notes that examining other distributions (like beta distribution) and better measures are suggested for further improvements
- The scale-invariant function family could be explored for more stable members in future research, so the verified derivation does not exhaust the space of aggregation functions

## Evidence

- [intro] we first elucidate the theoretical assumptions behind SD and CV as RG expression stability measures.: "we first elucidate the theoretical assumptions behind SD and CV as RG expression stability measures."
- [intro] The paper elucidates the theoretical assumptions behind SD and CV as stability measures and operationalizes them via four novel weighted aggregation methods that optimally minimize the SD or CV of a virtual RG.: [evidence span withheld — anchor status: not_found]
