---
name: phase-accuracy-benchmarking
description: Use when evaluating a circadian phase inference method (e.g. Tempo) on
  simulated scRNA-seq data with known true phases, to quantify within-3h phase accuracy
  and test whether de novo cyclers improve point estimates.
when_to_use_negative:
- Input is real unsynchronized scRNA-seq data with no simulated true phase, since
  the within-3 h accuracy metric cannot be computed
- You need general Tempo accuracy versus other methods on droplet data rather than
  a controlled waveform-realism ablation; use the paper's primary benchmark instead
- Simulation uses strictly pure 24 h sinusoidal CCG means, which removes the waveform-realism
  effect this benchmark is designed to expose
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Tempo
  role: Bayesian variational inference algorithm run in both core-clock-gene (supervised
    CCGs) and de novo cycler detection modes to produce phase estimates being benchmarked
  repo: https://github.com/bauerbach95/tempo
- name: anndata
  role: Container format for the simulated count matrix with true-phase annotations
- name: numpy
  role: Numerical computation of phase-distance (within-3 h) accuracy metrics in Python
- name: pandas
  role: Writes de novo cycler names and paired phase-accuracy metrics to simulation_fig3c_h_results.csv
- name: scanpy
  role: scRNA-seq preprocessing/analysis of the simulated anndata object (scanpy>=1.6)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:4f04396400a64acada7c1776086c503a88044e39ae8332d7ea501c69443a4f54
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# phase-accuracy-benchmarking

## Summary

Benchmark circadian phase inference accuracy on realistic-waveform single-cell simulations by measuring the fraction of cells whose inferred phase falls within 3 h of simulated truth, comparing a core-clock-gene baseline against de novo cycler detection.

## When to use

Use when evaluating a circadian phase inference method (e.g. Tempo) on simulated scRNA-seq data with known true phases, to quantify within-3h phase accuracy and test whether de novo cyclers improve point estimates.

## When NOT to use

- Input is real unsynchronized scRNA-seq data with no simulated true phase, since the within-3 h accuracy metric cannot be computed
- You need general Tempo accuracy versus other methods on droplet data rather than a controlled waveform-realism ablation; use the paper's primary benchmark instead
- Simulation uses strictly pure 24 h sinusoidal CCG means, which removes the waveform-realism effect this benchmark is designed to expose

## Inputs

- Simulated scRNA-seq count matrix with known true per-cell phases (anndata object; ~4800 cells, median 5000 UMIs/cell, realistic waveforms from Zhang et al. aorta microarray cyclers)
- Core circadian clock gene list (labeled CCGs)
- True-cycler gene list from JTKCycle q<0.05 on bulk aorta microarray
- Tempo phase estimates from core-clock-only and de novo cycler modes

## Outputs

- Within-3 h phase-accuracy fraction for the core-clock-gene baseline (~62%)
- De novo cycler gene list (~25 true-positive CCGs)
- Within-3 h phase-accuracy fraction using de novo cyclers (~72%)
- Paired metrics table, e.g. simulation_fig3c_h_results.csv

## How to apply

Construct a realistic-waveform simulation (e.g. 4800 cells at a median depth of 5000 UMIs) using JTKCycle q<0.05 genes from the Zhang et al. bulk aorta microarray time course as the true cycler set, stored in an anndata object. Run Tempo with only the core circadian clock genes as labeled CCGs to get baseline phase estimates and compute the fraction of cells within 3 h of true phase (~62% expected). Rerun Tempo in de novo cycler detection mode to identify ~25 unsupervised CCGs, cross-check each against the JTKCycle q<0.05 true-cycler list to confirm all are true positives, then recompute the within-3 h accuracy using the de novo cyclers (expected rise to ~72%). Persist the de novo cycler names and paired accuracy metrics (e.g. to simulation_fig3c_h_results.csv) so the two conditions are directly comparable.

## Related tools

- **Tempo** (Bayesian variational inference algorithm run in both core-clock-gene (supervised CCGs) and de novo cycler detection modes to produce phase estimates being benchmarked) — https://github.com/bauerbach95/tempo
- **anndata** (Container format for the simulated count matrix with true-phase annotations)
- **numpy** (Numerical computation of phase-distance (within-3 h) accuracy metrics in Python)
- **pandas** (Writes de novo cycler names and paired phase-accuracy metrics to simulation_fig3c_h_results.csv)
- **scanpy** (scRNA-seq preprocessing/analysis of the simulated anndata object (scanpy>=1.6))

## Examples

```
conda activate tempo; python run_test.py
```

## Evaluation signals

- Core-clock-only baseline within-3 h fraction lands near 62% on the 4800-cell realistic-waveform simulation
- De novo cycler mode recovers ~25 cyclers, every one of which appears in the JTKCycle q<0.05 true-cycler list (100% precision)
- Accuracy using de novo cyclers rises to ~72% (a ~10 point improvement over the core-clock baseline)
- Counts object matches simulation spec: 4800 cells at median depth 5000 UMIs drawn from aorta-microarray waveform cyclers
- Output CSV (simulation_fig3c_h_results.csv) contains both paired accuracy values and the de novo cycler names

## Limitations

- Tempo's phase inference assumes CCG expression means follow pure 24 h sinusoidal patterns, which the paper suggests explains degradation on realistic waveforms; more flexible waveform modeling is needed
- De novo cycler performance was validated here only on simulation with a trusted JTKCycle q<0.05 ground truth, not against experimentally validated single-cell phases
- Results depend on simulation parameters (cell number, depth, waveform source); accuracy values (62%/72%) are specific to this 4800-cell, 5000-UMI setup

## Evidence

- [other] Realistic-waveform baseline vs de novo accuracy: [evidence span withheld — anchor status: not_found]
- [other] Simulation construction parameters: [evidence span withheld — anchor status: not_found]
- [other] De novo cycler validation: [evidence span withheld — anchor status: not_found]
- [discussion] Waveform assumption limitation: "this may be due, in part, to the assumption that the expression means of CCGs follow 24 h sinusoidal patterns across the circadian cycle"
- [readme] Tempo install/test invocation: "To test if Tempo works properly, run the run_test.py file using using the activated environment:
python run_test.py"
