---
name: bayesian-parametric-comparator-design-implementation
description: Use when benchmarking a novel phase I/II dose-finding design against
  established Bayesian parametric designs (Thall–Cook P1, Yin et al. P2) in Monte
  Carlo simulations with a fixed highest acceptable DLT rate phi.
when_to_use_negative:
- The response outcome requires long follow-up or is not binary and quickly observable
  shortly after treatment initiation — the TSNP framework and its comparator setup
  do not apply
- The goal is to calibrate or tune the parametric designs themselves rather than to
  reproduce their published operating characteristics as a benchmark
tools:
- name: R
  role: Language in which the P1/P2 comparator design functions and the simulation
    harness are implemented and run
  repo: https://github.com/yongzang2020/TSNP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/pst.2075
    title: '<scp>TSNP</scp>: A <scp>two‐stage</scp> nonparametric phase I/<scp>II</scp>
      clinical trial design for immunotherapy'
  source_content_hash: sha256:ebd8a775bedd055aa1baa30e31300def4dc42f731110912b9b3de8b64c780e02
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yongzang2020/TSNP
  tools:
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# bayesian-parametric-comparator-design-implementation

## Summary

Implements the Bayesian parametric phase I/II comparator designs P1 (Thall et al.) and P2 (Yin et al.) in R so their OBD-selection operating characteristics can be benchmarked against the TSNP design under identical scenarios.

## When to use

Use when benchmarking a novel phase I/II dose-finding design against established Bayesian parametric designs (Thall–Cook P1, Yin et al. P2) in Monte Carlo simulations with a fixed highest acceptable DLT rate phi.

## When NOT to use

- The response outcome requires long follow-up or is not binary and quickly observable shortly after treatment initiation — the TSNP framework and its comparator setup do not apply
- The goal is to calibrate or tune the parametric designs themselves rather than to reproduce their published operating characteristics as a benchmark

## Inputs

- Dose-toxicity-efficacy scenario tables (true joint response probabilities per dose, Tables 1-2)
- Highest acceptable DLT rate phi (e.g., 0.3)
- Utility desirability scores omega00, omega01, omega10, omega11
- Maximum sample size, cohort size, and stage-1 dose count m
- Comparator design functions P1 (Thall et al.) and P2 (Yin et al.) from the TSNP repository

## Outputs

- Per-dose OBD selection percentage for P1 and P2
- No-dose (early-termination) rate per scenario
- Average percentage of patients assigned to each dose
- Comparison table against TSNP operating characteristics (Tables 1-2 format)

## How to apply

Load the comparator design functions from the github.com/yongzang2020/TSNP R repository alongside the TSNP scripts. Generate true joint toxicity-efficacy response probabilities for each dose-toxicity-efficacy scenario (Tables 1-2) using the article's crossratio() and Gumbel() generators, fixing the highest acceptable DLT rate phi = 0.3 and the utility desirability scores (omega01 = 100, omega10 = 0, with elicited omega00 and omega11) so that U(pi_jkl, omega_kl) is identical across all designs. Run P1 and P2 under the same scenarios and maximum sample sizes as the TSNP simulation, which uses PAVA isotonic toxicity estimates and a Fisher's exact test toxicity-admissibility screen at nominal level cT. Aggregate over Monte Carlo replicates the percentage of trials selecting each dose as OBD, the no-dose rate, and the average percentage of patients treated at each dose, then compare against the TSNP design's values.

## Related tools

- **R** (Language in which the P1/P2 comparator design functions and the simulation harness are implemented and run) — https://github.com/yongzang2020/TSNP

## Evaluation signals

- Reproduced P1 and P2 OBD selection percentages match the article's Tables 1-2 within Monte Carlo error (scenario 1, phi = 0.3: P1 = 37.4%, P2 = 60.0% vs TSNP = 64.6%)
- Comparator runs use identical scenarios, phi, utility scores, and sample sizes as the TSNP simulation
- No-dose rate and patient-assignment percentages are aggregated over the same number of Monte Carlo replicates as TSNP
- Joint probabilities from crossratio()/Gumbel() yield a valid 2x2 joint outcome distribution at every dose

## Limitations

- The comparators are Bayesian parametric designs whose sophisticated probability models are exactly what the article identifies as limiting in practice
- The TSNP framework, and hence this benchmarking setup, is limited to trials with binary, quickly observable response outcomes and cannot handle drug-combination dosing
- The Shiny app for the TSNP design is still under development, so all comparator runs must go through the R scripts

## Evidence

- [readme] R code and simulation: "R code for simulation and trial implementation of the TSNP design"
- [results] Probability generators: "we have developed easy-to-use R codes. In particular, crossratio () and Gumbel() generate the joint toxicity-efficacy response probabilities"
- [abstract] Superiority conclusion: "The simulation results show that the TSNP design yields superior operating characteristics than the existing Bayesian parametric designs."
- [methods] Toxicity admissibility test: "Let rj be the p-value testing this hypothesis based on Fisher’s exact test and cT the nominal level to claim overly toxic."
