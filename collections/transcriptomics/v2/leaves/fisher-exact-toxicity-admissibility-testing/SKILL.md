---
name: fisher-exact-toxicity-admissibility-testing
description: Use when implementing or simulating the TSNP two-stage dose-finding design
  and you must decide which dose levels are sufficiently safe (not overly toxic) to
  remain candidates for the OBD.
when_to_use_negative:
- The response outcome requires long follow-up to assess — the design is appropriate
  only for binary outcomes observable shortly after treatment initiation
- The trial targets drug combinations rather than single-agent dose finding — extending
  TSNP to combination trials is future work
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Simulation and trial implementation of the Fisher's exact admissibility screen
    within the TSNP design scripts
  repo: https://github.com/yongzang2020/TSNP
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_004
  - task_003
  source_papers:
  - doi: 10.1002/pst.2075
    title: '<scp>TSNP</scp>: A <scp>two‐stage</scp> nonparametric phase I/<scp>II</scp>
      clinical trial design for immunotherapy'
  source_content_hash: sha256:8bbcbb9985153bdea411d3f80b4f78c0bc40c181dc4a3013922d15af1a58342d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yongzang2020/TSNP
  tools:
  - R
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# fisher-exact-toxicity-admissibility-testing

## Summary

Fisher's exact-test admissibility screening from the TSNP phase I/II design: at each dose, test H0: pj <= phi (highest acceptable DLT rate) and exclude doses whose p-value rj falls below the nominal level cT, building the toxicity admissible set used for OBD selection.

## When to use

Use when implementing or simulating the TSNP two-stage dose-finding design and you must decide which dose levels are sufficiently safe (not overly toxic) to remain candidates for the OBD.

## When NOT to use

- The response outcome requires long follow-up to assess — the design is appropriate only for binary outcomes observable shortly after treatment initiation
- The trial targets drug combinations rather than single-agent dose finding — extending TSNP to combination trials is future work

## Inputs

- Per-dose DLT counts and cohort sizes accumulated during the trial (stage 1 CCD or stage 2)
- Highest acceptable toxicity rate phi (e.g., 0.3 or 0.2)
- Nominal admissibility level cT
- Number of pre-specified doses J

## Outputs

- Fisher's exact test p-values rj per dose
- Updated toxicity admissible set Omega
- Early-termination decision (recommend no dose) when Omega is empty

## How to apply

For each dose level j, compute the Fisher's exact test p-value rj for the hypothesis H0: pj <= phi against the accumulated DLT data, where phi is the highest acceptable toxicity rate (e.g., 0.3 in the main simulations, changed to 0.2 for the null scenario). Update the admissible set as Omega = {j: r1 >= cT, ..., rj >= cT, j = 1, ..., J}, i.e., a dose is admitted only if all doses up to and including it pass the test at the nominal level cT. If the admissible set becomes empty, terminate the trial early and recommend no dose — this rule gives the TSNP design a 96.4% early-termination probability under the all-toxic null scenario. In stage 1 this screen runs alongside the cumulative cohort design (CCD) with PAVA isotonic toxicity estimates compared against phi - tau; in stage 2 the admissible set is updated with accumulated data and utility U(pi_jkl_hat, omega_kl) is maximized only over admissible doses. The R implementation is available in the github.com/yongzang2020/TSNP repository.

## Related tools

- **R** (Simulation and trial implementation of the Fisher's exact admissibility screen within the TSNP design scripts) — https://github.com/yongzang2020/TSNP

## Examples

```
Rscript TSNP_simulation.R --phi 0.3 --cT 0.9 --scenario 1  # Fisher's exact admissibility screen runs inside the TSNP two-stage simulation
```

## Evaluation signals

- Reproduced null-scenario (all doses overly toxic, phi = 0.2) early-termination probability ≈ 96.4% within Monte Carlo error
- Admissible set definition matches Omega = {j: r1 >= cT ∩ ... ∩ rj >= cT; j = 1, ..., J} with cumulative membership across doses 1..j
- Doses excluded via rj < cT never receive patients in stage 2 and are never candidates for the OBD recommendation
- Simulated OBD selection percentages for TSNP match Tables 1-2 of the article within Monte Carlo error when the screen is active

## Limitations

- Limited to binary, quickly observable toxicity outcomes; not applicable when DLT assessment requires long follow-up
- The screen is a nominal-level test (cT), so its operating characteristics depend on the chosen cT and phi values

## Evidence

- [methods] Update the admissible set as Ω = {j: r1 ≥ cT ∩ … ∩ rj ≥ cT; j = 1, …, J}. If the admissible set is empty, terminate the trial early, and recommend no dose.: [evidence span withheld — anchor status: not_found]
