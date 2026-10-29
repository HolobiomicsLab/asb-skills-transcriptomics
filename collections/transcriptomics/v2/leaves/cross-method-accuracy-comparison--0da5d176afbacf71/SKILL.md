---
name: cross-method-accuracy-comparison
description: Use when a new method claims improved accuracy on a task (e.g., circadian
  phase inference from scRNA-seq) and you must quantify how close its errors are to
  a theoretical optimum versus baseline methods on simulated data with ground truth.
when_to_use_negative:
- The task is real (non-simulated) data with no ground-truth phases, since this skill's
  validation relies on comparing to simulated ground truth rather than reporter-based
  validation
- The method under test does not produce per-cell point estimates comparable to the
  baselines' outputs (e.g., only uncertainty summaries without phase estimates)
- Goal is uncertainty calibration assessment rather than point-estimate accuracy —
  Tempo's separate calibration property would need a different evaluation design
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: Tempo
  role: 'Method under test: Bayesian variational inference for circadian phase inference,
    run on simulated count matrices with PyTorch backend; also provides simulation
    utilities generating Negative Binomial count matrices with known ground-truth
    phases'
  repo: https://github.com/bauerbach95/tempo
- name: Cyclops
  role: 'Baseline method: provides competing circadian phase estimates on the same
    simulated count matrices'
- name: Cyclum
  role: 'Baseline method: provides competing phase estimates on the same simulated
    count matrices'
- name: PCA
  role: Baseline dimensionality-reduction approach used to obtain competing phase
    estimates on the simulated data
- name: Python >= 3.8
  role: Runtime for Tempo and the benchmark pipeline, with numpy, pandas, anndata,
    scanpy>=1.6, scikit-learn, scipy, statsmodels, and pytorch>=1.9.0 dependencies
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:78a9fbe1a65c1ea2fb418d781fc93282253185bb8833946cae89296828fbd0bf
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
  - Python >= 3.8
  - Tempo
  - anndata
  - numpy
  - pandas
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5294117647058824
n_claims: 17
---

# cross-method-accuracy-comparison

## Summary

Benchmark a new computational method (Tempo) against competing baselines (Cyclops, Cyclum, PCA) on simulated datasets with known ground truth, comparing phase point-estimate error distributions (eCDFs) and median errors.

## When to use

Use when a new method claims improved accuracy on a task (e.g., circadian phase inference from scRNA-seq) and you must quantify how close its errors are to a theoretical optimum versus baseline methods on simulated data with ground truth.

## When NOT to use

- The task is real (non-simulated) data with no ground-truth phases, since this skill's validation relies on comparing to simulated ground truth rather than reporter-based validation
- The method under test does not produce per-cell point estimates comparable to the baselines' outputs (e.g., only uncertainty summaries without phase estimates)
- Goal is uncertainty calibration assessment rather than point-estimate accuracy — Tempo's separate calibration property would need a different evaluation design

## Inputs

- Simulated scRNA-seq count matrices from Tempo's Negative Binomial generative model with known ground-truth circadian phases (numpy/pandas/anndata objects)
- Simulation grid of conditions: unsynchronized and ZT0/6/12/18 time-course conditions, cell counts 500-5000, median UMI counts 3000-20000
- Core-clock-gene-restricted and all-gene gene-expression inputs

## Outputs

- Absolute point-estimate phase errors per method and condition
- Empirical cumulative distribution function (eCDF) plots of phase error per method and condition (Fig. 2a/d-style panels)
- Table of median phase errors verifying method ordering (Tempo near the theoretical optimum, baselines degrading at small library sizes)

## How to apply

