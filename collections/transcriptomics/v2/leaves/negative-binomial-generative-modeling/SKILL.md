---
name: negative-binomial-generative-modeling
description: Use when benchmarking circadian phase-inference methods on simulated
  single-cell RNA-seq data, or when choosing a count likelihood distribution for unsupervised
  phase inference from droplet scRNA-seq (e.g. 10X Genomics Chromium) data.
when_to_use_negative:
- Analysis requires explicitly modeling technical/batch effects for data collected
  as multiple samples over time — Tempo's model does not incorporate these
- The biology demands subpopulation-specific gene amplitude variation or non-sinusoidal
  CCG waveforms, since Tempo assumes 24 h sinusoidal CCG expression means
- The task is cell-cycle phase inference with existing tools on plate-based data (e.g.
  Fluidigm C1) rather than circadian phase inference from droplet-based data
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Tempo
  role: Provides the Negative Binomial generative model, simulation utilities, and
    Bayesian variational inference (PyTorch backend) used to simulate data and estimate
    phases
  repo: github.com/bauerbach95/tempo
- name: Python >= 3.8
  role: Runtime environment for Tempo and its simulation/evaluation pipeline
- name: anndata
  role: Container format for simulated gene-by-cell count matrices with ground-truth
    phase annotations
- name: numpy
  role: Numerical backend for generating simulated count matrices and computing phase
    errors
- name: pandas
  role: Tabulation of median phase errors per method and condition
- name: scanpy
  role: scRNA-seq preprocessing utilities (scanpy>=1.6) within the Tempo environment
- name: scikit-learn
  role: Baseline methods including PCA phase estimation on simulated matrices
- name: PyTorch
  role: Backend for Tempo's Bayesian variational inference (pytorch>=1.9.0)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:2105ea8db2e24f9696b71deb65b8c456b8ce36d3533949ebc14bbdca1fa92968
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyTorch
  - Python
  - Python >= 3.8
  - Tempo
  - anndata
  - numpy
  - pandas
  - scanpy
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7619047619047619
n_claims: 21
---

# negative-binomial-generative-modeling

## Summary

Use Tempo's Negative Binomial generative model of droplet-based scRNA-seq counts to simulate datasets with known ground-truth circadian phases, so that phase-estimation accuracy can be benchmarked against a theoretical optimum. This addresses the core limitation that existing unsupervised phase methods assume likelihood distributions that poorly approximate true droplet count generation.

## When to use

Use when benchmarking circadian phase-inference methods on simulated single-cell RNA-seq data, or when choosing a count likelihood distribution for unsupervised phase inference from droplet scRNA-seq (e.g. 10X Genomics Chromium) data.

## When NOT to use

- Analysis requires explicitly modeling technical/batch effects for data collected as multiple samples over time — Tempo's model does not incorporate these
- The biology demands subpopulation-specific gene amplitude variation or non-sinusoidal CCG waveforms, since Tempo assumes 24 h sinusoidal CCG expression means
- The task is cell-cycle phase inference with existing tools on plate-based data (e.g. Fluidigm C1) rather than circadian phase inference from droplet-based data

## Inputs

- Simulated scRNA-seq count matrices (anndata/numpy/pandas) generated from Tempo's Negative Binomial generative model with known ground-truth circadian phases
- Simulation grid specification: unsynchronized and ZT0/6/12/18 conditions, 500-5000 cells, 3000-20000 median UMI counts
- Core clock gene set (~20 moderately expressed genes) and/or full gene-by-cell count matrices

## Outputs

- Simulated count matrices with paired ground-truth phase labels per cell
- Per-cell point-estimate circadian phases from Tempo and baseline methods (Cyclops, Cyclum, PCA)
- Empirical cumulative distribution functions (eCDFs) of absolute phase errors per method and condition (reproducing Fig. 2a/d panels) and median error tables

## How to apply

