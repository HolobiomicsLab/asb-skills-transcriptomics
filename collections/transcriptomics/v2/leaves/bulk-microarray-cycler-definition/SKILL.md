---
name: bulk-microarray-cycler-definition
description: Use when building simulated circadian scRNA-seq data that needs a realistic,
  non-sinusoidal set of true cyclers, or when validating unsupervised cycler detections
  (e.g., Tempo de novo cyclers) against a bulk microarray-derived ground truth.
when_to_use_negative:
- When a cell-type-specific ground-truth cycler list is already available from the
  target single-cell tissue — bulk aorta waveforms may not transfer
- When the benchmark only requires canonical sinusoidal waveforms and the ~20 core
  circadian clock genes suffice
- When the input is real unsynchronized scRNA-seq without any time course to support
  rhythmicity testing
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: Tempo
  role: Unsupervised Bayesian phase inference; run in de novo cycler detection mode
    on the simulated counts to identify clock-controlled genes for validation against
    the JTKCycle list
  repo: github.com/bauerbach95/tempo
- name: anndata
  role: Stores the realistic-waveform simulated counts as an anndata object consumed
    by Tempo
- name: numpy
  role: Numerical construction of the simulated count matrix
- name: pandas
  role: Writes de novo cycler names and paired phase-accuracy metrics to simulation_fig3c_h_results.csv
- name: scanpy
  role: Single-cell preprocessing of the simulated anndata counts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:647774c0243c62c98bfdbb777e37269c009b2e59b2c3d5dae360468342d63881
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
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# bulk-microarray-cycler-definition

## Summary

Defines the set of 'true cyclers' for realistic circadian simulation benchmarks by running JTKCycle on the Zhang et al. bulk aorta microarray time course and keeping genes with q<0.05, giving a bulk-derived CCG ground-truth list used to build simulated scRNA-seq counts and to verify de novo cycler calls.

## When to use

Use when building simulated circadian scRNA-seq data that needs a realistic, non-sinusoidal set of true cyclers, or when validating unsupervised cycler detections (e.g., Tempo de novo cyclers) against a bulk microarray-derived ground truth.

## When NOT to use

- When a cell-type-specific ground-truth cycler list is already available from the target single-cell tissue — bulk aorta waveforms may not transfer
- When the benchmark only requires canonical sinusoidal waveforms and the ~20 core circadian clock genes suffice
- When the input is real unsynchronized scRNA-seq without any time course to support rhythmicity testing

## Inputs

- Zhang et al. bulk aorta microarray time-course expression matrix
- Simulated single-cell counts in an anndata object (e.g., 4800 cells, median depth 5000 UMIs)
- De novo cycler calls from Tempo (e.g., ~25 detected genes)

## Outputs

- JTKCycle q<0.05 true-cycler gene list from the aorta microarray
- Realistic-waveform simulation counts (anndata) built from the true cyclers
- simulation_fig3c_h_results.csv with de novo cycler names and paired phase-accuracy metrics (within-3 h fractions)

## How to apply

Take the Zhang et al. bulk aorta microarray time course and run JTKCycle on it, retaining all genes with q<0.05 as the definitive list of circadianly expressed genes. Use these waveforms (which are more realistic than pure 24 h sinusoids) to construct the simulated expression means, e.g., for a 4800-cell simulation at a median depth of 5000 UMIs stored in an anndata object. After running an unsupervised method such as Tempo in de novo cycler mode (expected ~25 calls), cross-check every detected cycler against the JTKCycle q<0.05 list to confirm all calls are true positives, and write the cycler names plus paired phase-accuracy metrics (e.g., within-3 h fractions, 62% core-clock-only vs 72% with de novo cyclers) to an output CSV such as simulation_fig3c_h_results.csv.

## Related tools

- **Tempo** (Unsupervised Bayesian phase inference; run in de novo cycler detection mode on the simulated counts to identify clock-controlled genes for validation against the JTKCycle list) — github.com/bauerbach95/tempo
- **anndata** (Stores the realistic-waveform simulated counts as an anndata object consumed by Tempo)
- **numpy** (Numerical construction of the simulated count matrix)
- **pandas** (Writes de novo cycler names and paired phase-accuracy metrics to simulation_fig3c_h_results.csv)
- **scanpy** (Single-cell preprocessing of the simulated anndata counts)

## Examples

```
conda activate tempo; python run_tempo.py --mode de_novo --input aorta_sim_4800cells.h5ad; cross-check outputs against JTKCycle_q005_aorta.csv with pandas
```

## Evaluation signals

- All JTKCycle q<0.05 genes from the aorta microarray are retained as the true-cycler set (and nothing with q>=0.05 is included)
- Every de novo cycler called by Tempo appears in the JTKCycle q<0.05 true-cycler list (100% true-positive rate, expected ~25 calls)
- Baseline Tempo phase accuracy with core clock genes only is ~62% of cells within 3 h of true phase; incorporating de novo cyclers raises this to ~72%
- Simulation parameters match the benchmark: 4800 cells at a median depth of 5000 UMIs
- simulation_fig3c_h_results.csv contains paired accuracy metrics (core-only vs de novo) and the de novo cycler names

## Limitations

- Tempo's generative model assumes CCG expression means follow pure 24 h sinusoidal patterns; realistic bulk-derived waveforms expose this mismatch, so simulation accuracy underestimates performance on true sinusoidal data
- Cycler calls are defined against a bulk aorta microarray truth, which may not reflect cell-type-specific CCG expression in the simulated single-cell tissue
- Tempo requires python >= 3.8 and >=8GB RAM for in-memory operations

## Evidence

- [other] JTKCycle q<0.05 genes from the Zhang et al. bulk aorta microarray as the set of true cyclers: [evidence span withheld — anchor status: not_found]
- [other] Simulation scale and output: [evidence span withheld — anchor status: not_found]
- [results] Accuracy improvement with de novo cyclers: [evidence span withheld — anchor status: not_found]
- [results] De novo cycler assumption: [evidence span withheld — anchor status: not_found]
- [readme] README dependencies: "Tempo requires python >= 3.8 and depends on the following python packages:
  - anndata
  - numpy
  - pandas
  - scanpy>=1.6"
