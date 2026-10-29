---
name: cumulative-cohort-dose-escalation
description: Use when implementing or reproducing a phase I/II immunotherapy trial
  (e.g., the TSNP design) and you need to conduct the first-stage toxicity exploration
  — escalating, de-escalating, or retaining doses in cohorts of m patients based on
  isotonic toxicity estimates versus φ−τ.
when_to_use_negative:
- The response outcome requires long follow-up time to assess — the TSNP design cannot
  be applied directly in that setting
- The response outcome is not binary and not observable shortly after treatment initiation
  — the design is appropriate only for quickly observable binary responses
- The trial is a drug-combination study seeking an optimal dose combination — extending
  TSNP to combinations is explicitly future work, not yet done
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Language for the freely available TSNP trial-implementation and simulation
    code, including crossratio() and Gumbel() for joint toxicity-efficacy response
    probabilities
  repo: https://github.com/yongzang2020/TSNP
- name: Shiny app
  role: Planned graphical user interface for applying the TSNP design; still under
    development and not yet released
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pst.2075
    title: '<scp>TSNP</scp>: A <scp>two‐stage</scp> nonparametric phase I/<scp>II</scp>
      clinical trial design for immunotherapy'
  source_content_hash: sha256:4723681d19050df49753c9c71b5b4f552fcb68fa3f89c18ac9e1ca1992a1796c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yongzang2020/TSNP
  tools:
  - R
  - Shiny app
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cumulative-cohort-dose-escalation

## Summary

Stage 1 dose-escalation procedure of the TSNP phase I/II design, which uses cumulative-cohort (CCD) toxicity data, PAVA isotonic estimates, and the φ−τ escalation boundary with Fisher's exact toxicity admissibility tests to explore the toxicity profile before utility-based OBD selection.

## When to use

Use when implementing or reproducing a phase I/II immunotherapy trial (e.g., the TSNP design) and you need to conduct the first-stage toxicity exploration — escalating, de-escalating, or retaining doses in cohorts of m patients based on isotonic toxicity estimates versus φ−τ.

## When NOT to use

- The response outcome requires long follow-up time to assess — the TSNP design cannot be applied directly in that setting
- The response outcome is not binary and not observable shortly after treatment initiation — the design is appropriate only for quickly observable binary responses
- The trial is a drug-combination study seeking an optimal dose combination — extending TSNP to combinations is explicitly future work, not yet done

## Inputs

- Design parameters: number of dose levels J, highest acceptable toxicity rate phi, CCD cut-off tau, nominal toxicity-test level cT, cohort size m (e.g., m = 6), maximum sample size (e.g., 30)
- Per-dose cohort counts of patients and joint toxicity-efficacy outcome counts (e.g., no-toxicity/no-efficacy, no-toxicity/efficacy, toxicity/no-efficacy, toxicity/efficacy)
- Physician-elicited desirability scores omega (e.g., omega01 = 100, omega10 = 0, omega00 and omega11 elicited on (0, 100))
- TSNP R functions from github.com/yongzang2020/TSNP (including crossratio() and Gumbel() for generating joint toxicity-efficacy response probabilities)

## Outputs

- Current dose recommendation (escalate / de-escalate / retain) at each cohort decision point
- Updated toxicity admissible set Ω (dose indices passing Fisher's exact tests at level cT)
- Early-termination flag if the admissible set is empty
- Stage-1 completion status triggering the transition to stage 2
- Final per-dose utility values and OBD recommendation (e.g., written to tsnp_example_trial_result.csv)

## How to apply

Treat the first cohort at the lowest (or physician-specified) dose, then at each cohort decision compute isotonic toxicity estimates p̂_j with the pool-adjacent-violators algorithm (PAVA), which are equivalent to the MLE of p_j under the monotonic order constraint. Compare p̂_j to the boundary φ−τ (φ = highest acceptable toxicity rate, τ = the recommended CCD cut-off): escalate if p̂_j ≤ φ−τ, de-escalate if p̂_j > φ−τ, otherwise retain the current dose, and repeat until m patients have been treated at the current dose. After each cohort, apply Fisher's exact tests (H0: p_j ≤ φ) at nominal level cT to update the toxicity admissible set Ω = {j: r_1 ≥ cT ∩ … ∩ r_j ≥ cT}; if Ω is empty, terminate early and recommend no dose. Apply the stage-1 stopping rule to trigger transition to stage 2, where the remaining patients are used for utility-based OBD selection using U(π̂_jkl, ω_kl). In the illustrative example (φ = 0.3, J = 5 doses, max N = 30, m = 6), stage 1 ended with all doses safe and dose 4 reached the highest utility (46.67) as the final OBD.

## Related tools

- **R** (Language for the freely available TSNP trial-implementation and simulation code, including crossratio() and Gumbel() for joint toxicity-efficacy response probabilities) — https://github.com/yongzang2020/TSNP
- **Shiny app** (Planned graphical user interface for applying the TSNP design; still under development and not yet released)

## Evaluation signals

- PAVA output satisfies the monotonic order constraint on toxicity probabilities, and the isotonic estimates equal the MLE of p_j under that constraint
- At each cohort, the dose decision matches the φ−τ rule: escalate when p̂_j ≤ φ−τ, de-escalate when p̂_j exceeds φ−τ, otherwise retain
- Admissible set membership is consistent with Fisher's exact test p-values r_j at nominal level cT (dose j is admissible iff r_j ≥ cT through the cumulative set)
- Stage 1 terminates only after m patients (e.g., 6) have been treated at the current dose and the stage-1 stopping rule triggers
- For the illustrative example trial, the reproduced result shows no dose declared overly toxic and dose 4 with the maximum utility of 46.67 recommended as the final OBD

## Limitations

- Cannot be applied directly when the response outcome requires long follow-up to assess
- Appropriate only for trials with binary response outcomes observable shortly after treatment initiation
- Extension to phase I/II drug-combination trials is future work and not yet available
- A user-friendly Shiny app is under development and not yet released; software must currently be used via the R code repository

## Evidence

- [methods] Fisher's exact test toxicity admissibility: [evidence span withheld — anchor status: not_found]
- [methods] Update admissible set and terminate if empty: [evidence span withheld — anchor status: not_found]
