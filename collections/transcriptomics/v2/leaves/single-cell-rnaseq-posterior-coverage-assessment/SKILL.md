---
name: single-cell-rnaseq-posterior-coverage-assessment
description: Use when you have per-cell posterior samples of a circular (24 h) quantity
  such as circadian phase from a Bayesian single-cell method and need to verify that
  credible-interval uncertainty quantifications are well-calibrated against known
  true phases or sample-collection times.
when_to_use_negative:
- The method under evaluation provides only point estimates of phase and no posterior
  samples or uncertainty quantifications — existing unsupervised phase inference methods
  'do not quantify estimation uncertainty', leaving nothing to calibrate.
- The quantity of interest is not circular/periodic (e.g., linear measurements); the
  HDR interval construction here assumes a 24 h circular phase where intervals wrap
  around midnight.
- No ground-truth phases are available (e.g., unsynchronized samples with no collection-time
  reference), so empirical coverage cannot be computed.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: Tempo
  role: Bayesian variational inference method whose approximate posterior phase samples
    are the subject of the calibration assessment
  repo: https://github.com/bauerbach95/tempo
- name: numpy
  role: Load and manipulate posterior phase sample arrays and construct circular HDR
    credible intervals
- name: pandas
  role: Group-by aggregation of empirical coverage per dataset and writing calibration_coverage.csv
- name: scipy
  role: Support for circular-statistics computations used in HDR interval construction
- name: scikit-learn
  role: Python dependency of the tempo environment used in the calibration workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:387121982bd538815789e132050107683e7700eb4decb5694090456a6dfcdb82
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
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# single-cell-rnaseq-posterior-coverage-assessment

## Summary

A calibration-assessment skill that converts approximate posterior phase samples from a Bayesian single-cell method (e.g., Tempo) into Highest-Density-Region credible intervals and checks whether their empirical coverage matches nominal confidence levels for a circadian 24 h phase quantity.

## When to use

Use when you have per-cell posterior samples of a circular (24 h) quantity such as circadian phase from a Bayesian single-cell method and need to verify that credible-interval uncertainty quantifications are well-calibrated against known true phases or sample-collection times.

## When NOT to use

- The method under evaluation provides only point estimates of phase and no posterior samples or uncertainty quantifications — existing unsupervised phase inference methods 'do not quantify estimation uncertainty', leaving nothing to calibrate.
- The quantity of interest is not circular/periodic (e.g., linear measurements); the HDR interval construction here assumes a 24 h circular phase where intervals wrap around midnight.
- No ground-truth phases are available (e.g., unsynchronized samples with no collection-time reference), so empirical coverage cannot be computed.

## Inputs

- Per-cell approximate posterior phase samples from Tempo (numpy arrays / pandas-readable tutorial input files)
- Ground-truth phases: known true phase for simulated scRNA-seq datasets, or sample-collection phase for time-course datasets (aorta, hepatocyte)
- Grid of nominal confidence levels (e.g., 10%–99%)

## Outputs

- calibration_coverage.csv — coverage-versus-confidence-level table for all datasets
- calibration_curves.png — empirical coverage vs nominal level plot with y=x diagonal reference line

## How to apply

Load Tempo's approximate posterior phase samples (numpy/pandas) for each dataset — simulated scRNA-seq time courses and real time-course datasets such as aorta and hepatocyte. For each cell, build Highest-Density-Region credible intervals from the posterior samples at nominal levels from 10% to 99%, treating phase as a circular 24 h quantity so intervals can wrap around midnight. Compute empirical coverage as the fraction of intervals containing the known true phase (simulated data) or the sample-collection phase (real time-course data), aggregating per dataset with pandas group-by. Assemble the coverage-versus-confidence table (calibration_coverage.csv), plot empirical coverage against the nominal level with the y=x diagonal reference (calibration_curves.png), and run in the tempo conda environment. Well-calibrated posteriors track the diagonal; Tempo's were reported as well-calibrated and slightly conservative (coverage slightly above nominal).

## Related tools

- **Tempo** (Bayesian variational inference method whose approximate posterior phase samples are the subject of the calibration assessment) — https://github.com/bauerbach95/tempo
- **numpy** (Load and manipulate posterior phase sample arrays and construct circular HDR credible intervals)
- **pandas** (Group-by aggregation of empirical coverage per dataset and writing calibration_coverage.csv)
- **scipy** (Support for circular-statistics computations used in HDR interval construction)
- **scikit-learn** (Python dependency of the tempo environment used in the calibration workflow)

## Examples

```
conda activate tempo && python run_test.py  # then run the calibration script on posterior phase samples to emit calibration_coverage.csv and calibration_curves.png
```

## Evaluation signals

- The calibration script executes without error in the tempo conda environment (analogous to run_test.py yielding 'SUCCESSFULLY FINISHED').
- Empirical coverage curves lie on or slightly above the y=x diagonal across the 10%–99% nominal grid, confirming well-calibrated, slightly conservative quantifications as reported for Tempo.
- calibration_coverage.csv contains one coverage value per (dataset, nominal level) pair, with datasets aggregated via pandas group-by.
- HDR intervals correctly wrap around midnight, so cells whose posterior mass straddles phase 0/24 h are not misclassified as uncovered.
- calibration_curves.png renders empirical coverage vs nominal level for all datasets with the diagonal reference line present.

## Limitations

- Coverage assessment requires reference phases: true phase for simulated data or sample-collection phase for real time-course data; unsynchronized samples lack such references.
- Tempo's uncertainty quantification was found slightly conservative (coverage slightly above nominal), which should be expected rather than treated as a failure.
- Tempo's sinusoidal assumption for clock-controlled gene expression means may limit accuracy, and technical effects and subpopulation-specific amplitude variation are not explicitly modeled.

## Evidence

- [abstract] and provides well-calibrated uncertainty quantifications: "we demonstrate that Tempo yields more accurate estimates of circadian phase than existing methods"
- [intro] existing approaches do not quantify estimation uncertainty: [evidence span withheld — anchor status: not_found]
- [methods] conda activate tempo / python run_test.py: "conda activate tempo"
- [readme] README installation and test: "Successful completion of the test will yield a message "SUCCESSFULLY FINISHED"."
- [readme] README python dependencies: "Tempo requires python >= 3.8 and depends on the following python packages:
  - anndata
  - numpy
  - pandas
  - scanpy>=1.6
  - scikit-image"
