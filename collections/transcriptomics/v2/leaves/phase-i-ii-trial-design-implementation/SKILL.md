---
name: phase-i-ii-trial-design-implementation
description: Use when reproducing or executing a phase I/II immunotherapy dose-finding
  trial with the TSNP design — e.g., a hypothetical trial with J=5 doses, phi=0.3,
  max sample size 30, m=6 — to obtain per-dose utilities and a final OBD.
when_to_use_negative:
- The response outcome requires long follow-up to assess — the design is limited to
  binary, quickly observable response outcomes
- The trial is a drug-combination dose-finding study — extension of TSNP to combination
  trials is future work and not yet supported
- The goal is conventional MTD-targeting phase I dosing rather than OBD selection
  balancing toxicity and efficacy via a utility function
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: R
  role: R code for simulation and trial implementation of the TSNP design, including
    crossratio() and Gumbel() to generate joint toxicity-efficacy response probabilities
  repo: https://github.com/yongzang2020/TSNP
- name: Shiny app
  role: Graphical user interface for the TSNP design; under development, not yet released
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/pst.2075
    title: '<scp>TSNP</scp>: A <scp>two‐stage</scp> nonparametric phase I/<scp>II</scp>
      clinical trial design for immunotherapy'
  source_content_hash: sha256:8e21da47609473f52f87ed184ad90fa9807c74eda3f6e8350fee7bf37ae97dc3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yongzang2020/TSNP
  tools:
  - R
  - Shiny app
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# phase-i-ii-trial-design-implementation

## Summary

Implements the two-stage nonparametric (TSNP) phase I/II clinical trial design for identifying the optimal biological dose (OBD) of immunotherapy, using the R code from github.com/yongzang2020/TSNP. Stage 1 explores the toxicity profile via isotonic (PAVA) estimates and Fisher's exact admissibility tests; stage 2 selects the OBD by maximizing a utility function U(pi-hat_jkl, omega_kl) over the admissible set.

## When to use

Use when reproducing or executing a phase I/II immunotherapy dose-finding trial with the TSNP design — e.g., a hypothetical trial with J=5 doses, phi=0.3, max sample size 30, m=6 — to obtain per-dose utilities and a final OBD.

## When NOT to use

- The response outcome requires long follow-up to assess — the design is limited to binary, quickly observable response outcomes
- The trial is a drug-combination dose-finding study — extension of TSNP to combination trials is future work and not yet supported
- The goal is conventional MTD-targeting phase I dosing rather than OBD selection balancing toxicity and efficacy via a utility function

## Inputs

- Design parameters: number of doses J (e.g., 5), highest acceptable DLT rate phi (e.g., 0.3), CCD cut-off tau, nominal toxicity level cT, cohort size m (e.g., 6), maximum sample size (e.g., 30)
- Physician-elicited desirability scores omega (omega01=100, omega10=0, omega00 and omega11 on the (0,100) scale)
- Per-dose cohort counts of patients and joint toxicity-efficacy outcome counts (0/0, 1/0, 0/1, 1/1 combinations)

## Outputs

- Per-dose utility values U(pi-hat_jkl, omega_kl) for all doses in the final admissible set
- Final admissible set membership for each dose level
- Final OBD recommendation (the dose with the highest utility; e.g., dose 4 with utility 46.67 in the illustrative example)
- Result table such as tsnp_example_trial_result.csv

## How to apply

Load the TSNP trial-implementation R functions from the github.com/yongzang2020/TSNP repository and configure the design inputs: J pre-specified increasing doses, highest acceptable toxicity rate phi (e.g., 0.3), the CCD cut-off tau, and physician-elicited desirability scores omega (omega01=100, omega10=0, with omega00 and omega11 elicited on the (0,100) scale). In stage 1, treat the first cohort at the lowest or physician-specified dose, compute isotonic toxicity estimates with the pool-adjacent-violators algorithm (PAVA), compare p-hat_j with phi - tau to escalate, de-escalate, or retain, and apply Fisher's exact tests (H0: p_j <= phi, nominal level cT) to build the toxicity admissible set; repeat until m patients have been treated at the current dose and stop stage 1 via the early stopping rule. In stage 2, update the admissible set with accumulated data, escalate if the highest tried dose j* is safe, otherwise assign the next cohort to the dose maximizing the utility U(pi-hat_jkl, omega_kl) computed from closed-form MLEs of the joint toxicity-efficacy probabilities; repeat until the maximum sample size is reached. At trial end, construct the final admissible set and recommend the highest-utility dose as the final OBD (in the illustrative example, all doses were admissible and dose 4 achieved the maximum utility of 46.67).

## Related tools

- **R** (R code for simulation and trial implementation of the TSNP design, including crossratio() and Gumbel() to generate joint toxicity-efficacy response probabilities) — https://github.com/yongzang2020/TSNP
- **Shiny app** (Graphical user interface for the TSNP design; under development, not yet released)

## Examples

```
git clone https://github.com/yongzang2020/TSNP
```

## Evaluation signals

- Reproducing the illustrative example yields all doses in the admissible set (no dose declared overly toxic) and dose 4 with the maximum utility of 46.67 as the final OBD
- Isotonic toxicity estimates from PAVA satisfy the monotonic order constraint (1) across the J dose levels
- Any dose with an empty admissible set triggers early termination with no dose recommended
- Stage-1 dose assignments follow the p-hat_j vs phi - tau comparison (escalate if p-hat_j <= phi - tau, de-escalate if above, otherwise retain)
- The output table (e.g., tsnp_example_trial_result.csv) contains per-dose utilities, admissible set membership, and the OBD recommendation

## Limitations

- It cannot be applied directly to cases in which the response outcome requires a long follow-up time to be assessed
- The proposed design is appropriate for trials in which the response outcome is binary and observable shortly after treatment initiation
- Extension of TSNP to drug-combination trials is future work, not yet done
- The easy-to-use Shiny app is still under development and not yet released

## Evidence
