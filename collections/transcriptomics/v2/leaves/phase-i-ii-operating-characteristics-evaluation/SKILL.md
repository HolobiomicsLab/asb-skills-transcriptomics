---
name: phase-i-ii-operating-characteristics-evaluation
description: 'Use when reproducing or extending TSNP operating characteristics: you
  have scenario definitions with true joint toxicity-efficacy probabilities and need
  per-scenario OBD selection rates at multiple maximum sample sizes.'
when_to_use_negative:
- The response outcome requires long follow-up to assess or is not binary and quickly
  observable — the TSNP design is explicitly limited to binary, shortly observable
  responses
- The trial involves drug combinations — extension of TSNP to drug-combination trials
  is stated as future work, not yet implemented
- You only need a single-N operating characteristics run with no sample-size sensitivity
  comparison — the N=40/50/60 replication loop is unnecessary overhead
edam_operation: http://edamontology.org/operation_0282
tools:
- name: R
  role: Runs the Monte Carlo simulation of the TSNP design; crossratio() and Gumbel()
    generate the true joint toxicity-efficacy response probabilities for each scenario
  repo: https://github.com/yongzang2020/TSNP
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pst.2075
    title: '<scp>TSNP</scp>: A <scp>two‐stage</scp> nonparametric phase I/<scp>II</scp>
      clinical trial design for immunotherapy'
  source_content_hash: sha256:1c4d8c5068d4d971fb811b5e237d61e45a82aa9d8d8bbb52b49f9ed8b45e296c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yongzang2020/TSNP
  tools:
  - R
  edam_operation: http://edamontology.org/operation_0282
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# phase-i-ii-operating-characteristics-evaluation

## Summary

Runs Monte Carlo simulations of the two-stage nonparametric (TSNP) phase I/II dose-finding design across maximum sample sizes (N=40, 50, 60) to evaluate correct optimal biological dose (OBD) selection percentages per scenario and identify the smallest N meeting performance criteria.

## When to use

Use when reproducing or extending TSNP operating characteristics: you have scenario definitions with true joint toxicity-efficacy probabilities and need per-scenario OBD selection rates at multiple maximum sample sizes.

## When NOT to use

- The response outcome requires long follow-up to assess or is not binary and quickly observable — the TSNP design is explicitly limited to binary, shortly observable responses
- The trial involves drug combinations — extension of TSNP to drug-combination trials is stated as future work, not yet implemented
- You only need a single-N operating characteristics run with no sample-size sensitivity comparison — the N=40/50/60 replication loop is unnecessary overhead

## Inputs

- TSNP simulation R code from github.com/yongzang2020/TSNP
- Simulation scenario definitions with true joint toxicity-efficacy response probabilities (generated via crossratio() and Gumbel())
- Target maximum sample sizes to compare (e.g. N=40, N=50, N=60)

## Outputs

- obd_selection_percentages.csv with one row per scenario and columns for N=40, N=50, and N=60
- Per-scenario OBD selection percentages, dose allocation, and stopping rates for each sample size

## How to apply

Obtain the R simulation code from github.com/yongzang2020/TSNP, which implements the CCD-based stage 1, admissibility testing, and utility-based OBD selection. For each simulation scenario, generate true joint toxicity-efficacy response probabilities using the provided crossratio() and Gumbel() functions and set the maximum sample size to a target N. Run the Monte Carlo simulation identically for N=40, N=50, and N=60, recording OBD selection percentage, dose allocation, and stopping rates for every scenario. Aggregate the per-scenario OBD selection percentages into a table (e.g. obd_selection_percentages.csv) with one row per scenario and columns for N=40, N=50, and N=60. Validate against the published table: scenario 5 should show 49.0% at N=40, 52.6% at N=50, and 58.6% at N=60, with other scenarios matching the published values within Monte Carlo error. The sample-size comparison answers which N achieves at least 50% correct OBD selection across scenarios (N=60 in the published study).

## Related tools

- **R** (Runs the Monte Carlo simulation of the TSNP design; crossratio() and Gumbel() generate the true joint toxicity-efficacy response probabilities for each scenario) — https://github.com/yongzang2020/TSNP

## Evaluation signals

- Reproduced scenario 5 correct OBD selection percentages match the published values exactly at all three N: 49.0% (N=40), 52.6% (N=50), 58.6% (N=60)
- Other scenarios' OBD selection rates agree with the published simulation table within Monte Carlo error
- Correct OBD selection percentage monotonically increases with sample size within each scenario (49.0% < 52.6% < 58.6% in scenario 5)
- The output CSV contains one row per scenario and columns for each of N=40, N=50, and N=60
- N=60 reaches the criterion of at least 50% correct OBD selection across simulation scenarios

## Limitations

- Monte Carlo error means reproduced percentages must be validated against the published table within simulation error, not exactly for all scenarios
- The Shiny app for graphical simulation/trial execution was still under development, so all evaluation relies on R code
- Results are specific to the TSNP design's assumptions (monotonic increasing toxicity constraint, binary quickly observable outcomes); they do not transfer to designs with different admissibility or utility rules

## Evidence

- [results] correct OBD selection increases with N: [evidence span withheld — anchor status: not_found]
