---
name: isotonic-toxicity-estimation-pava
description: Use when estimating dose-toxicity (DLT) probabilities in a phase I/II
  dose-finding design such as TSNP, where isotonic estimates p̂j must satisfy a monotone
  increasing order constraint before escalation decisions or utility-based OBD selection.
when_to_use_negative:
- The efficacy/response outcome requires long follow-up to be assessed, or is not
  binary and quickly observable after treatment initiation
- The trial is a drug-combination phase I/II study — the article states extension
  of TSNP to drug-combination trials is future work, not yet done
- A parametric Bayesian model is explicitly required (e.g., reproducing Thall–Cook
  P1 or Yin et al. P2 comparators rather than the nonparametric TSNP design)
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: R
  role: Implements PAVA isotonic estimation, Fisher's exact admissibility tests, and
    the full TSNP simulation and trial-implementation pipeline, including crossratio()
    and Gumbel() probability generators
  repo: https://github.com/yongzang2020/TSNP
- name: Shiny app
  role: Planned graphical user interface for applying the design; still under development
    and not yet released
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_004
  - task_005
  source_papers:
  - doi: 10.1002/pst.2075
    title: '<scp>TSNP</scp>: A <scp>two‐stage</scp> nonparametric phase I/<scp>II</scp>
      clinical trial design for immunotherapy'
  source_content_hash: sha256:5eb98b55b29c596619c3f2ddb79de77983d532b77b623be5b42b812fbf95145d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yongzang2020/TSNP
  tools:
  - R
  - Shiny app
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# isotonic-toxicity-estimation-pava

## Summary

Derives nonparametric, closed-form isotonic estimates of dose-toxicity probabilities using the pool-adjacent-violators algorithm (PAVA), enforcing the monotone increasing toxicity constraint required by the TSNP two-stage phase I/II design for immunotherapy.

## When to use

Use when estimating dose-toxicity (DLT) probabilities in a phase I/II dose-finding design such as TSNP, where isotonic estimates p̂j must satisfy a monotone increasing order constraint before escalation decisions or utility-based OBD selection.

## When NOT to use

- The efficacy/response outcome requires long follow-up to be assessed, or is not binary and quickly observable after treatment initiation
- The trial is a drug-combination phase I/II study — the article states extension of TSNP to drug-combination trials is future work, not yet done
- A parametric Bayesian model is explicitly required (e.g., reproducing Thall–Cook P1 or Yin et al. P2 comparators rather than the nonparametric TSNP design)

## Inputs

- Cohort-level binary DLT counts per dose level j = 1..J (toxicity outcomes observable shortly after treatment)
- Number of pre-specified dose levels J (e.g., J = 5)
- Highest acceptable toxicity rate φ (e.g., 0.3, or 0.2 in the null scenario)
- Cumulative-cohort-design cut-off τ and Fisher's exact test nominal level cT
- Cohort size m for stage-1 repetition and maximum sample size (e.g., 30)

## Outputs

- Isotonic toxicity estimates p̂j (non-decreasing across doses) satisfying the order constraint
- Stage-1 dose-assignment decisions (escalate / de-escalate / retain) based on p̂j vs φ−τ
- Toxicity admissible set Ω = {j : rj ≥ cT} used for early termination and stage-2 OBD selection

## How to apply

Accumulate DLT counts per cohort at the current dose and apply the pool-adjacent-violators algorithm to obtain isotonic estimates p̂j that satisfy the order constraint across the J dose levels; these estimates are equivalent to the MLE of pj under that constraint. In TSNP stage 1, compare p̂j with the cut-off φ−τ (e.g., highest acceptable DLT rate φ = 0.3 or 0.2) to decide whether to escalate, de-escalate, or retain the current dose, repeating until m patients have been treated at the current dose. In parallel, run Fisher's exact tests of H0: pj ≤ φ with p-value rj, and exclude any dose with rj < cT from the admissible set; if the admissible set becomes empty, terminate early and recommend no dose. In stage 2, the same PAVA-based toxicity estimates feed the closed-form joint toxicity-efficacy probabilities π̂jkl used in the utility function U(π̂jkl, ωkl), with desirability scores ω01 = 100 and ω10 = 0, to identify the OBD. Use the R code from github.com/yongzang2020/TSNP, where crossratio() and Gumbel() generate the true joint toxicity-efficacy response probabilities for simulation.

## Related tools

- **R** (Implements PAVA isotonic estimation, Fisher's exact admissibility tests, and the full TSNP simulation and trial-implementation pipeline, including crossratio() and Gumbel() probability generators) — https://github.com/yongzang2020/TSNP
- **Shiny app** (Planned graphical user interface for applying the design; still under development and not yet released)

## Evaluation signals

- Isotonic estimates p̂j are non-decreasing across dose levels, verifying the monotone increasing toxicity constraint is satisfied
- PAVA estimates equal the MLE of pj under the order constraint, providing a correctness cross-check
- Reproduced simulation with φ = 0.3 matches Table 1: TSNP OBD selection 64.6% vs 37.4% for P1 and 60.0% for P2 in scenario 1, within Monte Carlo error
- Null scenario (all doses overly toxic, φ changed to 0.2) reproduces early-termination probability ≈ 96.4% for TSNP vs 93.3% for P1
- Illustrative trial walk-through reproduces all doses in the final admissible set and dose 4 with highest utility 46.67 as the OBD

## Limitations

- The design cannot be applied directly to cases in which the response outcome requires a long follow-up time to be assessed
- Appropriate only for trials where the response outcome is binary and observable shortly after treatment initiation
- Extension to identify optimal dose combinations in phase I/II drug-combination trials is future work and not yet supported

## Evidence
