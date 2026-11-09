---
name: dose-escalation-decision-rule-simulation
description: Use when evaluating or reproducing the operating characteristics (e.g.,
  early-termination probability, P1/P2 dose-selection percentages) of the TSNP phase
  I/II dose-escalation design under a specified toxicity/efficacy scenario.
when_to_use_negative:
- The response outcome requires long follow-up to assess — the TSNP design cannot
  be applied directly to such cases
- The trial involves drug combinations — extending TSNP to drug-combination trials
  is future work, not yet done
- The goal is model-based Bayesian parametric dose-finding rather than the nonparametric
  cumulative-cohort decision rules of TSNP
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Language used for the TSNP simulation and trial-implementation scripts; crossratio()
    and Gumbel() generate the joint toxicity-efficacy response probabilities
  repo: https://github.com/yongzang2020/TSNP
- name: Shiny app
  role: Planned graphical user interface for applying the design; still under development,
    not currently usable for this skill
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pst.2075
    title: '<scp>TSNP</scp>: A <scp>two‐stage</scp> nonparametric phase I/<scp>II</scp>
      clinical trial design for immunotherapy'
  source_content_hash: sha256:c8ee5da1bf3520c2143b96c51ab08c9eb09f162a2cf98b272e00bc20d4749f7b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yongzang2020/TSNP
  tools:
  - R
  - Shiny app
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# dose-escalation-decision-rule-simulation

## Summary

Simulates the TSNP two-stage nonparametric phase I/II design's dose-escalation and early-termination decision rules in R to estimate operating characteristics such as early-termination probability and dose-selection percentages under null and alternative scenarios.

## When to use

Use when evaluating or reproducing the operating characteristics (e.g., early-termination probability, P1/P2 dose-selection percentages) of the TSNP phase I/II dose-escalation design under a specified toxicity/efficacy scenario.

## When NOT to use

- The response outcome requires long follow-up to assess — the TSNP design cannot be applied directly to such cases
- The trial involves drug combinations — extending TSNP to drug-combination trials is future work, not yet done
- The goal is model-based Bayesian parametric dose-finding rather than the nonparametric cumulative-cohort decision rules of TSNP

## Inputs

- R simulation and trial-implementation scripts from the TSNP repository
- Scenario specification: true DLT rates per dose, true TR (efficacy) rates per dose
- Design parameters: target toxicity probability phi (e.g., 0.3 or 0.2), cutoff tau, toxicity admissibility nominal level cT, efficacy admissibility threshold cE, cohort size m, maximum sample size
- Number of Monte Carlo simulation replications

## Outputs

- Early-termination probability per design (e.g., TSNP 96.4% vs P1 93.3% in the null scenario)
- Dose-selection percentage tables (P1/P2) per design and scenario
- Aggregated operating-characteristics CSV (e.g., null_scenario_early_termination.csv)

## How to apply

Obtain the TSNP simulation and trial-implementation R scripts from github.com/yongzang2020/TSNP and configure the target scenario by setting true DLT rates and true TR rates (e.g., in the null case every dose exceeds the highest acceptable toxicity rate, with phi changed from 0.3 to 0.2). Simulate trial replications in which stage 1 uses the cumulative cohort design with pool-adjacent-violators (PAVA) isotonic toxicity estimates, escalating or de-escalating against the cutoff phi−τ, and Fisher's exact toxicity admissibility tests at nominal level cT. In each replication, apply the early-termination rule that ends the trial and recommends no dose whenever the admissible set becomes empty; otherwise continue into stage 2 utility-based OBD selection until the maximum sample size is reached. Tally the early-termination probability and dose-selection percentages P1/P2 over replications and write the aggregated table (e.g., null_scenario_early_termination.csv). Validate by checking that simulated values reproduce published figures, e.g., a null-scenario early-termination probability of approximately 96.4% for TSNP (vs. 93.3% for P1) within Monte Carlo error.

## Related tools

- **R** (Language used for the TSNP simulation and trial-implementation scripts; crossratio() and Gumbel() generate the joint toxicity-efficacy response probabilities) — https://github.com/yongzang2020/TSNP
- **Shiny app** (Planned graphical user interface for applying the design; still under development, not currently usable for this skill)

## Examples

```
Rscript TSNP_simulation.R --scenario null --phi 0.2 --reps 10000 --out null_scenario_early_termination.csv
```

## Evaluation signals

- Simulated null-scenario early-termination probability for TSNP reproduces approximately 96.4% within Monte Carlo error
- Null-scenario comparison holds: TSNP early termination similar to P2 and better than P1 (93.3%)
- Isotonic estimates from PAVA equal the MLE of pj under the monotonic order constraint
- Early termination is triggered exactly when the admissible set Ω becomes empty, recommending no dose
- Output table includes early-termination probability and P1/P2 dose-selection percentages per design

## Limitations

- Design cannot be directly applied when the response outcome requires long follow-up to assess
- Design is limited to trials with binary, quickly observable response outcomes
- Extension to drug-combination trials is future work, not yet done
- The Shiny app is under development and not yet released

## Evidence

- [methods] Update the admissible set as Ω = {j: r1 ≥ cT ∩ … ∩ rj ≥ cT; j = 1, …, J}. If the admissible set is empty, terminate the trial early, and recommend no dose.: [evidence span withheld — anchor status: not_found]
- [other] In the null scenario (all doses overly toxic), the TSNP design has a 96.4% chance of early trial termination, similar to the P2 design and better than the P1 design (93.3%).: [evidence span withheld — anchor status: not_found]
