---
name: matrix-exponential-expression-dynamics-computation
description: Use when validating an inferred gene regulatory network (A matrix) by
  generating synthetic expression data — sample starting states x(0), propagate through
  the linear dynamics at known time points, add Gaussian noise (e.g. SD=5), and refit.
when_to_use_negative:
- The goal is biological interpretation of velocity or regulators on real data (e.g.
  acinar-to-ductal transdifferentiation) rather than ground-truth recovery benchmarking
- The dynamical model is nonlinear, so no closed-form propagator x(t)=e^{At}x(0) exists
- No fitted A matrix or ground-truth network is available to compare against
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2258
tools:
- name: scKINETICS
  role: Dynamical model whose fitted A matrix seeds the simulation and whose expectation-maximization
    fitting refits the recovered network from simulated data
  repo: github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Notebook environment for running the simulation, matrix-exponential propagation,
    and correlation analysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:cb79e09b2448cc7e4f5cc2c15fcd20f03204017466f54ae7c753f6dadb8dba4f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# matrix-exponential-expression-dynamics-computation

## Summary

Simulates single-cell expression trajectories under scKINETICS's linear dynamical model by propagating sampled starting states through x(t)=e^{At}x(0) with added Gaussian noise, enabling simulation-based validation that the EM-fitted gene regulatory network A recovers the true network.

## When to use

Use when validating an inferred gene regulatory network (A matrix) by generating synthetic expression data — sample starting states x(0), propagate through the linear dynamics at known time points, add Gaussian noise (e.g. SD=5), and refit.

## When NOT to use

- The goal is biological interpretation of velocity or regulators on real data (e.g. acinar-to-ductal transdifferentiation) rather than ground-truth recovery benchmarking
- The dynamical model is nonlinear, so no closed-form propagator x(t)=e^{At}x(0) exists
- No fitted A matrix or ground-truth network is available to compare against

## Inputs

- Fitted gene regulatory network matrix A from a scKINETICS EM fit
- Starting cell expression states x(0) sampled from single-cell expression data
- Known simulation time points
- Noise specification (Gaussian, SD = 5 in the benchmark)

## Outputs

- Simulated expression matrices with Gaussian noise (one per experiment)
- Refitted A matrices from scKINETICS EM on each simulated dataset
- Per-experiment Pearson correlations of inferred vs true A over nonzero elements
- network_recovery_correlations.csv with per-experiment and mean correlation values

## How to apply

Load the fitted gene regulatory network matrix A (e.g. produced by a prior scKINETICS EM fit) and randomly sample starting cell expression states x(0). For each independent simulation experiment (the benchmark used 15 experiments from distinct start cells), propagate expression through the linear dynamics x(t)=e^{At}x(0) at the known time points, computing the matrix exponential with numpy/scipy (scipy.linalg.expm), and add Gaussian noise with SD=5 to the simulated counts. Refit scKINETICS via its expectation-maximization fitting of the dynamical model on each simulated expression matrix to recover A. For each refit, compute the Pearson correlation between the inferred A and the true A restricted to the nonzero elements of the true matrix, then average the per-experiment correlations. The rationale is that the linear model admits an exact closed-form propagator, so simulated data with known ground truth provides a direct test of recovery accuracy under noise.

## Related tools

- **scKINETICS** (Dynamical model whose fitted A matrix seeds the simulation and whose expectation-maximization fitting refits the recovered network from simulated data) — github.com/dpeerlab/scKINETICS
- **Jupyter** (Notebook environment for running the simulation, matrix-exponential propagation, and correlation analysis)

## Examples

```
from scipy.linalg import expm; Xt = np.array([expm(A*t) @ x0 for t in timepoints]) + np.random.normal(0, 5, Xt.shape)
```

## Evaluation signals

- Mean Pearson correlation over nonzero elements of the true A across 15 experiments should be within a small tolerance of the benchmark r = 0.665
- Per-experiment correlations written to network_recovery_correlations.csv should be positive and clustered near the mean value
- Correlations must be computed only on the nonzero elements of the true A matrix (support restriction)
- Simulated counts must include Gaussian noise at the specified SD = 5; noise-free replication is a different test
- Matrix-exponential propagation must use the exact known time points, matching scipy/numpy implementation

## Limitations

- Recovery accuracy is bounded (~r=0.665 on nonzero elements); substantial residual error remains even under the model's own dynamics
- Validation is specific to Gaussian noise (SD = 5) and the linear dynamical assumption; other noise models or nonlinearities are not covered
- Correlation restricted to nonzero elements ignores false positives outside the true support

## Evidence

- [full_text] linear dynamics and matrix exponential: [evidence span withheld — anchor status: not_found]
- [abstract] EM fitting of dynamical model: "a dynamical model of gene expression change which is fit with the simultaneous learning of per-cell transcriptional velocities and a governing gene regulatory network"
- [intro] EM derived to learn regulator impact: "This is accomplished through an expectation-maximization approach derived to learn the impact of each regulator on its target genes"
- [full_text] benchmark correlation value: [evidence span withheld — anchor status: not_found]
- [readme] README demo scope: [evidence span withheld — anchor status: external_doc]
