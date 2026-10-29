---
name: credible-interval-calibration-analysis
description: Use when a Bayesian phase-inference method (e.g., Tempo) returns posterior
  phase samples per cell and you must verify whether its uncertainty quantifications
  are well-calibrated on sparse droplet-based scRNA-seq data.
when_to_use_negative:
- The method under evaluation only produces point estimates of phase with no posterior
  samples, since existing methods 'do not quantify estimation uncertainty' and thus
  cannot generate credible intervals.
- The quantity of interest is point-estimate accuracy of circadian phase rather than
  uncertainty quantification — a phase-accuracy comparison workflow is the wrong tool
  here.
tools:
- name: Tempo
  role: Source of approximate posterior phase samples whose Highest-Density-Region
    credible intervals are being calibrated
  repo: https://github.com/bauerbach95/tempo
- name: numpy
  role: Load and manipulate posterior phase sample arrays when constructing circular
    HDR intervals
- name: pandas
  role: Group-by aggregation of per-dataset empirical coverage and export of calibration_coverage.csv
- name: scipy
  role: Python dependency of the tempo environment used in the calibration workflow
- name: scikit-learn
  role: Python dependency of the tempo environment used in the calibration workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:31acb36eb72b34af14e6fa6e8de9b8fbb3a7e0314d2baaf1c0e5a0eb429d47cf
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# credible-interval-calibration-analysis

## Summary

Validates Bayesian posterior uncertainty by constructing circular Highest-Density-Region credible intervals from Tempo's approximate posterior phase samples and comparing their empirical coverage against nominal confidence levels on scRNA-seq circadian data.

## When to use

Use when a Bayesian phase-inference method (e.g., Tempo) returns posterior phase samples per cell and you must verify whether its uncertainty quantifications are well-calibrated on sparse droplet-based scRNA-seq data.

## When NOT to use

- The method under evaluation only produces point estimates of phase with no posterior samples, since existing methods 'do not quantify estimation uncertainty' and thus cannot generate credible intervals.
- The quantity of interest is point-estimate accuracy of circadian phase rather than uncertainty quantification — a phase-accuracy comparison workflow is the wrong tool here.

## Inputs

- Tempo approximate posterior phase samples per cell (numpy/pandas-readable tutorial input files) for simulated time-course scRNA-seq datasets
- Tempo posterior phase samples for aorta and hepatocyte time-course scRNA-seq datasets with associated sample-collection phases
- Grid of nominal confidence levels (e.g., 10%–99%)

## Outputs

- calibration_coverage.csv — coverage-versus-confidence-level table per dataset
- calibration_curves.png — empirical coverage vs nominal level plot with y=x diagonal reference line

## How to apply

Load Tempo's approximate posterior phase samples (from the tutorial-documented input files) for simulated time-course datasets and real time-course scRNA-seq datasets such as aorta and hepatocyte samples. For each cell, build Highest-Density-Region credible intervals on a grid of nominal confidence levels (e.g., 10%–99%), treating 24 h phase as circular so intervals wrap around midnight. Compute empirical coverage as the fraction of intervals containing the known true phase (simulated) or sample-collection phase (aorta/hepatocyte), aggregating per dataset with pandas group-by. Write the coverage-versus-confidence-level table to calibration_coverage.csv and plot coverage against nominal level with a diagonal y=x reference line, saving calibration_curves.png; well-calibrated intervals lie on or above the diagonal (slightly conservative coverage is expected).

## Related tools

- **Tempo** (Source of approximate posterior phase samples whose Highest-Density-Region credible intervals are being calibrated) — https://github.com/bauerbach95/tempo
- **numpy** (Load and manipulate posterior phase sample arrays when constructing circular HDR intervals)
- **pandas** (Group-by aggregation of per-dataset empirical coverage and export of calibration_coverage.csv)
- **scipy** (Python dependency of the tempo environment used in the calibration workflow)
- **scikit-learn** (Python dependency of the tempo environment used in the calibration workflow)

## Examples

```
conda activate tempo && python run_test.py
```

## Evaluation signals

- The calibration script executes in the tempo conda environment without error (analogous to run_test.py yielding "SUCCESSFULLY FINISHED").
- Empirical coverage curves lie at or above the y=x diagonal, confirming well-calibrated, slightly conservative uncertainty quantification as reported for Tempo.
- calibration_coverage.csv contains a coverage value at every nominal confidence level for each dataset (simulated time-course, aorta, hepatocyte).
- Circular wrap-around is handled: cells whose HDR interval crosses midnight are still counted as covered when the true phase falls within the wrapped interval.
- The plot calibration_curves.png includes the diagonal reference line and one curve per dataset.

## Limitations

- Coverage for aorta/hepatocyte data is assessed against sample-collection phase, which assumes sample timing proxies circadian phase — the very assumption unsupervised phase inference seeks to avoid.
- Tempo's sinusoidal assumption for clock-controlled gene expression means may bias phase posteriors; the article notes this 'may be due, in part, to the assumption that the expression means of CCGs follow 24 h sinusoidal patterns'.
- For multiple samples collected over time, explicit modeling of technical effects becomes necessary and is not modeled here.

## Evidence

- [abstract] Tempo provides well-calibrated uncertainty quantifications: "and provides well-calibrated uncertainty quantifications"
- [intro] Existing methods lack uncertainty quantification: [evidence span withheld — anchor status: not_found]
- [readme] Tempo conda environment required for running the calibration workflow: "you can activate the conda environment containing the installed Tempo package:
conda activate tempo"
- [readme] Validation signal for successful execution: "Successful completion of the test will yield a message "SUCCESSFULLY FINISHED"."
- [methods] Tempo dependency stack grounding the tool list: "Tempo requires python >= 3.8 and depends on the following python packages: - anndata - numpy - pandas"
