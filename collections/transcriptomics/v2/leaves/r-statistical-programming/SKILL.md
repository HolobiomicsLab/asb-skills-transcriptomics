---
name: r-statistical-programming
description: Use when reproducing or extending Monte Carlo simulations of the TSNP
  design — e.g., a sample-size sensitivity analysis of correct OBD selection percentages
  — from the R code at github.com/yongzang2020/TSNP.
when_to_use_negative:
- The response outcome requires long follow-up to assess — the TSNP design cannot
  be applied directly in such cases
- The response outcome is not binary and quickly observable shortly after treatment
  initiation
- The goal is a drug-combination trial; extending TSNP to dose-combination trials
  is future work, not implemented in the current code
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Language used for the TSNP simulation and trial implementation code, including
    crossratio() and Gumbel() generators of joint toxicity-efficacy response probabilities
  repo: https://github.com/yongzang2020/TSNP
- name: Shiny app
  role: Graphical user interface under development to facilitate application of the
    TSNP design (not yet released)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pst.2075
    title: '<scp>TSNP</scp>: A <scp>two‐stage</scp> nonparametric phase I/<scp>II</scp>
      clinical trial design for immunotherapy'
  source_content_hash: sha256:d4bee8a569cfdca197985d4a4f4dfa4e945e4f11bdb1a6864c70487bc75d417b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yongzang2020/TSNP
  tools:
  - R
  - Shiny app
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# r-statistical-programming

## Summary

Use R to implement and run the TSNP two-stage nonparametric phase I/II dose-finding simulations, including the crossratio() and Gumbel() joint probability generators, to reproduce operating characteristics such as correct OBD selection percentages across sample sizes N=40, 50, and 60.

## When to use

Use when reproducing or extending Monte Carlo simulations of the TSNP design — e.g., a sample-size sensitivity analysis of correct OBD selection percentages — from the R code at github.com/yongzang2020/TSNP.

## When NOT to use

- The response outcome requires long follow-up to assess — the TSNP design cannot be applied directly in such cases
- The response outcome is not binary and quickly observable shortly after treatment initiation
- The goal is a drug-combination trial; extending TSNP to dose-combination trials is future work, not implemented in the current code

## Inputs

- Simulation R code from github.com/yongzang2020/TSNP implementing the TSNP two-stage nonparametric design
- Scenario definitions with true joint toxicity-efficacy response probabilities generated via crossratio() and Gumbel()
- Design parameters: maximum sample size N (40, 50, 60), target toxicity φ, admissibility thresholds cT and cE, cohort size m

## Outputs

- Per-scenario OBD selection percentages for each sample size (N=40, 50, 60), e.g. obd_selection_percentages.csv with one row per scenario
- Dose allocation percentages and early stopping rates for every scenario
- Validation check that scenario 5 yields 49.0% (N=40), 52.6% (N=50), 58.6% (N=60) correct OBD selection

## How to apply

Obtain the simulation R code from the TSNP repository (github.com/yongzang2020/TSNP), which implements the two-stage nonparametric algorithm: the CCD-based stage 1 toxicity exploration, the Fisher's exact test admissibility rules (toxicity H0: pj≤φ, efficacy exclusion with sj < cE), and utility-based OBD selection. Generate each scenario's true joint toxicity-efficacy response probabilities with the provided crossratio() and Gumbel() functions, set the maximum sample size (e.g., N=40), and run the Monte Carlo simulation. Repeat identically for N=50 and N=60, recording per-scenario OBD selection percentage, dose allocation, and stopping rates. Aggregate per-scenario OBD selection percentages into a CSV (one row per scenario, columns for N=40, 50, 60). Validate by checking that scenario 5 reproduces the reported correct OBD selection percentages of 49.0% (N=40), 52.6% (N=50), and 58.6% (N=60), and that other scenarios agree with the published table within Monte Carlo error.

## Related tools

- **R** (Language used for the TSNP simulation and trial implementation code, including crossratio() and Gumbel() generators of joint toxicity-efficacy response probabilities) — https://github.com/yongzang2020/TSNP
- **Shiny app** (Graphical user interface under development to facilitate application of the TSNP design (not yet released))

## Examples

```
Rscript TSNP_simulation.R --N 50 --scenario 5
```

## Evaluation signals

- Scenario 5 correct OBD selection percentage reproduces 49.0% at N=40, 52.6% at N=50, and 58.6% at N=60
- Correct OBD selection percentage in scenario 5 increases monotonically with sample size across N=40, 50, 60
- Other scenarios' OBD selection rates agree with the published simulation table within Monte Carlo error
- obd_selection_percentages.csv contains one row per scenario with columns for N=40, N=50, and N=60
- Dose allocation and stopping rates are recorded for every scenario and are consistent with the published table

## Limitations

- The design cannot be applied directly when the response outcome requires a long follow-up time to be assessed
- The proposed design is appropriate only for trials in which the response outcome is binary and observable shortly after treatment initiation
- Extension of TSNP to drug-combination trials is future work and not supported by the current code
- The Shiny app providing a graphical user interface is still under development and not yet available

## Evidence