Define a simulation grid over unsynchronized and ZT0/6/12/18 time-course conditions, spanning 500-5000 cells and 3000-20000 median UMIs. Generate count matrices from Tempo's Negative Binomial generative model using the package's simulation utilities (numpy/pandas/anndata), retaining the known ground-truth phases. Run Tempo via its Bayesian variational inference implementation (PyTorch backend) on core-clock-gene-restricted and all-gene inputs, and run baselines (Cyclops, Cyclum, PCA) on the same matrices. Compute absolute point-estimate phase errors against ground truth and compare error eCDFs: Tempo should lie near the maximum-likelihood optimum under the true generative model, with baselines degrading at small library sizes. Install via install.sh, activate the tempo conda environment, and first confirm the pipeline by running run_test.py, which must report 'SUCCESSFULLY FINISHED'.

## Related tools

- **Tempo** (Provides the Negative Binomial generative model, simulation utilities, and Bayesian variational inference (PyTorch backend) used to simulate data and estimate phases) — github.com/bauerbach95/tempo
- **Python >= 3.8** (Runtime environment for Tempo and its simulation/evaluation pipeline)
- **anndata** (Container format for simulated gene-by-cell count matrices with ground-truth phase annotations)
- **numpy** (Numerical backend for generating simulated count matrices and computing phase errors)
- **pandas** (Tabulation of median phase errors per method and condition)
- **scanpy** (scRNA-seq preprocessing utilities (scanpy>=1.6) within the Tempo environment)
- **scikit-learn** (Baseline methods including PCA phase estimation on simulated matrices)
- **PyTorch** (Backend for Tempo's Bayesian variational inference (pytorch>=1.9.0))

## Examples

```
git clone https://github.com/bauerbach95/tempo; cd tempo; source install.sh; conda activate tempo; python run_test.py
```

## Evaluation signals

- Packaged check run_test.py completes and prints 'SUCCESSFULLY FINISHED' (takes <5 minutes) before any benchmarking is trusted
- Tempo's point-estimate phase error eCDF lies only slightly above the maximum-likelihood optimum computed under the true generative model (Fig. 2a/d ordering reproduced)
- Median phase errors are tabulated per method and condition and confirm the reported method ordering across the 500-5000 cell and 3000-20000 UMI grid
- Baseline methods (Cyclops, Cyclum, PCA) show clearly worse error eCDFs than Tempo at small library sizes, matching the paper's finding that existing methods yield poor point estimates on droplet data
- Simulated counts are confirmed to come from Tempo's Negative Binomial generative model with ground-truth phases retained for error computation

## Limitations

- Assumption that CCG expression means follow 24 h sinusoidal patterns may not hold; simulated analyses suggest this causes issues for de novo cyclers, and more flexible waveform modeling is needed
- Technical effects are not explicitly modeled, which becomes necessary for data collected as multiple samples over time
- Subpopulation-specific gene amplitude variation is not explicitly modeled; a mapping from cell state embedding to sinusoidal parameters would be an ideal solution
- Many open questions in chronobiology (cell-type-specific CCG identification, phase gating of cell fate decisions) still require experimental pairing of clock reporters with transcriptomics, which is lacking

## Evidence

- [abstract] we introduce Tempo, a Bayesian variational inference approach that incorporates domain knowledge of the clock and quantifies phase estimation uncertainty: "we introduce Tempo, a Bayesian variational inference approach that incorporates domain knowledge of the clock and quantifies phase estimation uncertainty"
- [intro] existing approaches yield poor point estimates of cell phase due to transcript likelihood distribution choices that do not closely approximate the true generative distribution of droplet-based: "existing approaches yield poor point estimates of cell phase due to transcript likelihood distribution choices that do not closely approximate the true generative distribution of droplet-based"
- [methods] Tempo requires python >= 3.8 and depends on the following python packages: "Tempo requires python >= 3.8 and depends on the following python packages"
- [readme] Running the test should take less than 5 minutes. Successful completion of the test will yield a message "SUCCESSFULLY FINISHED".: "Running the test should take less than 5 minutes. Successful completion of the test will yield a message "SUCCESSFULLY FINISHED"."
