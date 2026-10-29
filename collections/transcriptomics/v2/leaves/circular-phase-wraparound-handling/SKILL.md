---
name: circular-phase-wraparound-handling
description: Use when computing credible intervals or coverage statistics over 24
  h circadian phase estimates whose posterior density straddles phase 0/24 (midnight),
  e.g. validating Tempo's uncertainty calibration on scRNA-seq time-course data.
when_to_use_negative:
- The quantity being interval-estimated is linear/unbounded (not a 24 h circular phase),
  where wraparound handling would distort interval widths.
- Only point estimates of circadian phase are needed — the article notes existing
  methods that do not quantify estimation uncertainty, but this skill specifically
  requires posterior phase samples.
- Data were not analyzed with Tempo or lack approximate posterior samples, so HDR
  credible intervals cannot be constructed.
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_3304
- http://edamontology.org/topic_3517
tools:
- name: Tempo
  role: Source of approximate posterior phase samples whose uncertainty quantification
    is being calibration-tested; installed and run in the tempo conda environment
  repo: https://github.com/bauerbach95/tempo
- name: numpy
  role: Load posterior phase sample arrays and compute circular HDR interval boundaries
    across confidence levels
- name: pandas
  role: Aggregate per-dataset empirical coverage via group-by and write calibration_coverage.csv
- name: scipy
  role: Support density estimation for Highest-Density-Region computation from posterior
    samples
- name: scikit-learn
  role: Supporting analysis/plotting stack for the calibration workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:491a3c4069aeaff32da35c5d4e13819c0ec307b6a35b2110d6abc7e10187837d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bauerbach95/tempo
  tools:
  - Python
  - Tempo
  - numpy
  - pandas
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_3304
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# circular-phase-wraparound-handling

## Summary

Treats circadian phase as a circular 24 h quantity when constructing Highest-Density-Region (HDR) credible intervals from Tempo's approximate posterior phase samples, so intervals that cross midnight wrap correctly and calibration coverage is computed without artificially splitting nighttime intervals.

## When to use

Use when computing credible intervals or coverage statistics over 24 h circadian phase estimates whose posterior density straddles phase 0/24 (midnight), e.g. validating Tempo's uncertainty calibration on scRNA-seq time-course data.

## When NOT to use

- The quantity being interval-estimated is linear/unbounded (not a 24 h circular phase), where wraparound handling would distort interval widths.
- Only point estimates of circadian phase are needed — the article notes existing methods that do not quantify estimation uncertainty, but this skill specifically requires posterior phase samples.
- Data were not analyzed with Tempo or lack approximate posterior samples, so HDR credible intervals cannot be constructed.

## Inputs

- Tempo approximate posterior phase samples per cell (numeric arrays from tutorial-documented input files, loaded via numpy/pandas)
- Ground-truth phases: known true phase for simulated scRNA-seq data, or sample-collection phase for aorta and hepatocyte time-course datasets
- Grid of nominal confidence levels (e.g. 10%–99%)

## Outputs

- calibration_coverage.csv — coverage-versus-confidence-level table for all datasets
- calibration_curves.png — empirical coverage vs nominal level plot with y=x diagonal reference line

## How to apply

Load Tempo's approximate posterior phase samples per cell (e.g. with numpy/pandas from the tutorial-documented input files) for simulated time-course datasets and the aorta and hepatocyte scRNA-seq datasets. For each cell, construct Highest-Density-Region credible intervals from the posterior phase samples at a grid of nominal confidence levels (e.g. 10%–99%), treating phase as a circular 24 h quantity so intervals wrap around midnight rather than being truncated at 0/24. Compute empirical coverage as the fraction of wrap-aware intervals containing the known true phase (simulated data) or the sample-collection phase (aorta/hepatocyte time-course data), aggregating per dataset with pandas group-by. Assemble the coverage-versus-confidence-level table and plot calibration curves against the y=x diagonal: well-calibrated intervals track the diagonal; the article reports Tempo's coverage is slightly conservative. Validate by running the calibration script in the tempo conda environment to completion, analogous to run_test.py yielding 'SUCCESSFULLY FINISHED'.

## Related tools

- **Tempo** (Source of approximate posterior phase samples whose uncertainty quantification is being calibration-tested; installed and run in the tempo conda environment) — https://github.com/bauerbach95/tempo
- **numpy** (Load posterior phase sample arrays and compute circular HDR interval boundaries across confidence levels)
- **pandas** (Aggregate per-dataset empirical coverage via group-by and write calibration_coverage.csv)
- **scipy** (Support density estimation for Highest-Density-Region computation from posterior samples)
- **scikit-learn** (Supporting analysis/plotting stack for the calibration workflow)

## Examples

```
conda activate tempo && python calibration_coverage.py --posterior phase_samples.csv --truth true_phases.csv --levels 0.10-0.99 --out calibration_coverage.csv
```

## Evaluation signals

- The calibration script in the tempo conda environment completes without error, analogous to run_test.py yielding 'SUCCESSFULLY FINISHED'.
- calibration_coverage.csv contains one empirical-coverage value per (dataset, nominal level) pair with coverage computed on wrap-around-midnight intervals.
- Empirical coverage curves sit at or above the y=x diagonal (slightly conservative), matching the reported finding that Tempo's HDR credible intervals are well-calibrated.
- Intervals whose posterior density straddles phase 0/24 are not split or truncated — spot-check that coverage near midnight is not artificially deflated.
- calibration_curves.png renders empirical coverage vs nominal confidence with the diagonal reference line for all datasets (simulated, aorta, hepatocyte).

## Limitations

- Calibration on real aorta/hepatocyte data relies on sample-collection phase as a proxy for true cell phase, which the article identifies as an assumption of time-course designs.
- Tempo assumes CCG expression means follow 24 h sinusoidal patterns; waveform mis-specification could distort posterior phase shapes and thus HDR interval geometry near midnight.
- Tempo does not explicitly model technical effects for data collected as multiple samples over time, which may affect posterior samples in time-course settings.

## Evidence

- [results] Tempo's Highest-Density-Region credible intervals from the approximate phase posteriors are well-calibrated for both unsynchronized and time course data, with coverage slightly conservative.: [evidence span withheld — anchor status: not_found]
- [methods] construct Highest-Density-Region credible intervals from the posterior phase samples at a grid of nominal confidence levels (e.g., 10%–99%), treating phase as a circular 24 h quantity so intervals wrap around midnight: [evidence span withheld — anchor status: not_found]
- [abstract] we introduce Tempo, a Bayesian variational inference approach that incorporates domain knowledge of the clock and quantifies phase estimation uncertainty: "we introduce Tempo, a Bayesian variational inference approach that incorporates domain knowledge of the clock and quantifies phase estimation uncertainty"
- [readme] Running the test should take less than 5 minutes. Successful completion of the test will yield a message "SUCCESSFULLY FINISHED".: "Running the test should take less than 5 minutes. Successful completion of the test will yield a message "SUCCESSFULLY FINISHED"."
