---
name: circadian-phase-uncertainty-quantification
description: Use when you have approximate posterior phase samples from Tempo alongside
  known true or sample-collection phases and need to verify that reported phase uncertainty
  (credible intervals) is calibrated in sparse single-cell RNA-seq data.
when_to_use_negative:
- Point estimates of circadian phase accuracy are the goal and no posterior samples
  are available, since existing methods that 'do not quantify estimation uncertainty'
  cannot support credible-interval coverage analysis
- The data lack any reference phases (neither simulated true phases nor time-course
  sample-collection times), so empirical coverage cannot be computed
- Applying uncertainty quantification assumptions to non-Bayesian phase inference
  methods whose likelihood models poorly approximate droplet-based scRNA-seq data
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Tempo
  role: Bayesian variational inference model whose approximate posterior phase samples
    are the source of the credible intervals being calibrated
  repo: https://github.com/bauerbach95/tempo
- name: numpy
  role: Load and manipulate posterior phase sample arrays and compute HDR interval
    boundaries
- name: pandas
  role: Aggregate per-cell coverage results per dataset via group-by and write calibration_coverage.csv
- name: scipy
  role: Support circular statistics for 24 h phase quantities when constructing intervals
- name: scikit-learn
  role: Python dependency of the tempo conda environment supporting the calibration
    analysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:7d73f2616eeb54445963440ce4128c81a76e57be089418fd0508e37248561bf1
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
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# circadian-phase-uncertainty-quantification

## Summary

Validates that Bayesian circadian phase inference (Tempo) produces well-calibrated uncertainty estimates by constructing circular Highest-Density-Region credible intervals from approximate posterior phase samples and comparing empirical coverage against nominal confidence levels on simulated and time-course scRNA-seq datasets.

## When to use

Use when you have approximate posterior phase samples from Tempo alongside known true or sample-collection phases and need to verify that reported phase uncertainty (credible intervals) is calibrated in sparse single-cell RNA-seq data.

## When NOT to use

- Point estimates of circadian phase accuracy are the goal and no posterior samples are available, since existing methods that 'do not quantify estimation uncertainty' cannot support credible-interval coverage analysis
- The data lack any reference phases (neither simulated true phases nor time-course sample-collection times), so empirical coverage cannot be computed
- Applying uncertainty quantification assumptions to non-Bayesian phase inference methods whose likelihood models poorly approximate droplet-based scRNA-seq data

## Inputs

- Tempo approximate posterior phase samples for cells (numpy/pandas-readable tutorial input files)
- Known true phases for simulated time-course datasets
- Sample-collection phases for aorta and hepatocyte scRNA-seq time-course datasets
- Grid of nominal confidence levels (e.g., 10%-99%)

## Outputs

- calibration_coverage.csv (coverage-versus-confidence-level table for all datasets)
- calibration_curves.png (empirical coverage vs nominal level with y=x reference line)

## How to apply

Load Tempo's approximate posterior phase samples (numpy/pandas) for the simulated time-course datasets and the aorta and hepatocyte scRNA-seq datasets from the tutorial-documented input files. For each cell, build Highest-Density-Region credible intervals from the posterior samples at a grid of nominal confidence levels (e.g., 10%-99%), treating phase as a circular 24 h quantity so intervals can wrap around midnight. Compute empirical coverage as the fraction of intervals containing the known true phase (simulated data) or the sample-collection phase (aorta/hepatocyte time-course data) at each level, aggregating per dataset with pandas group-by. Assemble the coverage-versus-confidence-level table into calibration_coverage.csv, then plot calibration curves (empirical coverage vs nominal level with a diagonal y=x reference line) saved as calibration_curves.png. Run inside the tempo conda environment; a well-calibrated, slightly conservative curve (coverage at or above the diagonal) confirms the article's reported behavior.

## Related tools

- **Tempo** (Bayesian variational inference model whose approximate posterior phase samples are the source of the credible intervals being calibrated) — https://github.com/bauerbach95/tempo
- **numpy** (Load and manipulate posterior phase sample arrays and compute HDR interval boundaries)
- **pandas** (Aggregate per-cell coverage results per dataset via group-by and write calibration_coverage.csv)
- **scipy** (Support circular statistics for 24 h phase quantities when constructing intervals)
- **scikit-learn** (Python dependency of the tempo conda environment supporting the calibration analysis)

## Examples

```
conda activate tempo; python calibrate_coverage.py --posterior posterior_samples.csv --true-phase true_phases.csv --out calibration_coverage.csv
```

## Evaluation signals

- The calibration script runs to completion in the tempo conda environment without error, analogous to run_test.py yielding 'SUCCESSFULLY FINISHED'
- Empirical coverage increases monotonically with nominal confidence level across the 10%-99% grid
- Coverage curves lie on or above the y=x diagonal, indicating well-calibrated and slightly conservative uncertainty quantification as reported for Tempo
- Circular HDR intervals correctly wrap around midnight so 24 h phase periodicity is preserved in coverage computation
- calibration_coverage.csv contains rows for every dataset (simulated unsynchronized, simulated time-course, aorta, hepatocyte) at every confidence level

## Limitations

- Coverage is assessed against sample-collection phase for real data, which assumes collection time proxies true circadian phase; the article notes breaking this assumption requires single-cell phase measures
- Tempo assumes CCG expression means follow 24 h sinusoidal patterns, and simulated analyses suggest this assumption is a source of error for de novo cyclers
- Technical effects are not explicitly modeled, which becomes more necessary for data collected as multiple samples over time; subpopulation-specific amplitude variation is not modeled

## Evidence

- [abstract] Tempo provides well-calibrated uncertainty quantifications: "and provides well-calibrated uncertainty quantifications"
- [intro] existing methods do not quantify uncertainty: "existing approaches do not quantify the uncertainty of phase estimates. This becomes crucial for interpretation of results from very sparse droplet-based"
- [methods] Tempo dependency list: "Tempo requires python >= 3.8 and depends on the following python packages"
- [abstract] Tempo quantifies phase estimation uncertainty: "we introduce Tempo, a Bayesian variational inference approach that incorporates domain knowledge of the clock and quantifies phase estimation uncertainty"
- [readme] run_test.py success message: "Running the test should take less than 5 minutes. Successful completion of the test will yield a message "SUCCESSFULLY FINISHED"."
