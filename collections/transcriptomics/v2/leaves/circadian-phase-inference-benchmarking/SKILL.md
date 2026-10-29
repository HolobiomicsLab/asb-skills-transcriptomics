---
name: circadian-phase-inference-benchmarking
description: Use when a single-cell RNA-seq dataset (droplet or plate based) has an
  unknown or partially known circadian phase per cell and you need to quantify how
  accurately different unsupervised phase-inference methods recover it.
when_to_use_negative:
- Input data is plate-based scRNA-seq from a time-course where sample timing can safely
  proxy cell phase and no method comparison is needed
- The goal is identifying cell-type-specific clock-controlled genes rather than evaluating
  circadian phase estimation accuracy
- The transcript likelihood distribution of the data does not approximate a droplet-based
  Negative Binomial generative model and no simulation matching is possible
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3168
tools:
- name: Tempo
  role: Primary Bayesian variational inference method whose phase estimates are benchmarked;
    also provides simulation utilities and packaged validation test
  repo: https://github.com/bauerbach95/tempo
- name: Cyclops
  role: Baseline unsupervised phase-inference method run on the same count matrices
    for comparison
- name: Cyclum
  role: Baseline unsupervised phase-inference method run on the same count matrices
    for comparison
- name: PCA
  role: Dimensionality-reduction baseline used to obtain competing phase estimates
    from the same data
- name: scanpy
  role: anndata handling and preprocessing of scRNA-seq inputs (scanpy>=1.6)
- name: PyTorch
  role: Backend for Tempo's Bayesian variational inference (pytorch>=1.9.0)
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:e8c70ebf184802b7491cbeced616001c626e855d00ab300888389fdab968c01e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bauerbach95/tempo
  tools:
  - Cyclops
  - Cyclum
  - PCA
  - PyTorch
  - Python
  - Tempo
  - anndata
  - numpy
  - pandas
  - scanpy
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# circadian-phase-inference-benchmarking

## Summary

Benchmark circadian phase inference from scRNA-seq by running Tempo (Bayesian variational inference) against Cyclops, Cyclum, and PCA on simulated and real droplet-based datasets, comparing per-cell phase point estimates to ground truth via error eCDFs, median errors, and Bayes factors.

## When to use

Use when a single-cell RNA-seq dataset (droplet or plate based) has an unknown or partially known circadian phase per cell and you need to quantify how accurately different unsupervised phase-inference methods recover it.

## When NOT to use

- Input data is plate-based scRNA-seq from a time-course where sample timing can safely proxy cell phase and no method comparison is needed
- The goal is identifying cell-type-specific clock-controlled genes rather than evaluating circadian phase estimation accuracy
- The transcript likelihood distribution of the data does not approximate a droplet-based Negative Binomial generative model and no simulation matching is possible

## Inputs

- Simulated scRNA-seq count matrices from Tempo's Negative Binomial generative model with known ground-truth circadian phases (unsynchronized and ZT0/6/12/18 conditions; 500–5000 cells; median UMI 3000–20000)
- 10X Chromium mouse aorta light-dark scRNA-seq dataset as an anndata object, subsettable to SMCs, fibroblasts, endothelial cells, and macrophages
- Droin et al. hepatocyte time-course scRNA-seq dataset as an anndata object
- Core circadian clock gene set (~20 genes) and full gene set inputs

## Outputs

- Per-cell phase estimates in hours for each method (Tempo, Cyclops, Cyclum, PCA)
- Empirical cumulative distribution functions of absolute phase error per method and condition (Fig. 2a/d-style plots)
- phase_benchmark_results.csv tabulating median phase errors and log Bayes factors per method-cell-type combination

## How to apply

