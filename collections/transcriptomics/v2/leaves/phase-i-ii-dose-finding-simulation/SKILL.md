---
name: phase-i-ii-dose-finding-simulation
description: Use when simulating a phase I/II trial with binary toxicity and efficacy
  outcomes to identify an optimal biological dose (OBD) for immunotherapy, or when
  benchmarking dose-finding designs on OBD selection and patient-allocation percentages.
when_to_use_negative:
- The response outcome requires long follow-up to assess — the design is only appropriate
  for binary, quickly observable response outcomes
- The trial targets an MTD-only phase I objective rather than joint toxicity-efficacy
  OBD selection for immunotherapy
- The goal is dose- combination finding — extension of TSNP to drug-combination trials
  is future work, not yet implemented
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3305
tools:
- name: R
  role: Implements the TSNP two-stage simulation (PAVA isotonic estimation, Fisher's
    exact test admissibility screens, utility-based OBD selection) and the comparator
    design simulations
  repo: https://github.com/yongzang2020/TSNP
- name: crossratio()
  role: R function that generates joint toxicity-efficacy response probabilities for
    each scenario in the simulation tables
  repo: https://github.com/yongzang2020/TSNP
- name: Gumbel()
  role: R function that generates correlated joint toxicity-efficacy response probabilities
    (e.g., for the copula-based Table 3 scenarios)
  repo: https://github.com/yongzang2020/TSNP
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_002
  source_papers:
  - doi: 10.1002/pst.2075
    title: '<scp>TSNP</scp>: A <scp>two‐stage</scp> nonparametric phase I/<scp>II</scp>
      clinical trial design for immunotherapy'
  source_content_hash: sha256:c6c1ed3fa670ecfb04102463c0a32134bab56e2bc41a621aea2a0963fef453c4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yongzang2020/TSNP
  tools:
  - Gumbel()
  - R
  - crossratio()
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3305
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# phase-i-ii-dose-finding-simulation

## Summary

Reproduce and evaluate the TSNP two-stage nonparametric phase I/II dose-finding design for immunotherapy via Monte Carlo simulation in R, comparing OBD selection percentages against Bayesian parametric comparators (Thall-Cook P1, Yin et al. P2) and U-BOIN.

## When to use

Use when simulating a phase I/II trial with binary toxicity and efficacy outcomes to identify an optimal biological dose (OBD) for immunotherapy, or when benchmarking dose-finding designs on OBD selection and patient-allocation percentages.

## When NOT to use

- The response outcome requires long follow-up to assess — the design is only appropriate for binary, quickly observable response outcomes
- The trial targets an MTD-only phase I objective rather than joint toxicity-efficacy OBD selection for immunotherapy
- The goal is dose- combination finding — extension of TSNP to drug-combination trials is future work, not yet implemented

## Inputs

- Scenario definitions with true joint toxicity-efficacy response probabilities (generated via crossratio() or Gumbel() with association parameter, e.g., 0.2)
- Design parameters: number of doses J, target toxicity rate phi, cutoff tau, admissibility levels cT and cE, stage-1 cohort size m, maximum sample size N
- Elicited desirability scores omega_kl (omega01=100, omega10=0, elicited omega00 and omega11)
- Monte Carlo replication settings and shared random seeds for comparator runs

## Outputs

- Per-scenario OBD selection percentages for TSNP and each comparator design
- No-dose (early termination) rates per scenario
- Average percentage of patients treated at each dose level
- Aggregated results tables (e.g., obd_selection_percentages.csv, tsnp_uboin_simulation_results.csv)

## How to apply

Load the TSNP R code from github.com/yongzang2020/TSNP and configure J dose levels, target toxicity rate phi (e.g., 0.3), cutoff tau, admissibility levels cT and cE, stage-1 cohort size m, and maximum sample size N (40, 50, or 60). Generate true joint toxicity-efficacy response probabilities per scenario using the crossratio() or Gumbel() functions, and fix desirability scores omega01=100, omega10=0 with elicited omega00 and omega11 so the utility U(pi_jkl, omega_kl) is the probability-weighted sum over joint outcomes. Run stage 1: PAVA isotonic toxicity estimates guide escalation (if isotonic estimate <= phi-tau), de-escalation, or retention, with Fisher's exact test p-values rj < cT excluding overly toxic doses from the admissible set. Run stage 2: assign each cohort to the admissible dose maximizing the utility estimate until the maximum sample size is reached, then recommend the highest-utility dose as the final OBD. Run comparators (P1 Thall-Cook, P2 Yin et al., U-BOIN) on identical scenarios and seeds, and aggregate OBD selection percentages, no-dose rates, and average percentage of patients treated at each dose over Monte Carlo replicates.

## Related tools

- **R** (Implements the TSNP two-stage simulation (PAVA isotonic estimation, Fisher's exact test admissibility screens, utility-based OBD selection) and the comparator design simulations) — https://github.com/yongzang2020/TSNP
- **crossratio()** (R function that generates joint toxicity-efficacy response probabilities for each scenario in the simulation tables) — https://github.com/yongzang2020/TSNP
- **Gumbel()** (R function that generates correlated joint toxicity-efficacy response probabilities (e.g., for the copula-based Table 3 scenarios)) — https://github.com/yongzang2020/TSNP

## Examples

```
Rscript tsnp_simulation.R --phi 0.3 --N 60 --scenario 5 --generator crossratio --out obd_selection_percentages.csv
```

## Evaluation signals

- Reproduced OBD selection percentages for TSNP, P1, and P2 match the values in Tables 1-2 within Monte Carlo error (e.g., scenario 1 with phi=0.3: TSNP 64.6%, P1 37.4%, P2 60.0%)
- TSNP-vs-U-BOIN results are comparable: U-BOIN slightly better in scenarios 1-2, TSNP better in the other two scenarios of the Gumbel copula comparison
- Sample-size sensitivity check: scenario 5 correct OBD selection is 49.0% at N=40, 52.6% at N=50, 58.6% at N=60, with N=60 reaching at least 50% across scenarios
- Empty admissible set cases correctly trigger early termination and a no-dose recommendation
- Isotonic toxicity estimates satisfy the monotonicity (order) constraint across doses at every interim look

## Limitations

- Cannot be applied directly when the response outcome requires a long follow-up time to be assessed; limited to binary, quickly observable responses
- The easy-to-use Shiny app with a graphical user interface is still under development
- Extension to drug-combination trials is future work and not yet available

## Evidence

- [results] TSNP design achieved the highest OBD selection percentage of 64.6%, versus 37.4% for P1 (Thall et al.) and 60.0% for P2 (Yin et al.): [evidence span withheld — anchor status: not_found]
