---
name: circadian-cycler-de-novo-detection
description: 'Use when scRNA-seq counts lack labeled clock-controlled genes and you
  need unsupervised phase inference: run Tempo in de novo cycler-detection mode, cross-check
  calls against an independent rhythmicity test (e.g., JTKCycle q<0.05), and re-estimate
  phase.'
when_to_use_negative:
- When a curated, cell-type-specific list of clock-controlled genes is already available
  as labeled CCGs — supervised use of known CCGs may suffice and de novo detection
  is unnecessary
- When gene expression means follow strongly non-sinusoidal 24 h waveforms and flexible
  waveform modeling is required — Tempo's assumption of 24 h sinusoidal CCG expression
  means may bias results
- When the analysis requires explicit modeling of technical effects for data collected
  as multiple samples over time — Tempo does not explicitly model such effects
tools:
- name: Tempo
  role: Bayesian variational inference algorithm run in de novo cycler-detection mode
    to identify CCGs and infer circadian phase
  repo: github.com/bauerbach95/tempo
- name: anndata
  role: Container for the simulated/real single-cell counts matrix passed to Tempo
- name: scanpy
  role: Python package (>=1.6) dependency used with anndata for single-cell data handling
- name: numpy
  role: Numerical dependency for constructing the simulation and computing phase-accuracy
    fractions
- name: pandas
  role: Writing de novo cycler names and paired phase-accuracy metrics to simulation_fig3c_h_results.csv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:94fc78698a28d4965b846568b12fdf92cd132da4a8137742582546afa7f47a20
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - Tempo
  - anndata
  - numpy
  - pandas
  - scanpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# circadian-cycler-de-novo-detection

## Summary

Identify clock-controlled genes (CCGs) de novo with Tempo's unsupervised cycler-detection mode and incorporate them into circadian phase inference, avoiding reliance on the ~20 moderately expressed core clock genes alone.

## When to use

Use when scRNA-seq counts lack labeled clock-controlled genes and you need unsupervised phase inference: run Tempo in de novo cycler-detection mode, cross-check calls against an independent rhythmicity test (e.g., JTKCycle q<0.05), and re-estimate phase.

## When NOT to use

- When a curated, cell-type-specific list of clock-controlled genes is already available as labeled CCGs — supervised use of known CCGs may suffice and de novo detection is unnecessary
- When gene expression means follow strongly non-sinusoidal 24 h waveforms and flexible waveform modeling is required — Tempo's assumption of 24 h sinusoidal CCG expression means may bias results
- When the analysis requires explicit modeling of technical effects for data collected as multiple samples over time — Tempo does not explicitly model such effects

## Inputs

- Single-cell gene count matrix stored as an anndata object (e.g., 4800 cells at median depth 5000 UMIs built from Zhang et al. bulk aorta microarray waveforms)
- List of core circadian clock genes used as labeled CCGs
- Independent true-cycler reference list (JTKCycle q<0.05 genes) for validating de novo calls

## Outputs

- De novo cycler gene names
- Paired phase-accuracy metrics (fraction of cells within 3 h of true phase, e.g., core-clock-only ~62% vs de novo ~72%), written to simulation_fig3c_h_results.csv
- Per-cell circadian phase estimates from Tempo

## How to apply

Clone github.com/bauerbach95/tempo and install via install.sh, then activate the tempo conda environment (Python >= 3.8 with anndata, numpy, pandas, scanpy>=1.6, scikit-image, pytorch>=1.9.0). Build the counts as an anndata object (e.g., 4800 simulated cells at a median depth of 5000 UMIs, using the Zhang et al. bulk aorta microarray JTKCycle q<0.05 genes as true cyclers). First run Tempo with only the core circadian clock genes as labeled CCGs to obtain a baseline phase accuracy — on realistic-waveform simulations expect ~62% of cells within 3 h of true phase. Then rerun Tempo in de novo cycler-detection mode (expect ~25 detected cyclers) and verify every call against the JTKCycle q<0.05 true-cycler list to confirm no false positives. Finally, recompute the fraction of phase estimates within 3 h of truth using the de novo cyclers and record the paired metrics (expected improvement from ~62% to ~72%) in an output CSV such as simulation_fig3c_h_results.csv.

## Related tools

- **Tempo** (Bayesian variational inference algorithm run in de novo cycler-detection mode to identify CCGs and infer circadian phase) — github.com/bauerbach95/tempo
- **anndata** (Container for the simulated/real single-cell counts matrix passed to Tempo)
- **scanpy** (Python package (>=1.6) dependency used with anndata for single-cell data handling)
- **numpy** (Numerical dependency for constructing the simulation and computing phase-accuracy fractions)
- **pandas** (Writing de novo cycler names and paired phase-accuracy metrics to simulation_fig3c_h_results.csv)

## Examples

```
git clone https://github.com/bauerbach95/tempo; cd tempo; source install.sh; conda activate tempo; python run_test.py
```

## Evaluation signals

- All detected de novo cyclers appear in the JTKCycle q<0.05 true-cycler list (zero false positives)
- Core-clock-only baseline: ~62% of cells have phase estimates within 3 h of true phase
- With de novo cyclers: ~72% of cells within 3 h of true phase, an improvement over the baseline
- run_test.py completes with the message 'SUCCESSFULLY FINISHED' confirming a correct Tempo installation
- Output CSV simulation_fig3c_h_results.csv contains the de novo cycler names and both paired accuracy fractions

## Limitations

- Tempo assumes expression means of CCGs follow 24 h sinusoidal patterns across the circadian cycle, which may explain degraded performance on realistic (non-sinusoidal) waveforms; more flexible waveform modeling is needed
- The core circadian clock comprises only ~20 moderately expressed genes and cell-type-specific CCG identities are often unknown, motivating but also constraining de novo detection
- Tempo has been tested on macOS 10.14.5/12.2.1 and CentOS Linux 7, and requires mini-conda plus pytorch>=1.9.0; no explicit software version for Tempo is stated in the discussion

## Evidence

- [abstract] tempo_intro_variational: "we introduce Tempo, a Bayesian variational inference approach that incorporates domain knowledge of the clock and quantifies phase estimation uncertainty"
- [abstract] tempo_intro_existing_poor: "existing methods yield poor circadian phase estimates, and do not quantify estimation uncertainty"
- [discussion] tempo_discussion_sinusoidal: "this may be due, in part, to the assumption that the expression means of CCGs follow 24 h sinusoidal patterns across the circadian cycle"
- [readme] tempo_readme_python_req: "Tempo requires python >= 3.8 and depends on the following python packages"
- [readme] tempo_readme_install: "git clone https://github.com/bauerbach95/tempo"