Set up the tempo conda environment (git clone https://github.com/bauerbach95/tempo, source install.sh, conda activate tempo) and first run `python run_test.py`, accepting only if it prints 'SUCCESSFULLY FINISHED'. For simulation benchmarks, define a grid of unsynchronized and ZT0/6/12/18 time-course conditions with 500–5000 cells and median UMI counts of 3000–20000, generate count matrices from Tempo's Negative Binomial generative model with known ground-truth phases, and run Tempo (core-clock-gene-restricted and all-gene inputs, pytorch variational backend) alongside Cyclops, Cyclum, and PCA. For real-data benchmarks, load 10X Chromium mouse aorta light-dark and Droin et al. hepatocyte time-course anndata objects, subset the aorta data to SMCs, fibroblasts, endothelial cells, and macrophages, run all four methods per cell type, compute median absolute phase error against the known sampling-time reference, and compute Tempo-vs-baseline Bayes factors. Assemble median errors and log Bayes factors into phase_benchmark_results.csv and plot per-method error eCDFs (Fig. 2a/d style) to verify that Tempo's eCDF lies near the maximum-likelihood optimum while baselines degrade at small library sizes.

## Related tools

- **Tempo** (Primary Bayesian variational inference method whose phase estimates are benchmarked; also provides simulation utilities and packaged validation test) — https://github.com/bauerbach95/tempo
- **Cyclops** (Baseline unsupervised phase-inference method run on the same count matrices for comparison)
- **Cyclum** (Baseline unsupervised phase-inference method run on the same count matrices for comparison)
- **PCA** (Dimensionality-reduction baseline used to obtain competing phase estimates from the same data)
- **scanpy** (anndata handling and preprocessing of scRNA-seq inputs (scanpy>=1.6))
- **PyTorch** (Backend for Tempo's Bayesian variational inference (pytorch>=1.9.0))

## Examples

```
git clone https://github.com/bauerbach95/tempo && cd tempo && source install.sh && conda activate tempo && python run_test.py
```

## Evaluation signals

- Packaged smoke test must print 'SUCCESSFULLY FINISHED' before any benchmark run
- On simulated data, Tempo's phase error eCDF should lie only slightly worse than the maximum-likelihood optimum under the true generative model
- Reproduced median phase errors should match Table 1, e.g. Tempo all-genes aorta SMCs = 1.88 h
- Bayes factors should reproduce reported orders of magnitude: 10^17896 (SMCs), 10^2257 (fibroblasts), 10^110 (endothelial cells), 10^184 (macrophages), 10^1761 (hepatocytes)
- Baseline methods should show degraded error eCDFs at small library sizes (median UMI near 3000) relative to Tempo

## Limitations

- Tempo assumes CCG expression means follow 24 h sinusoidal patterns, which may drive de novo cycler issues; more flexible waveform modeling is needed
- Explicit modeling of technical effects is not included and becomes necessary for data collected as multiple samples over time
- Subpopulation-specific gene amplitude variation is not explicitly modeled; a mapping from cell-state embedding to sinusoidal parameters would be ideal
- Tempo requires python >= 3.8 with scanpy>=1.6 and pytorch>=1.9.0; tested only on macOS 10.14.5/12.2.1 and CentOS Linux 7

## Evidence

- [readme] README installation and validation: "Running the test should take less than 5 minutes. Successful completion of the test will yield a message "SUCCESSFULLY FINISHED"."
- [readme] README installation commands: "git clone https://github.com/bauerbach95/tempo"
- [abstract] Abstract claim: "we demonstrate that Tempo yields more accurate estimates of circadian phase than existing methods"
- [methods] Python requirement: "Tempo requires python >= 3.8 and depends on the following python packages"
- [intro] Motivation for simulation-based evaluation: "existing approaches yield poor point estimates of cell phase due to transcript likelihood distribution choices that do not closely approximate the true generative distribution of droplet-based"
- [intro] Uncertainty motivation: "existing approaches do not quantify the uncertainty of phase estimates. This becomes crucial for interpretation of results from very sparse droplet-based scRNA-seq data."
- [discussion] Sinusoidal assumption caveat: "this may be due, in part, to the assumption that the expression means of CCGs follow 24 h sinusoidal patterns across the circadian cycle"
- [discussion] Technical effect caveat: "While less important for applications to single-sample unsynchronized scRNA-seq data, this becomes more necessary for data collected as multiple samples over time."
