---
name: circadian-phase-stability-analysis
description: Use when benchmarking unsupervised circadian phase inference on scRNA-seq
  data and you need to quantify how stable per-cell phase point estimates are across
  repeated runs of competing methods, as in Fig. 2f/5d stability distributions.
when_to_use_negative:
- The analysis goal is phase accuracy against known ground truth (simulations with
  true phases) rather than run-to-run stability of point estimates
- The method under test provides only a single deterministic output or cannot be run
  repeatedly, so replicate circular SD is not defined
- Data are bulk time-course samples rather than single-cell data, where Tempo's per-cell
  uncertainty and stability framing does not apply
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3517
tools:
- name: Tempo
  role: Primary Bayesian variational phase inference method whose per-cell phase estimates
    are tested for run-to-run stability
  repo: https://github.com/bauerbach95/tempo
- name: Cyclops
  role: Competing phase inference method used as an unstable baseline in the five-run
    comparison
- name: Cyclum
  role: Competing phase inference method used as an unstable baseline in the five-run
    comparison
- name: scanpy>=1.6
  role: Loading and handling the scRNA-seq AnnData datasets for stability analysis
- name: pytorch>=1.9.0
  role: Deep-learning dependency of the Tempo package required by the conda environment
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:e8fa608bc7991bd9ad5ffa127b0a93b92967d4abda989acea1ad9a49511cb53f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bauerbach95/tempo
  tools:
  - Cyclops
  - Cyclum
  - Tempo
  - anndata
  - numpy
  - pandas
  - pytorch>=1.9.0
  - scanpy>=1.6
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# circadian-phase-stability-analysis

## Summary

Quantifies run-to-run stability of circadian phase inference by running Tempo, Cyclops, and Cyclum five times per scRNA-seq dataset, converting per-cell posterior phases to hours, and computing per-cell circular standard deviations across replicates. It reproduces the paper's finding that Tempo achieves median circular SD below 1 h while competing methods yield unstable phase estimates.

## When to use

Use when benchmarking unsupervised circadian phase inference on scRNA-seq data and you need to quantify how stable per-cell phase point estimates are across repeated runs of competing methods, as in Fig. 2f/5d stability distributions.

## When NOT to use

- The analysis goal is phase accuracy against known ground truth (simulations with true phases) rather than run-to-run stability of point estimates
- The method under test provides only a single deterministic output or cannot be run repeatedly, so replicate circular SD is not defined
- Data are bulk time-course samples rather than single-cell data, where Tempo's per-cell uncertainty and stability framing does not apply

## Inputs

- scRNA-seq datasets from Fig. 2f simulation settings and Fig. 5d real datasets (including the aorta endothelial-cell dataset), loaded as AnnData objects via anndata/scanpy
- Per-cell posterior phase estimates from each of five runs of Tempo, Cyclops, and Cyclum per dataset

## Outputs

- stability_distributions.csv containing per-cell circular standard deviations (in hours, over the 24 h cycle) across five replicates for each method and dataset
- Fig. 2f/5d-style stability distribution plots comparing circular SD spread across methods

## How to apply

Activate the tempo conda environment built from github.com/bauerbach95/tempo via install.sh (Python >= 3.8 with numpy, pandas, anndata, scanpy>=1.6, pytorch>=1.9.0), and verify installation by running run_test.py until it reports 'SUCCESSFULLY FINISHED'. Load each scRNA-seq dataset from Fig. 2f (simulation settings) and Fig. 5d (real datasets), including the aorta endothelial-cell dataset, using anndata/scanpy. Run Tempo, Cyclops, and Cyclum five times per dataset to capture run-to-run variability in phase inference. For each run, convert per-cell posterior phase estimates to hours over the 24 h circadian cycle, then compute per-cell circular standard deviations across the five replicates. Aggregate the circular SD distributions per method and dataset, save stability_distributions.csv, and render Fig. 2f/5d-style stability distribution plots.

## Related tools

- **Tempo** (Primary Bayesian variational phase inference method whose per-cell phase estimates are tested for run-to-run stability) — https://github.com/bauerbach95/tempo
- **Cyclops** (Competing phase inference method used as an unstable baseline in the five-run comparison)
- **Cyclum** (Competing phase inference method used as an unstable baseline in the five-run comparison)
- **scanpy>=1.6** (Loading and handling the scRNA-seq AnnData datasets for stability analysis)
- **pytorch>=1.9.0** (Deep-learning dependency of the Tempo package required by the conda environment)

## Examples

```
git clone https://github.com/bauerbach95/tempo; cd tempo; source install.sh; conda activate tempo; python run_test.py
```

## Evaluation signals

- Tempo reproduces a median per-cell circular standard deviation below 1 h across simulation settings and real datasets (Fig. 2f/5d), with the aorta endothelial cells as the only real-dataset exception
- Cyclops and Cyclum reproduce visibly unstable (larger-spread) circular SD distributions matching the published figures
- Per-cell phases in each run convert to hours over a 24 h circadian cycle with values wrapping correctly around 0/24 h
- stability_distributions.csv contains one row per cell per method per dataset with five-replicate circular SD values and no missing replicates
- Tempo installation is validated by run_test.py completing with the message 'SUCCESSFULLY FINISHED'

## Limitations

- Tempo's stability exception: median circular SD is not below 1 h on the aorta endothelial-cell real dataset
- Existing methods (Cyclops, Cyclum) yield poor point estimates and do not quantify estimation uncertainty, so their instability is expected rather than fixable via this workflow
- Simulated analyses suggest methods assuming 24 h sinusoidal CCG expression means may misbehave; more flexible waveform modeling may be needed for some datasets
- Explicit modeling of technical effects becomes necessary for data collected as multiple samples over time, which this stability workflow does not cover

## Evidence

- [methods] Tempo python/dependencies: "Tempo requires python >= 3.8 and depends on the following python packages"
- [methods] tempo repo: "git clone https://github.com/bauerbach95/tempo"
- [abstract] tempo abstract claim: "we demonstrate that Tempo yields more accurate estimates of circadian phase than existing methods"
- [discussion] tempo discussion: "we developed Tempo, a Bayesian algorithm for single-cell phase inference from scRNA-seq data"
- [readme] readme test: "Running the test should take less than 5 minutes. Successful completion of the test will yield a message "SUCCESSFULLY FINISHED"."
