---
name: cross-design-operating-characteristics-comparison
description: 'Use when two phase I/II dose-finding designs (e.g., TSNP vs U-BOIN)
  must be compared on the same simulated scenarios: generate correlated outcomes with
  Gumbel() in R, run both designs with matched seeds, and tabulate OBD selection and
  patient-allocation percentages.'
when_to_use_negative:
- The response outcome is binary but requires long follow-up to assess; TSNP is only
  appropriate for responses observable shortly after treatment initiation
- The goal is a drug-combination phase I/II trial — the TSNP design does not yet handle
  dose-combination optimization (future work)
- The comparison targets a Bayesian parametric design benchmark rather than U-BOIN
  — that is a different comparison workflow than this TSNP-vs-U-BOIN skill
edam_operation: http://edamontology.org/operation_3659
edam_topics: []
tools:
- name: R
  role: Runs the TSNP two-stage simulation, the U-BOIN simulation path, and the Gumbel()/crossratio()
    generation of joint toxicity-efficacy response probabilities
  repo: https://github.com/yongzang2020/TSNP
- name: TSNP (R code)
  role: Repository code implementing the TSNP design simulation and trial comparison
    workflow
  repo: https://github.com/yongzang2020/TSNP
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/pst.2075
    title: '<scp>TSNP</scp>: A <scp>two‐stage</scp> nonparametric phase I/<scp>II</scp>
      clinical trial design for immunotherapy'
  source_content_hash: sha256:bbd38815e06381ffbc3ce5844e9e0da8bce6a49870846d373a80ab03283320c7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yongzang2020/TSNP
  tools:
  - R
  - TSNP (R code)
  edam_operation: http://edamontology.org/operation_3659
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cross-design-operating-characteristics-comparison

## Summary

Head-to-head simulation comparison of the TSNP two-stage nonparametric phase I/II design against U-BOIN under identical correlated toxicity-efficacy scenarios (Gumbel copula, association parameter 0.2), benchmarking OBD selection percentages and per-dose patient-assignment operating characteristics.

## When to use

Use when two phase I/II dose-finding designs (e.g., TSNP vs U-BOIN) must be compared on the same simulated scenarios: generate correlated outcomes with Gumbel() in R, run both designs with matched seeds, and tabulate OBD selection and patient-allocation percentages.

## When NOT to use

- The response outcome is binary but requires long follow-up to assess; TSNP is only appropriate for responses observable shortly after treatment initiation
- The goal is a drug-combination phase I/II trial — the TSNP design does not yet handle dose-combination optimization (future work)
- The comparison targets a Bayesian parametric design benchmark rather than U-BOIN — that is a different comparison workflow than this TSNP-vs-U-BOIN skill

## Inputs

- Scenario configuration: number of doses J, target toxicity rate phi, cut-off tau, admissibility thresholds cT and cE, stage-1 cohort size m, maximum sample size
- Elicited desirability scores omega_kl (with omega_01=100 and omega_10=0)
- True joint toxicity-efficacy response probabilities per scenario generated via the Gumbel() function (association parameter 0.2)
- Common random seeds shared by the TSNP and U-BOIN simulation paths

## Outputs

- Side-by-side operating-characteristics table written to tsnp_uboin_simulation_results.csv
- Per-scenario OBD selection percentages for TSNP and U-BOIN
- Average number of patients treated at each dose level for both designs

## How to apply

Load the TSNP R code from yongzang2020/TSNP and configure each scenario with J doses, target toxicity rate phi, cut-off tau, admissibility levels cT and cE, stage-1 cohort size m, maximum sample size, and elicited desirability scores omega_kl (omega_01=100, omega_10=0). Generate joint toxicity-efficacy response probabilities per scenario using Gumbel() (and crossratio()) from the R code. Run the TSNP simulation: stage 1 uses PAVA isotonic toxicity estimates compared against phi and tau to escalate/de-escalate/retain, with Fisher's exact test p-values rj < cT excluding overly toxic doses from the admissible set; stage 2 assigns cohorts to the dose maximizing U(pi_jkl_hat, omega_kl) within the admissible set. Run U-BOIN on identical scenarios and seeds for the head-to-head contrast, then aggregate over replicates the OBD selection percentages and average number of patients at each dose for both designs.

## Related tools

- **R** (Runs the TSNP two-stage simulation, the U-BOIN simulation path, and the Gumbel()/crossratio() generation of joint toxicity-efficacy response probabilities) — https://github.com/yongzang2020/TSNP
- **TSNP (R code)** (Repository code implementing the TSNP design simulation and trial comparison workflow) — https://github.com/yongzang2020/TSNP

## Evaluation signals

- Verify that joint probabilities from Gumbel() yield valid toxicity-efficacy response tables consistent with the monotonic increasing toxicity constraint used by PAVA
- Confirm Fisher's exact test p-values rj < cT correctly exclude overly toxic doses from the admissible set Omega at every interim
- Check that both designs consumed the same scenarios and seeds, so differences in OBD selection are attributable to the designs, not the data stream
- Confirm the resulting table shows comparable operating characteristics: U-BOIN slightly better in scenarios 1 and 2, TSNP better in the other two scenarios (Table 3 setting, association parameter 0.2)
- Confirm patient counts across dose levels sum to the maximum sample size (or fewer if early stopping with no dose recommended occurred)

## Limitations

- TSNP is limited to trials with binary, quickly observable response outcomes
- The design cannot be directly applied when the response outcome requires long follow-up to assess
- Extension to drug-combination trials is future work, not yet available
- The Shiny app GUI is still under development, so all comparison runs must use the raw R code

## Evidence
