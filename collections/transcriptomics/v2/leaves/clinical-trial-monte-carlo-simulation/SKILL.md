---
name: clinical-trial-monte-carlo-simulation
description: Use when estimating operating characteristics — early-termination probability,
  dose-selection percentages P1/P2 — of the TSNP design for a given toxicity/efficacy
  scenario, such as the null (all doses overly toxic, φ=0.2) case.
when_to_use_negative:
- The response outcome requires long follow-up to assess — the TSNP design only handles
  binary, quickly observable response outcomes
- The goal is dose-combination trials — extension of TSNP to drug combinations is
  future work, not implemented
- A Bayesian parametric phase I/II design simulation is required — this skill specifically
  simulates the nonparametric TSNP design and its P1/P2 comparators
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Language used for both the simulation code and trial implementation of the
    TSNP design
  repo: https://github.com/yongzang2020/TSNP
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pst.2075
    title: '<scp>TSNP</scp>: A <scp>two‐stage</scp> nonparametric phase I/<scp>II</scp>
      clinical trial design for immunotherapy'
  source_content_hash: sha256:447c291f86dc112e32231e4e78b5471f0456c8bf4ce40d2557fec600ccd7b0d8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yongzang2020/TSNP
  tools:
  - R
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# clinical-trial-monte-carlo-simulation

## Summary

Monte Carlo simulation of the TSNP two-stage nonparametric phase I/II design's operating characteristics (e.g. early-termination probability under the null scenario with φ changed from 0.3 to 0.2) by replicating thousands of simulated trials in R and aggregating selection percentages into a CSV table.

## When to use

Use when estimating operating characteristics — early-termination probability, dose-selection percentages P1/P2 — of the TSNP design for a given toxicity/efficacy scenario, such as the null (all doses overly toxic, φ=0.2) case.

## When NOT to use

- The response outcome requires long follow-up to assess — the TSNP design only handles binary, quickly observable response outcomes
- The goal is dose-combination trials — extension of TSNP to drug combinations is future work, not implemented
- A Bayesian parametric phase I/II design simulation is required — this skill specifically simulates the nonparametric TSNP design and its P1/P2 comparators

## Inputs

- Design parameters (φ, τ, cT, cE, m, cohort size, maximum sample size) with φ set to 0.2 in the null scenario
- True toxicity (DLT) and efficacy (TR) probability vectors per dose — all doses overly toxic and inefficacious in the null case
- TSNP simulation and trial-implementation R scripts from github.com/yongzang2020/TSNP

## Outputs

- Aggregated operating-characteristics table per design (TSNP, P1, P2) with early-termination probability and dose-selection percentages P1/P2, written to null_scenario_early_termination.csv
- Per-scenario estimates of early-termination probability reproducible within Monte Carlo error (e.g. TSNP ≈ 96.4% in the null scenario)

## How to apply

Obtain the TSNP simulation and trial-implementation R scripts from github.com/yongzang2020/TSNP and configure the target scenario: for the null case, set true DLT rates so every dose exceeds the highest acceptable toxicity rate (φ changed from 0.3 to 0.2) and true TR rates below the lowest acceptable efficacy rate. Simulate trial replications of the two-stage design: stage 1 uses a cumulative cohort design with PAVA isotonic toxicity estimates, dose escalation/de-escalation against the cutoff φ−τ, and Fisher's exact toxicity admissibility tests at nominal level cT. In each replicated trial, apply the early-termination rule — update the admissible set Ω = {j: r1 ≥ cT ∩ … ∩ rj ≥ cT} and, if it is empty, end the trial and recommend no dose; otherwise proceed to stage 2 utility-based OBD selection. Tally the early-termination probability and dose-selection percentages P1/P2 across replications and write the aggregated table to a CSV (e.g. null_scenario_early_termination.csv). Correctness is validated by reproducing reported operating characteristics (e.g. TSNP null-scenario early termination ≈ 96.4%) within Monte Carlo error.

## Related tools

- **R** (Language used for both the simulation code and trial implementation of the TSNP design) — https://github.com/yongzang2020/TSNP

## Evaluation signals

- Simulated null-scenario early-termination probability for the TSNP design reproduces the reported ≈96.4% within Monte Carlo error
- Null-scenario comparison matches the paper: TSNP ≈ P2 and both better than P1 (93.3% early termination)
- Simulated trials terminate early exactly when the admissible set Ω = {j: r1 ≥ cT ∩ … ∩ rj ≥ cT} becomes empty and recommend no dose
- Isotonic toxicity estimates satisfy the monotonic order constraint (PAVA estimates are the MLE under the order constraint)
- Output CSV contains early-termination probability and P1/P2 columns for each design

## Limitations

- Validation is by reproduction within Monte Carlo error; small numbers of replications may not match the reported 96.4% exactly
- Design cannot be directly applied when the response outcome requires long follow-up to assess
- Limited to binary, quickly observable response outcomes; drug-combination extension is future work

## Evidence

- [methods] Update the admissible set as Ω = {j: r1 ≥ cT ∩ … ∩ rj ≥ cT; j = 1, …, J}. If the admissible set is empty, terminate the trial early, and recommend no dose.: [evidence span withheld — anchor status: not_found]
- [methods] the isotonic estimates pȷ^ are equivalent to the MLE of pj under the order constraint (1): "the isotonic estimates pȷ^ are equivalent to the MLE of pj under the order constraint (1)"
- [results] All the codes are freely available at https://github.com/yongzang2020/TSNP.: "All the codes are freely available at https://github.com/yongzang2020/TSNP."
- [readme] R code for simulation and trial implementation of the TSNP design: "R code for simulation and trial implementation of the TSNP design"
- [discussion] It cannot be applied directly to cases in which the response outcome requires a long follow-up time to be assessed.: "It cannot be applied directly to cases in which the response outcome requires a long follow-up time to be assessed."