Define a simulation grid spanning biologically relevant conditions — e.g., unsynchronized and ZT0/6/12/18 time-course conditions, 500-5000 cells, and median UMI counts of 3000-20000 — and generate count matrices from the method's generative model with known ground-truth phases (Tempo uses a Negative Binomial generative model via its simulation utilities). Run the method under test (Tempo's Bayesian variational inference, PyTorch backend, on core-clock-gene-restricted and all-gene inputs) and each baseline (Cyclops, Cyclum, PCA) on identical simulated matrices. Compute absolute point-estimate phase errors against ground truth for every method and condition. Plot empirical cumulative distribution functions (eCDFs) of phase errors per method and condition (reproducing Fig. 2a/d panels) and tabulate median errors to verify the reported ordering. Before benchmarking, confirm the software installation by running the packaged test, which must report 'SUCCESSFULLY FINISHED'.

## Related tools

- **Tempo** (Method under test: Bayesian variational inference for circadian phase inference, run on simulated count matrices with PyTorch backend; also provides simulation utilities generating Negative Binomial count matrices with known ground-truth phases) — https://github.com/bauerbach95/tempo
- **Cyclops** (Baseline method: provides competing circadian phase estimates on the same simulated count matrices)
- **Cyclum** (Baseline method: provides competing phase estimates on the same simulated count matrices)
- **PCA** (Baseline dimensionality-reduction approach used to obtain competing phase estimates on the simulated data)
- **Python >= 3.8** (Runtime for Tempo and the benchmark pipeline, with numpy, pandas, anndata, scanpy>=1.6, scikit-learn, scipy, statsmodels, and pytorch>=1.9.0 dependencies)

## Examples

```
git clone https://github.com/bauerbach95/tempo; cd tempo; source install.sh; conda activate tempo; python run_test.py
```

## Evaluation signals

- Packaged test run_test.py completes with the message 'SUCCESSFULLY FINISHED' before benchmarking
- Tempo's phase-error eCDF lies near the theoretical optimum (maximum-likelihood phases under the true generative model) across simulation conditions
- Median phase-error table reproduces the reported ordering: Tempo more accurate than Cyclops, Cyclum, and PCA
- Baselines show error distributions that degrade specifically at small library sizes (low median UMI counts), consistent with the reported limitation of existing methods on droplet-based scRNA-seq
- Errors are computed as absolute phase deviation from the simulated ground-truth phases for every method and condition in the grid (unsynchronized plus ZT0/6/12/18)

## Limitations

- The benchmark's 'optimum' is defined under Tempo's own Negative Binomial generative model, so simulated-analyses conclusions may partly reflect generative-model assumptions (e.g., the 24 h sinusoidal CCG mean assumption flagged in the discussion of de novo cycler issues)
- Existing approaches' poor performance is partly attributed to transcript likelihood distribution choices poorly approximating droplet-based scRNA-seq; conclusions may not transfer to plate-based or other likelihood regimes without re-simulating
- Explicit modeling of technical effects over multiple samples over time and subpopulation-specific gene amplitude variation are not handled, which can affect real-data extension of the comparison

## Evidence

- [other] On simulated scRNA-seq datasets, Tempo's cell phase point estimate error eCDFs are only slightly worse than the optimum: [evidence span withheld — anchor status: not_found]
- [abstract] we demonstrate that Tempo yields more accurate estimates of circadian phase than existing methods: "we demonstrate that Tempo yields more accurate estimates of circadian phase than existing methods"
- [intro] existing approaches yield poor point estimates of cell phase due to transcript likelihood distribution choices that do not closely approximate the true generative distribution of droplet-based: "existing approaches yield poor point estimates of cell phase due to transcript likelihood distribution choices that do not closely approximate the true generative distribution of droplet-based"
- [methods] Tempo requires python >= 3.8 and depends on the following python packages: "Tempo requires python >= 3.8 and depends on the following python packages"
- [readme] Successful completion of the test will yield a message "SUCCESSFULLY FINISHED".: "Successful completion of the test will yield a message "SUCCESSFULLY FINISHED"."
- [discussion] this may be due, in part, to the assumption that the expression means of CCGs follow 24 h sinusoidal patterns across the circadian cycle: "this may be due, in part, to the assumption that the expression means of CCGs follow 24 h sinusoidal patterns across the circadian cycle"
