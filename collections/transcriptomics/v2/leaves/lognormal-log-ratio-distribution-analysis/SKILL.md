---
name: lognormal-log-ratio-distribution-analysis
description: Use when validating, teaching, or extending the theoretical assumptions
  behind SD and CV as reference-gene stability measures, or when checking numerically
  that argmax fZ(z=1) coincides with argmin of the respective stability measure under
  Gaussian or log-normal ratio models.
when_to_use_negative:
- Input is empirical qPCR CT data requiring actual aggregation/normalization — use
  InterOpt's calcWeight/run_experiment instead of re-deriving the theory
- You need comparative stability measures such as NormFinder or geNorm output rather
  than the theoretical SD/CV assumptions
- The expression distribution is known or suspected to deviate from Gaussian/log-normal
  — the article itself flags that other distributions (like beta distribution) and
  better measures are suggested for further improvement
edam_operation: http://edamontology.org/operation_2269
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Implementation language for the parameter sweeps, numerical monotonicity checks,
    and validation plots of the Gaussian and log-normal derivations
  repo: https://github.com/asalimih/InterOpt
- name: InterOpt
  role: 'R package that operationalizes the derivations: its geom_sd weights optimize
    the SD of the logarithm of expression, the log-normal stability measure'
  repo: https://github.com/asalimih/InterOpt
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2023.107945
    title: 'InterOpt: Improved gene expression quantification in qPCR experiments
      using weighted aggregation of reference genes'
  source_content_hash: sha256:5a9ab001500a91d1af309a3b0401d960754b3544b04efbbfc5b4395136c86357
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/asalimih/InterOpt
  tools:
  - InterOpt
  - R
  edam_operation: http://edamontology.org/operation_2269
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# lognormal-log-ratio-distribution-analysis

## Summary

A theoretical derivation and numerical-validation skill that justifies SD and CV as qPCR reference-gene stability measures: under a Gaussian two-gene model, minimizing the CV of the ratio Z = X1/X2 is equivalent to maximizing the ratio density fZ(z=1), and under a log-normal model, minimizing the SD of the log-ratio log(Z) achieves the same optimum.

## When to use

Use when validating, teaching, or extending the theoretical assumptions behind SD and CV as reference-gene stability measures, or when checking numerically that argmax fZ(z=1) coincides with argmin of the respective stability measure under Gaussian or log-normal ratio models.

## When NOT to use

- Input is empirical qPCR CT data requiring actual aggregation/normalization — use InterOpt's calcWeight/run_experiment instead of re-deriving the theory
- You need comparative stability measures such as NormFinder or geNorm output rather than the theoretical SD/CV assumptions
- The expression distribution is known or suspected to deviate from Gaussian/log-normal — the article itself flags that other distributions (like beta distribution) and better measures are suggested for further improvement

## Inputs

- Gaussian two-gene model parameter sweep over dispersion parameter delta for ratio Z = X1/X2
- Log-normal two-gene model parameter sweep over dispersion parameter delta
- Raw CT values of candidate reference genes (for the operational geom(sd) counterpart)
- R environment with numerical differentiation/optimization primitives

## Outputs

- Vector of fZ(z=1) values across the delta sweep for both Gaussian and log-normal models
- Vector of CV values (Gaussian case) and SD of log-ratio log(Z) values (log-normal case)
- Boolean confirmation that argmax fZ(z=1) aligns with argmin of the respective stability measure in both models
- Plots of fZ(z=1) against CV and against SD of log-ratio showing the monotone inverse relationship

## How to apply

In R, define a parameter sweep over the dispersion parameter delta of a two-gene model. First, under the Gaussian model, compute the ratio density fZ(z=1) at Z = X1/X2 for each delta and compute the CV of each delta; numerically verify the monotonicity claim that argmax fZ(z=1) coincides with argmin CV (d fZ/d delta monotone). Second, repeat the sweep under a log-normal model, computing fZ(z=1) and the SD of the log-ratio log(Z), verifying numerically that maximizing fZ(z=1) is equivalent to minimizing the SD of the log-ratio. This equivalence is the rationale for InterOpt's geom(sd) weighting, whose weights optimized on the SD of logarithm of expression can be calculated solely from raw CT values. Plot fZ(z=1) against both stability measures (CV for Gaussian, SD of log-ratio for log-normal) to exhibit the monotone inverse relationship; this connects to the scale-invariant function family explaining why arithmetic and geometric means aggregate reference genes.

## Related tools

- **R** (Implementation language for the parameter sweeps, numerical monotonicity checks, and validation plots of the Gaussian and log-normal derivations) — https://github.com/asalimih/InterOpt
- **InterOpt** (R package that operationalizes the derivations: its geom_sd weights optimize the SD of the logarithm of expression, the log-normal stability measure) — https://github.com/asalimih/InterOpt

## Examples

```
w = calcWeight(x, ctVal = TRUE, weight_method = 'geom_sd_plus')
```

## Evaluation signals

- The script prints a boolean confirmation that argmax of fZ(z=1) coincides with argmin CV in the Gaussian two-gene model
- The script prints a boolean confirmation that argmax of fZ(z=1) coincides with argmin of the SD of the log-ratio in the log-normal model
- d fZ/d delta is numerically monotone across the delta sweep (monotonicity claim holds)
- Plots show a monotone inverse relationship between fZ(z=1) and each stability measure (CV, SD of log-ratio)
- Weights derived from the log-normal (SD-of-log) criterion can be recomputed solely from raw CT values, consistent with the geom_sd_plus weight method

## Limitations

- The equivalence between maximizing fZ(z=1) and minimizing SD/CV holds only under the assumed (Gaussian or log-normal) ratio distributions; the article suggests examining other distributions (like beta distribution) and providing better measures for further improvements
- The scale-invariant function family underlying the aggregation means could be explored for more stable members, so the derived measures are not claimed to be globally optimal
- Aggregating more than 5 RGs by geom(sd) does not have expected benefits compared to the regular geometric mean, bounding the practical payoff of the SD-based derivation

## Evidence

- [intro] Elucidate the theoretical assumptions behind SD and CV as reference gene expression stability measures: "we first elucidate the theoretical assumptions behind SD and CV as RG expression stability measures."
- [intro] Weights from SD of logarithm computable from raw CT: [evidence span withheld — anchor status: not_found]
- [intro] Scale-invariant family justifying mean functions: "by introducing the family of scale-invariant functions, we explain the reason arithmetic and geometric mean functions can be used for the aggregation of multiple RGs."
- [discussion] Distribution-assumption limitation: "examining other distributions (like beta distribution) and providing better measures are suggested for further improvements"
- [readme] Operational geom_sd_plus weight calculation: "w = calcWeight(x, ctVal = TRUE, weight_method = 'geom_sd_plus')"
