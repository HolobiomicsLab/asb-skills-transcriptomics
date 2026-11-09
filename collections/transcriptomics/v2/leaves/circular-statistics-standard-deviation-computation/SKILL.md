---
name: circular-statistics-standard-deviation-computation
description: Use when quantifying run-to-run stability of circular quantities (e.g.,
  circadian phase in hours on a 24 h cycle) by computing per-cell circular standard
  deviation across five or more replicate runs of a phase-inference method on scRNA-seq
  data.
when_to_use_negative:
- The quantity being summarized is linear (not circular/wrapping) — ordinary standard
  deviation is appropriate and circular SD would distort it
- Only a single run per method is available — run-to-run circular SD across replicates
  cannot be computed
- The goal is phase accuracy against ground truth rather than run-to-run stability
  of the phase point estimates
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Tempo
  role: Bayesian variational phase-inference method whose per-cell posterior phase
    estimates, run five times per dataset, provide the circular quantities summarized
    by circular standard deviation
  repo: https://github.com/bauerbach95/tempo
- name: Python >= 3.8
  role: Runtime for the tempo conda environment in which the phase-to-hours conversion
    and circular SD computation are implemented
- name: numpy
  role: Array handling for stacking the five per-run phase vectors before circular
    SD computation
- name: pandas
  role: Writing and aggregating the per-cell circular SD table (stability_distributions.csv)
- name: anndata
  role: Loading each scRNA-seq dataset (Fig. 2f simulations and Fig. 5d real datasets,
    including endothelial cells) into per-cell structures
- name: scanpy>=1.6
  role: Reading and manipulating the scRNA-seq datasets prior to phase inference
- name: pytorch>=1.9.0
  role: Backend dependency for running Tempo's variational inference across the five
    replicates
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:dea076f3b5d4e4b3d65a31ad7545e0f6a53a17ce3d8b718db4d500f3775b7fe5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bauerbach95/tempo
  tools:
  - Python >= 3.8
  - Tempo
  - anndata
  - numpy
  - pandas
  - pytorch>=1.9.0
  - scanpy>=1.6
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# circular-statistics-standard-deviation-computation

## Summary

Compute per-cell circular standard deviations of circadian phase estimates (expressed in hours over the 24 h circadian cycle) across replicate runs of phase-inference methods (Tempo, Cyclops, Cyclum) to quantify run-to-run stability, as in the Tempo paper's Fig. 2f/5d stability analysis.

## When to use

Use when quantifying run-to-run stability of circular quantities (e.g., circadian phase in hours on a 24 h cycle) by computing per-cell circular standard deviation across five or more replicate runs of a phase-inference method on scRNA-seq data.

## When NOT to use

- The quantity being summarized is linear (not circular/wrapping) — ordinary standard deviation is appropriate and circular SD would distort it
- Only a single run per method is available — run-to-run circular SD across replicates cannot be computed
- The goal is phase accuracy against ground truth rather than run-to-run stability of the phase point estimates

## Inputs

- Per-cell posterior circadian phase estimates from each of five replicate runs of Tempo, Cyclops, or Cyclum on a scRNA-seq dataset (anndata/scanpy objects or equivalent phase vectors)
- ScRNA-seq datasets used in Fig. 2f (simulation settings) and Fig. 5d (real datasets, including the aorta endothelial-cell dataset)

## Outputs

- Per-cell circular standard deviation table (stability_distributions.csv) with one row per cell per method per dataset
- Fig. 2f/5d-style stability distribution plots comparing per-cell circular SD distributions across methods and datasets

## How to apply

Run each phase-inference method (Tempo, Cyclops, Cyclum) five times per dataset — simulation settings from Fig. 2f and real datasets from Fig. 5d, including the endothelial-cell dataset — using the tempo conda environment (Python >= 3.8, numpy, pandas, anndata, scanpy>=1.6, pytorch>=1.9.0). For every run, convert per-cell posterior phase estimates to hours over the 24 h circadian cycle, then compute the per-cell circular standard deviation across the five replicates using circular statistics (e.g., wrapping the phase values to [0, 24) so that 23 h and 1 h are treated as adjacent). Aggregate the per-cell circular SD distributions per method and dataset, and compare medians: the stability criterion is a median circular SD below 1 h. Rationale: linear standard deviation is inappropriate for phase because the 24 h cycle has no natural origin, so circular SD correctly measures spread across the cycle boundary.

## Related tools

- **Tempo** (Bayesian variational phase-inference method whose per-cell posterior phase estimates, run five times per dataset, provide the circular quantities summarized by circular standard deviation) — https://github.com/bauerbach95/tempo
- **Python >= 3.8** (Runtime for the tempo conda environment in which the phase-to-hours conversion and circular SD computation are implemented)
- **numpy** (Array handling for stacking the five per-run phase vectors before circular SD computation)
- **pandas** (Writing and aggregating the per-cell circular SD table (stability_distributions.csv))
- **anndata** (Loading each scRNA-seq dataset (Fig. 2f simulations and Fig. 5d real datasets, including endothelial cells) into per-cell structures)
- **scanpy>=1.6** (Reading and manipulating the scRNA-seq datasets prior to phase inference)
- **pytorch>=1.9.0** (Backend dependency for running Tempo's variational inference across the five replicates)

## Examples

```
import numpy as np, pandas as pd, scipy.stats; csd = scipy.stats.circstd(phase_runs_h, high=24, low=0, axis=1); pd.DataFrame({'cell': cells, 'circular_sd_h': csd}).to_csv('stability_distributions.csv', index=False)
```

## Evaluation signals

- Tempo's median per-cell circular standard deviation is below 1 hour on all simulation settings and real datasets except the aorta endothelial cells
- Cyclops and Cyclum reproduce visibly unstable (larger-spread) per-cell circular SD distributions matching the published Fig. 2f/5d figures
- Per-cell phases were correctly wrapped to the 24 h cycle before computing circular SD (no boundary artifacts at 0/24 h)
- Exactly five replicate runs per method per dataset contributed to each per-cell SD value
- stability_distributions.csv is produced with per-cell circular SD values for every method-dataset combination

## Limitations

- Tempo's median circular SD exceeds 1 h on the aorta endothelial-cell dataset, the one real-data exception to the stability criterion
- Stability of point estimates does not by itself establish accuracy; the article separately reports existing methods yield poor phase estimates and do not quantify estimation uncertainty
- Run times of competing approaches (e.g., Cyclops, Cyclum) scale poorly with the number of cells, making five-run stability analysis untenable for large droplet-based datasets without sufficient compute

## Evidence

- [results] Tempo's phase point estimates are stable across repeated runs, with median circular standard deviation below 1 hour on all real datasets except the aorta endothelial cells: [evidence span withheld — anchor status: not_found]
- [methods] For every run, convert per-cell posterior phase estimates to hours over the 24 h circadian cycle and compute per-cell circular standard deviations across the five replicates.: [evidence span withheld — anchor status: not_found]
- [readme] Tempo requires python >= 3.8 and depends on the following python packages: "Tempo requires python >= 3.8 and depends on the following python packages"
- [readme] Successful completion of the test will yield a message "SUCCESSFULLY FINISHED": "Successful completion of the test will yield a message "SUCCESSFULLY FINISHED""
