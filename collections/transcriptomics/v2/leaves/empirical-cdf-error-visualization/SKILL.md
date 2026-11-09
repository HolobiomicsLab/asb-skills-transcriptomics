---
name: empirical-cdf-error-visualization
description: Use when you have simulated scRNA-seq count matrices with known ground-truth
  circadian phases and multiple phase-inference methods (e.g., Tempo vs Cyclops/Cyclum/PCA)
  whose point-estimate errors you need to compare across conditions.
when_to_use_negative:
- Real scRNA-seq data without ground-truth or simulated phases, where absolute error
  against truth cannot be computed
- The goal is phase point estimation or uncertainty quantification itself (run Tempo)
  rather than benchmarking/visualizing error distributions across methods
- Data involve non-circadian oscillations where the 24 h sinusoidal mean assumption
  of the generative model does not hold
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Tempo
  role: Bayesian variational inference method whose phase point estimates are benchmarked
    against the optimum and baselines via error eCDFs
  repo: https://github.com/bauerbach95/tempo
- name: Python
  role: Implementation environment (>= 3.8) for simulating data, computing phase errors,
    and plotting eCDFs with numpy/pandas/matplotlib
- name: scanpy
  role: Single-cell data handling (scanpy>=1.6 dependency) for anndata-based count
    matrices used in the simulation grid
- name: scikit-learn
  role: Supports baseline (PCA) phase estimates computed on the same simulated count
    matrices
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:f2b2235f76421d44ca8da77121884177fb1188cc6ac35915621c41ea8040ef90
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bauerbach95/tempo
  tools:
  - PyTorch
  - Python
  - Tempo
  - anndata
  - numpy
  - pandas
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# Empirical CDF error visualization

## Summary

Benchmark circadian phase inference by computing absolute point-estimate phase errors against simulated ground truth and comparing methods via empirical cumulative distribution functions (eCDFs) of these errors, as done to show Tempo's estimates sit near the maximum-likelihood optimum while Cyclops, Cyclum, and PCA degrade.

## When to use

Use when you have simulated scRNA-seq count matrices with known ground-truth circadian phases and multiple phase-inference methods (e.g., Tempo vs Cyclops/Cyclum/PCA) whose point-estimate errors you need to compare across conditions.

## When NOT to use

- Real scRNA-seq data without ground-truth or simulated phases, where absolute error against truth cannot be computed
- The goal is phase point estimation or uncertainty quantification itself (run Tempo) rather than benchmarking/visualizing error distributions across methods
- Data involve non-circadian oscillations where the 24 h sinusoidal mean assumption of the generative model does not hold

## Inputs

- Simulated scRNA-seq count matrices (anndata objects, numpy/pandas) with ground-truth circadian phases per cell
- Per-cell point-estimate phases from each method (Tempo, Cyclops, Cyclum, PCA)
- Theoretical optimum: maximum-likelihood phases under the true generative model

## Outputs

- Empirical cumulative distribution function plots of absolute phase error per method and condition (Fig. 2a/d-style panels)
- Table of median phase errors per method and condition

## How to apply

Simulate count matrices under a Negative Binomial generative model with known ground-truth phases across a grid of conditions (unsynchronized and ZT0/6/12/18 time courses; 500–5000 cells; 3000–20000 median UMIs). Run each phase-inference method — Tempo via its Bayesian variational inference (PyTorch backend) on core-clock-gene-restricted and all-gene inputs, plus baselines Cyclops, Cyclum, and PCA — on the same matrices. Compute absolute point-estimate phase errors for every cell, method, and condition, then plot per-method, per-condition eCDFs of these errors (reproducing Fig. 2a/d panels) alongside tabulated median errors. Judge correctness by verifying Tempo's eCDF lies only slightly worse than the theoretical optimum (maximum-likelihood phases under the true generative model) and that baselines degrade at small library sizes.

## Related tools

- **Tempo** (Bayesian variational inference method whose phase point estimates are benchmarked against the optimum and baselines via error eCDFs) — https://github.com/bauerbach95/tempo
- **Python** (Implementation environment (>= 3.8) for simulating data, computing phase errors, and plotting eCDFs with numpy/pandas/matplotlib)
- **scanpy** (Single-cell data handling (scanpy>=1.6 dependency) for anndata-based count matrices used in the simulation grid)
- **scikit-learn** (Supports baseline (PCA) phase estimates computed on the same simulated count matrices)

## Examples

```
git clone https://github.com/bauerbach95/tempo; cd tempo; source install.sh; conda activate tempo; python run_test.py
```

## Evaluation signals

- Run the packaged validation test first; successful completion must report 'SUCCESSFULLY FINISHED'
- Tempo's phase error eCDF lies only slightly to the right of the maximum-likelihood optimum eCDF under the true generative model
- Baseline methods (Cyclops, Cyclum, PCA) show eCDFs degrading (larger errors) at small library sizes and low cell counts
- Tabulated median errors reproduce the reported ordering: Tempo more accurate than existing methods
- eCDF curves are computed per condition across the full grid (unsynchronized plus ZT0/6/12/18; 500–5000 cells; 3000–20000 median UMIs)

## Limitations

- Existing methods yield poor point estimates of cell phase on droplet-based scRNA-seq, particularly at sparse library sizes, which the eCDF comparison is designed to expose rather than fix
- Tempo's generative model assumes 24 h sinusoidal CCG expression means; the de novo cycler issue may stem from this assumption and more flexible waveform modeling may be needed
- Comparison requires ground truth (simulation); on real data no absolute error eCDF against truth can be drawn

## Evidence

- [abstract] Accuracy benchmark vs optimum: "we demonstrate that Tempo yields more accurate estimates of circadian phase than existing methods"
- [intro] Limitation driving baseline degradation: "existing approaches yield poor point estimates of cell phase due to transcript likelihood distribution choices that do not closely approximate the true generative distribution of droplet-based"
- [methods] Workflow: install and validate before benchmarking: "git clone https://github.com/bauerbach95/tempo"
- [discussion] Model assumption caveat affecting error interpretation: "this may be due, in part, to the assumption that the expression means of CCGs follow 24 h sinusoidal patterns across the circadian cycle"
- [readme] Packaged test success criterion: "Successful completion of the test will yield a message "SUCCESSFULLY FINISHED"."
