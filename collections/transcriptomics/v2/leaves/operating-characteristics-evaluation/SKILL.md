---
name: operating-characteristics-evaluation
description: Use when a phase I/II dose-finding design needs quantitative performance
  evaluation across dose-toxicity-efficacy scenarios, e.g. reproducing OBD selection
  or null-case early-termination rates from simulated trials.
when_to_use_negative:
- The response outcome requires long follow-up to assess — the TSNP design is only
  appropriate when response outcomes are binary and observable shortly after treatment
  initiation
- The goal is dose-finding for drug combinations — the TSNP design covers single-agent
  trials; extension to combination trials is future work
- The target is MTD only via a conventional phase I design rather than OBD via joint
  toxicity-efficacy monitoring
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Implementation language for the TSNP simulation and trial-implementation scripts,
    including crossratio() and Gumbel() probability generators and comparator designs
    P1/P2
  repo: https://github.com/yongzang2020/TSNP
- name: Shiny app
  role: Planned graphical user interface for applying the TSNP design; still under
    development, so simulation currently relies on the R scripts
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1002/pst.2075
    title: '<scp>TSNP</scp>: A <scp>two‐stage</scp> nonparametric phase I/<scp>II</scp>
      clinical trial design for immunotherapy'
  source_content_hash: sha256:629a08793ac8c9c2e5aad60979916808bc8cfb80c8550a81b74bcc40a6d23d8f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yongzang2020/TSNP
  tools:
  - R
  - Shiny app
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# operating-characteristics-evaluation

## Summary

Monte Carlo simulation of the TSNP two-stage nonparametric phase I/II design to estimate operating characteristics (OBD selection percentages, patient-assignment percentages, and early-termination probability) and benchmark them against Bayesian parametric comparators (P1, P2).

## When to use

Use when a phase I/II dose-finding design needs quantitative performance evaluation across dose-toxicity-efficacy scenarios, e.g. reproducing OBD selection or null-case early-termination rates from simulated trials.

## When NOT to use

- The response outcome requires long follow-up to assess — the TSNP design is only appropriate when response outcomes are binary and observable shortly after treatment initiation
- The goal is dose-finding for drug combinations — the TSNP design covers single-agent trials; extension to combination trials is future work
- The target is MTD only via a conventional phase I design rather than OBD via joint toxicity-efficacy monitoring

## Inputs

- Dose-toxicity-efficacy scenario tables (Tables 1-2 of the article) as true joint response probabilities
- Design parameters: highest acceptable DLT rate phi (0.3; 0.2 in the null scenario), cut-off phi−tau, toxicity admissibility level cT, efficacy futility level cE, cohort size, m patients per stage-1 dose, maximum sample size
- Desirability scores omega00, omega01=100, omega10=0, omega11
- R scripts from github.com/yongzang2020/TSNP (stage-1 CCD, stage-2 utility-based OBD selection, comparator design functions)

## Outputs

- Per-scenario operating-characteristics table: percentage of trials selecting each dose as OBD, no-dose rate, and average percentage of patients treated at each dose for TSNP, P1, and P2
- Null-scenario early-termination probability per design (e.g. null_scenario_early_termination.csv)
- Monte Carlo error bounds for all reported percentages

## How to apply

Obtain the R simulation and trial-implementation scripts from github.com/yongzang2020/TSNP and generate true joint toxicity-efficacy probabilities for each scenario with the crossratio() (or Gumbel()) probability generator, fixing desirability scores (omega01=100, omega10=0, elicited omega00, omega11) and the utility U(pi_jkl, omega_kl). Stage 1 uses the cumulative cohort design: PAVA isotonic toxicity estimates drive escalation/de-escalation against the cut-off phi−tau, with Fisher's exact tests (H0: pj≤phi at nominal level cT) maintaining the admissibility set; if the admissible set becomes empty, terminate early and recommend no dose. Stage 2 treats cohorts at the dose maximizing the utility estimate until the maximum sample size, then recommends the highest-utility dose as final OBD. Run comparator designs P1 (Thall–Cook) and P2 (Yin et al.) under identical scenarios, aggregate over Monte Carlo replicates the OBD-selection percentage, no-dose rate, per-dose patient assignment, and early-termination probability, and write aggregated tables (e.g. null_scenario_early_termination.csv) for validation against published values.

## Related tools

- **R** (Implementation language for the TSNP simulation and trial-implementation scripts, including crossratio() and Gumbel() probability generators and comparator designs P1/P2) — https://github.com/yongzang2020/TSNP
- **Shiny app** (Planned graphical user interface for applying the TSNP design; still under development, so simulation currently relies on the R scripts)

## Examples

```
Rscript TSNP_simulation.R  # run stage-1 CCD + stage-2 utility-based OBD simulation from github.com/yongzang2020/TSNP with phi=0.3 and the crossratio()-generated scenario probabilities
```

## Evaluation signals

- Reproduced OBD selection percentages for TSNP, P1, and P2 match Tables 1-2 of the article within Monte Carlo error (e.g. scenario 1, phi=0.3: TSNP 64.6%, P1 37.4%, P2 60.0%)
- Null scenario with all doses overly toxic and phi changed to 0.2: TSNP early-termination probability ≈ 96.4% (vs P2 similar and P1 93.3%), within Monte Carlo error
- Stage-1 isotonic toxicity estimates satisfy the monotonic order constraint (PAVA outputs are MLE under the constraint)
- Trials where the admissibility set Ω becomes empty must terminate early and recommend no dose — no OBD should be reported for those replicates
- Per-replicate patient counts sum to the maximum sample size across doses plus early-terminating trials

## Limitations

- The design cannot be applied directly when the response outcome requires long follow-up; it is limited to binary, quickly observable response outcomes
- The Shiny app is under development, so there is currently no released GUI for trial implementation
- Extension to phase I/II drug-combination trials is not yet available
- Results depend on elicited desirability scores (omega00, omega11) and thresholds (cT, cE, tau); comparator performance can vary with these choices

## Evidence

- [methods] Update the admissible set as Ω = {j: r1 ≥ cT ∩ … ∩ rj ≥ cT; j = 1, …, J}. If the admissible set is empty, terminate the trial early, and recommend no dose.: [evidence span withheld — anchor status: not_found]
