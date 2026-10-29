---
name: obd-selection-rate-analysis
description: Use when characterizing how the TSNP design's correct OBD selection percentage
  varies with maximum sample size (N=40, 50, 60), or when verifying whether a given
  N achieves at least 50% correct OBD selection across simulation scenarios.
when_to_use_negative:
- The response outcome requires long follow-up to assess — the TSNP design (and hence
  this simulation) assumes binary, quickly observable outcomes
- The goal is a drug-combination trial dose-finding design — extending TSNP to combination
  trials is future work not yet implemented
- Only a single sample size is of interest with no comparison across N — a simpler
  single-scenario simulation suffices
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_2640
tools:
- name: R
  role: Language used to run the TSNP Monte Carlo simulations and produce OBD selection
    percentages across sample sizes
  repo: https://github.com/yongzang2020/TSNP
- name: TSNP R code
  role: Implements the two-stage nonparametric design (CCD-based stage 1, admissibility
    testing, utility-based OBD selection) used in the simulations
  repo: https://github.com/yongzang2020/TSNP
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pst.2075
    title: '<scp>TSNP</scp>: A <scp>two‐stage</scp> nonparametric phase I/<scp>II</scp>
      clinical trial design for immunotherapy'
  source_content_hash: sha256:c1510a5acc96f7fbd07e0d2b230248be3b7c4bef52b32eed0483506e0eed61e4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yongzang2020/TSNP
  tools:
  - R
  - TSNP R code
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# obd-selection-rate-analysis

## Summary

Reproduces and analyzes the sample-size sensitivity of correct optimal biological dose (OBD) selection for the TSNP two-stage nonparametric phase I/II design, running Monte Carlo simulations at N=40, 50, and 60 and comparing per-scenario OBD selection percentages.

## When to use

Use when characterizing how the TSNP design's correct OBD selection percentage varies with maximum sample size (N=40, 50, 60), or when verifying whether a given N achieves at least 50% correct OBD selection across simulation scenarios.

## When NOT to use

- The response outcome requires long follow-up to assess — the TSNP design (and hence this simulation) assumes binary, quickly observable outcomes
- The goal is a drug-combination trial dose-finding design — extending TSNP to combination trials is future work not yet implemented
- Only a single sample size is of interest with no comparison across N — a simpler single-scenario simulation suffices

## Inputs

- TSNP simulation R code from github.com/yongzang2020/TSNP
- Per-scenario true joint toxicity-efficacy response probabilities generated via crossratio() and Gumbel()
- Maximum sample size settings (N=40, N=50, N=60)
- Monte Carlo simulation configuration (scenarios, target toxicity rate, admissibility thresholds)

## Outputs

- obd_selection_percentages.csv with one row per scenario and columns for N=40, N=50, N=60
- Per-scenario dose allocation and early-stopping rates for each sample size
- Identification of the minimum N meeting the ≥50% correct OBD selection criterion (N=60 in the published study)

## How to apply

Obtain the simulation R code from the TSNP repository (github.com/yongzang2020/TSNP), which implements the two-stage nonparametric dose-finding algorithm including the CCD-based stage 1, admissibility testing, and utility-based OBD selection. For each simulation scenario, generate the true joint toxicity-efficacy response probabilities using the provided crossratio() and Gumbel() functions, and set the maximum sample size to a target N (e.g., 40). Run the Monte Carlo simulation identically for N=40, N=50, and N=60, recording the OBD selection percentage, dose allocation, and stopping rates for every scenario. Aggregate the per-scenario OBD selection percentages into a CSV file with one row per scenario and columns for each N. Validate by checking that the reproduced scenario 5 OBD selection percentages match the reported values (49.0% at N=40, 52.6% at N=50, 58.6% at N=60), and that other scenarios agree with the published simulation table within Monte Carlo error.

## Related tools

- **R** (Language used to run the TSNP Monte Carlo simulations and produce OBD selection percentages across sample sizes) — https://github.com/yongzang2020/TSNP
- **TSNP R code** (Implements the two-stage nonparametric design (CCD-based stage 1, admissibility testing, utility-based OBD selection) used in the simulations) — https://github.com/yongzang2020/TSNP

## Examples

```
Rscript run_TSNP_simulation.R --N 60 --scenario 5  # repeat with --N 40 and --N 50 to build obd_selection_percentages.csv
```

## Evaluation signals

- Reproduced scenario 5 correct OBD selection percentage matches 49.0% (N=40), 52.6% (N=50), and 58.6% (N=60) within Monte Carlo error
- OBD selection percentage increases monotonically with sample size across N=40, 50, 60 in the reported scenario
- Other scenarios' OBD selection rates agree with the published simulation table within Monte Carlo error
- obd_selection_percentages.csv has exactly one row per scenario and three sample-size columns
- Per-scenario stopping rates and dose allocations are recorded for every N and are internally consistent with the OBD selection results

## Limitations

- The design cannot be applied directly when the response outcome requires long follow-up to be assessed
- The proposed design is appropriate only for trials with binary, quickly observable response outcomes
- Extending the TSNP design to drug-combination trials is future work, not yet done
- Monte Carlo error means reproduced percentages should match published values only approximately, within simulation error

## Evidence

- [results] Sample-size sensitivity finding: [evidence span withheld — anchor status: not_found]
