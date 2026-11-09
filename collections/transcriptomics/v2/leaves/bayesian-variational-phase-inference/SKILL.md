---
name: bayesian-variational-phase-inference
description: Use when scRNA-seq counts (e.g., droplet-based 10X data stored as anndata)
  must be ordered over the circadian cycle without sample-time labels, and phase point
  estimates plus uncertainty are needed.
when_to_use_negative:
- Data were collected as multiple samples over time where explicit modeling of technical
  effects is necessary — Tempo does not explicitly model these technical effects.
- The analysis requires cell-type-specific or subpopulation-specific CCG amplitude
  modeling — Tempo assumes 24 h sinusoidal CCG expression means and does not map cell
  state embeddings to sinusoidal parameters.
- The goal is rhythmicity detection in bulk time-course data (e.g., JTKCycle on microarray)
  rather than single-cell circadian phase inference.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Tempo
  role: Bayesian variational inference algorithm that performs the circadian phase
    inference and de novo CCG detection
  repo: https://github.com/bauerbach95/tempo
- name: anndata
  role: Container format for the single-cell count matrix passed to Tempo
- name: scanpy>=1.6
  role: Python dependency for single-cell preprocessing and handling alongside Tempo
- name: pandas
  role: Writing de novo cycler names and paired phase-accuracy metrics to CSV outputs
- name: numpy
  role: Numerical computation supporting the variational inference implementation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:d4f1b08c03cea7a090b789a4453be226bb226a85987ab2ef04ea9ec8e1a686bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bauerbach95/tempo
  tools:
  - Python
  - Tempo
  - anndata
  - numpy
  - pandas
  - scanpy
  - scanpy>=1.6
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# bayesian-variational-phase-inference

## Summary

Apply Tempo, a Bayesian variational inference algorithm, to infer single-cell circadian phase from scRNA-seq count data, incorporating core clock genes or de novo-identified clock-controlled genes (CCGs). It supports unsupervised phase inference for droplet- and plate-based data with calibrated uncertainty quantification.

## When to use

Use when scRNA-seq counts (e.g., droplet-based 10X data stored as anndata) must be ordered over the circadian cycle without sample-time labels, and phase point estimates plus uncertainty are needed.

## When NOT to use

- Data were collected as multiple samples over time where explicit modeling of technical effects is necessary — Tempo does not explicitly model these technical effects.
- The analysis requires cell-type-specific or subpopulation-specific CCG amplitude modeling — Tempo assumes 24 h sinusoidal CCG expression means and does not map cell state embeddings to sinusoidal parameters.
- The goal is rhythmicity detection in bulk time-course data (e.g., JTKCycle on microarray) rather than single-cell circadian phase inference.

## Inputs

- Single-cell RNA-seq count matrix stored as an anndata object (droplet- or plate-based, e.g., 10X Genomics Chromium data)
- Optional set of labeled core circadian clock genes (CCGs) for supervised mode
- Optional true-cycler list (e.g., JTKCycle q<0.05 genes from bulk microarray time course) for simulation validation

## Outputs

- Per-cell circadian phase point estimates
- Phase estimation uncertainty quantifications
- De novo cycler (CCG) gene names in unsupervised mode (e.g., written with phase-accuracy metrics to simulation_fig3c_h_results.csv)

## How to apply

Install Tempo from github.com/bauerbach95/tempo via git clone and source install.sh (requires mini-conda, python >= 3.8, pytorch>=1.9.0, scanpy>=1.6), then activate the tempo conda environment. Provide counts as an anndata object and run Tempo either supervised with core circadian clock genes labeled as CCGs (~20 moderately expressed genes), or in de novo cycler detection mode to identify CCGs without supervision. On realistic-waveform simulations (e.g., 4800 cells at median depth 5000 UMIs, using JTKCycle q<0.05 genes from Zhang et al. bulk aorta microarray as true cyclers), augmenting core-clock-only inference with 25 de novo cyclers raised the fraction of cells whose phase estimate was within 3 h of truth from 62% to 72%. Cross-check every de novo cycler against an independent rhythmicity call set (e.g., JTKCycle q<0.05) to confirm true positives before trusting phase improvements. Judge success by phase-accuracy fractions (within-3 h of known truth in simulations), calibration of uncertainty estimates, and reasonable run time on droplet-scale data.

## Related tools

- **Tempo** (Bayesian variational inference algorithm that performs the circadian phase inference and de novo CCG detection) — https://github.com/bauerbach95/tempo
- **anndata** (Container format for the single-cell count matrix passed to Tempo)
- **scanpy>=1.6** (Python dependency for single-cell preprocessing and handling alongside Tempo)
- **pandas** (Writing de novo cycler names and paired phase-accuracy metrics to CSV outputs)
- **numpy** (Numerical computation supporting the variational inference implementation)

## Examples

```
git clone https://github.com/bauerbach95/tempo; cd tempo; source install.sh; conda activate tempo; python run_test.py
```

## Evaluation signals

- On realistic-waveform simulations with known truth, at least ~62% of cells should have phase estimates within 3 h using core clock genes alone, improving to ~72% when de novo cyclers are added.
- In de novo mode, every detected cycler (e.g., 25 expected genes) should be cross-checked against the independent JTKCycle q<0.05 true-cycler list and all calls must be true positives.
- The run_test.py validation script completes with the message 'SUCCESSFULLY FINISHED'.
- Uncertainty quantifications should be well-calibrated against known simulated phases.
- Output metrics (paired within-3 h accuracy fractions for core-clock-only vs. de novo modes) are written to simulation_fig3c_h_results.csv.

## Limitations

- Tempo assumes CCG expression means follow 24 h sinusoidal patterns across the circadian cycle, which degrades performance on more realistic (non-sinusoidal) waveforms; more flexible waveform modeling is needed.
- Run time and technical-effect modeling: explicit modeling of technical effects becomes necessary for data collected as multiple samples over time, which Tempo does not currently provide.
- Subpopulation-specific gene amplitude variation is not explicitly modeled; a mapping from cell state embedding to sinusoidal parameters would be an ideal solution.
- Requires python >= 3.8, pytorch >= 1.9.0, and mini-conda for installation; >=8GB RAM is suggested for most datasets.

## Evidence

- [abstract] Tempo: a Bayesian variational inference approach that incorporates domain knowledge of the clock and quantifies phase estimation uncertainty: "we introduce Tempo, a Bayesian variational inference approach that incorporates domain knowledge of the clock and quantifies phase estimation uncertainty"
- [intro] de novo cycler identification rationale: "existing unsupervised phase inference methods perform poorly when tasked with ordering cells over the circadian cycle."
- [intro] core clock gene scale for CCG supervision: [evidence span withheld — anchor status: not_found]
- [other] workflow for supervised vs de novo phase accuracy: [evidence span withheld — anchor status: not_found]
- [other] phase accuracy metric definition: [evidence span withheld — anchor status: not_found]
- [readme] installation procedure: "1) git clone https://github.com/bauerbach95/tempo 2) cd tempo 3) source install.sh"
- [readme] validation test signal: "To test if Tempo works properly, run the run_test.py file using using the activated environment: python run_test.py"
- [discussion] sinusoidal mean assumption limitation: "this may be due, in part, to the assumption that the expression means of CCGs follow 24 h sinusoidal patterns across the circadian cycle"
