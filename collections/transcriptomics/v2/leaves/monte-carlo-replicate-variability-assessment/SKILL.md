---
name: monte-carlo-replicate-variability-assessment
description: Use when comparing stochastic unsupervised phase/cycle inference methods
  on scRNA-seq data and you must demonstrate that point estimates (e.g., circadian
  phase over the 24 h cycle) are reproducible across repeated runs rather than just
  accurate once.
when_to_use_negative:
- The phase-inference method is deterministic (no run-to-run stochasticity), so replicate
  variability assessment adds no information.
- The analysis goal is accuracy against known sampling-time labels rather than reproducibility
  of point estimates across runs.
- Input data are bulk time-course rather than single-cell, where unsupervised per-cell
  phase inference and its stability do not apply.
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: Tempo
  role: Bayesian variational inference method whose five-run phase estimates are assessed
    for stability (median circular SD < 1 h expected)
  repo: github.com/bauerbach95/tempo
- name: anndata
  role: Loads the scRNA-seq datasets used in the stability analysis
- name: scanpy>=1.6
  role: Supports loading and handling of the single-cell datasets prior to replicate
    phase inference
- name: numpy
  role: Computes per-cell circular standard deviations of phase (in hours) across
    the five replicates
- name: pandas
  role: Aggregates per-cell circular SD values and writes stability_distributions.csv
- name: pytorch>=1.9.0
  role: Backend dependency for Tempo's variational inference runs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:22c66984269287393348412bc8c724cb810da953d99b5a2cdd173a8cbabe3533
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Tempo
  - anndata
  - numpy
  - pandas
  - pytorch>=1.9.0
  - scanpy>=1.6
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# monte-carlo-replicate-variability-assessment

## Summary

Quantifies run-to-run variability of unsupervised phase-inference algorithms by running each method (Tempo, Cyclops, Cyclum) five times per scRNA-seq dataset and computing per-cell circular standard deviations of 24 h circadian phase predictions, validating Tempo's median circular SD < 1 h.

## When to use

Use when comparing stochastic unsupervised phase/cycle inference methods on scRNA-seq data and you must demonstrate that point estimates (e.g., circadian phase over the 24 h cycle) are reproducible across repeated runs rather than just accurate once.

## When NOT to use

- The phase-inference method is deterministic (no run-to-run stochasticity), so replicate variability assessment adds no information.
- The analysis goal is accuracy against known sampling-time labels rather than reproducibility of point estimates across runs.
- Input data are bulk time-course rather than single-cell, where unsupervised per-cell phase inference and its stability do not apply.

## Inputs

- scRNA-seq datasets from Fig. 2f simulation settings and Fig. 5d real datasets (including the aorta endothelial-cell dataset), loaded via anndata/scanpy
- Per-cell posterior phase estimates from five independent runs each of Tempo, Cyclops, and Cyclum

## Outputs

- stability_distributions.csv: per-cell circular standard deviation (in hours over the 24 h cycle) across five runs, per method and dataset
- Fig. 2f/5d-style stability distribution plots comparing per-method circular SD spread

## How to apply

Activate the tempo conda environment built from github.com/bauerbach95/tempo via install.sh (Python >= 3.8 with numpy, pandas, anndata, scanpy>=1.6, pytorch>=1.9.0), then load each scRNA-seq dataset used in Fig. 2f (simulation settings) and Fig. 5d (real datasets), including the endothelial-cell dataset, with anndata/scanpy. Run Tempo, Cyclops, and Cyclum five times each per dataset to capture run-to-run variability, and for every run convert per-cell posterior phase estimates to hours over the 24 h circadian cycle. Compute per-cell circular standard deviations across the five replicates, aggregate the distributions per method and dataset, and save the table as stability_distributions.csv. Rationale: circular SD is the correct dispersion metric because phase is angular/periodic, and five replicates suffice to expose the instability of Cyclops and Cyclum versus Tempo's stable Bayesian variational inference. Decision point: verify Tempo's median circular SD is below 1 h on all real datasets except aorta endothelial cells, while Cyclops and Cyclum should show visibly larger-spread (unstable) distributions matching the published Fig. 2f/5d.

## Related tools

- **Tempo** (Bayesian variational inference method whose five-run phase estimates are assessed for stability (median circular SD < 1 h expected)) — github.com/bauerbach95/tempo
- **anndata** (Loads the scRNA-seq datasets used in the stability analysis)
- **scanpy>=1.6** (Supports loading and handling of the single-cell datasets prior to replicate phase inference)
- **numpy** (Computes per-cell circular standard deviations of phase (in hours) across the five replicates)
- **pandas** (Aggregates per-cell circular SD values and writes stability_distributions.csv)
- **pytorch>=1.9.0** (Backend dependency for Tempo's variational inference runs)

## Examples

```
conda activate tempo && python run_test.py  # then, per dataset: run Tempo, Cyclops, Cyclum 5x each and save per-cell circular SDs to stability_distributions.csv
```

## Evaluation signals

- Tempo reproduces a median per-cell circular standard deviation below 1 h across all simulation settings and real datasets except aorta endothelial cells.
- Cyclops and Cyclum produce visibly larger-spread (unstable) per-cell circular SD distributions matching the published Fig. 2f/5d figures.
- stability_distributions.csv contains per-cell circular SD values in hours over the 24 h circadian cycle for every method x dataset x replicate combination.
- Each phase-inference method is run exactly five times per dataset, and circular SD is computed per cell across those five replicates.
- The tempo conda environment passes python run_test.py with 'SUCCESSFULLY FINISHED' before the stability analysis is run.

## Limitations

- Tempo's stability exception: median circular SD is not below 1 h on the aorta endothelial-cell real dataset.
- Results depend on correctly building the tempo environment from github.com/bauerbach95/tempo; Tempo requires Python >= 3.8, scanpy>=1.6, and pytorch>=1.9.0.
- Temporal stability does not establish accuracy — competing methods can be stably wrong; the article separately reports accuracy comparisons.

## Evidence

- [other] Tempo's phase point estimates are stable across repeated runs, with median circular standard deviation below 1 hour on all real datasets except the aorta endothelial cells, whereas Cyclops and Cyclum yielded unstable estimates.: [evidence span withheld — anchor status: not_found]
- [other] For every run, convert per-cell posterior phase estimates to hours over the 24 h circadian cycle and compute per-cell circular standard deviations across the five replicates.: [evidence span withheld — anchor status: not_found]
- [methods] Tempo requires python >= 3.8 and depends on the following python packages: "Tempo requires python >= 3.8 and depends on the following python packages"
- [abstract] Tempo, a Bayesian variational inference approach that incorporates domain knowledge of the clock and quantifies phase estimation uncertainty: "we introduce Tempo, a Bayesian variational inference approach that incorporates domain knowledge of the clock and quantifies phase estimation uncertainty"
- [readme] To test if Tempo works properly, run the run_test.py file using using the activated environment: "To test if Tempo works properly, run the run_test.py file using using the activated environment"
