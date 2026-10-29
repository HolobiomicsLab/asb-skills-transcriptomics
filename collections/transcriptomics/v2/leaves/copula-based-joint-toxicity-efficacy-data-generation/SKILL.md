---
name: copula-based-joint-toxicity-efficacy-data-generation
description: Use when simulating phase I/II trials with correlated binary toxicity
  and efficacy outcomes, or when reproducing Table 3-style comparisons (e.g., Gumbel
  association parameter 0.2) between dose-finding designs like TSNP and U-BOIN.
when_to_use_negative:
- The response outcome requires long follow-up to assess — the article states the
  design 'cannot be applied directly to cases in which the response outcome requires
  a long follow-up time to be assessed'
- Outcomes are not binary and quickly observable — the design is appropriate only
  for trials where 'the response outcome is binary and observable shortly after the
  treatment initiation'
- The trial is a drug-combination study — extending TSNP to combination dosing is
  explicitly listed as future work, not yet available
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3398
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Runs the Gumbel() and crossratio() functions from the TSNP package code to
    generate joint toxicity-efficacy response probabilities for simulation scenarios
  repo: https://github.com/yongzang2020/TSNP
- name: TSNP R package code
  role: Source of the Gumbel() copula generator and the TSNP two-stage simulation
    being fed the copula-generated scenarios
  repo: https://github.com/yongzang2020/TSNP
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pst.2075
    title: '<scp>TSNP</scp>: A <scp>two‐stage</scp> nonparametric phase I/<scp>II</scp>
      clinical trial design for immunotherapy'
  source_content_hash: sha256:a7d333ae359b2c58e03a702f818981c282d8bae60fc716d296c311cf23c5a4e1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yongzang2020/TSNP
  tools:
  - R
  - TSNP R package code
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3398
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# copula-based-joint-toxicity-efficacy-data-generation

## Summary

Generates correlated joint toxicity-efficacy response probabilities for phase I/II simulation scenarios using the Gumbel() function from the TSNP R code, enabling head-to-head comparison of designs such as TSNP vs U-BOIN on identical scenario inputs.

## When to use

Use when simulating phase I/II trials with correlated binary toxicity and efficacy outcomes, or when reproducing Table 3-style comparisons (e.g., Gumbel association parameter 0.2) between dose-finding designs like TSNP and U-BOIN.

## When NOT to use

- The response outcome requires long follow-up to assess — the article states the design 'cannot be applied directly to cases in which the response outcome requires a long follow-up time to be assessed'
- Outcomes are not binary and quickly observable — the design is appropriate only for trials where 'the response outcome is binary and observable shortly after the treatment initiation'
- The trial is a drug-combination study — extending TSNP to combination dosing is explicitly listed as future work, not yet available

## Inputs

- Scenario configuration: number of doses J, target toxicity phi, cut-off tau, admissibility levels cT and cE, stage-1 cohort size m, maximum sample size
- Elicited desirability scores omega_kl (omega_01=100, omega_10=0)
- Gumbel copula association parameter (e.g. 0.2 as in Table 3)
- Marginal toxicity and efficacy probabilities per dose for each scenario

## Outputs

- Joint toxicity-efficacy response probabilities (pi_jkl) per dose per scenario
- Scenario set ready to be fed into TSNP and U-BOIN simulation runs
- Aggregated OBD selection percentages and average number of patients treated at each dose (e.g. written to tsnp_uboin_simulation_results.csv)

## How to apply

Load the TSNP R package code from github.com/yongzang2020/TSNP and define the scenario configuration: number of doses J, target toxicity rate phi, cut-off tau, admissibility levels cT and cE, stage-1 cohort size m, maximum sample size, and elicited desirability scores omega_kl (with omega_01=100 and omega_10=0). Call the Gumbel() function (with the association parameter, e.g. 0.2 as in Table 3) — optionally crossratio() for the cross-ratio association measure — to generate the joint toxicity-efficacy response probabilities pi_jkl for each dose in each scenario. Run these identical scenarios (with matched random seeds) through both the TSNP two-stage simulation and the comparator design (e.g. U-BOIN) so the copula-generated probabilities are the only difference-free input. Correctness is judged by both designs seeing exactly the same marginal toxicity and efficacy probabilities per dose, with correlation induced solely by the copula association parameter.

## Related tools

- **R** (Runs the Gumbel() and crossratio() functions from the TSNP package code to generate joint toxicity-efficacy response probabilities for simulation scenarios) — https://github.com/yongzang2020/TSNP
- **TSNP R package code** (Source of the Gumbel() copula generator and the TSNP two-stage simulation being fed the copula-generated scenarios) — https://github.com/yongzang2020/TSNP

## Examples

```
source("Gumbel.R"); pi_jkl <- Gumbel(p_tox_j, p_eff_j, theta=0.2)  # per-dose joint probabilities, as in Table 3
```

## Evaluation signals

- Marginal toxicity and efficacy probabilities per dose match the intended scenario values; only the joint distribution is altered by the copula
- Association between toxicity and efficacy corresponds to the specified Gumbel association parameter (e.g. 0.2, as for Table 3)
- The same generated probabilities (and random seeds) are passed to both TSNP and U-BOIN, giving a clean head-to-head comparison
- Resulting OBD selection and patient-assignment percentages for TSNP are comparable to U-BOIN's — U-BOIN slightly better in scenarios 1 and 2, TSNP better in the other two
- Aggregated side-by-side table (e.g. tsnp_uboin_simulation_results.csv) reproduces the reported qualitative pattern across all four scenarios

## Limitations

- The Shiny app providing a GUI for the TSNP code is still under development, so scenario setup and copula generation must be done via raw R code
- The Gumbel copula assumes a monotonic increasing toxicity constraint; the underlying nonparametric estimates are valid only under that order constraint
- Only binary toxicity and efficacy outcomes that are quickly observable can be simulated under this design

## Evidence
