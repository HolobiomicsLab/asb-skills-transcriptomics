---
name: regulatory-network-trajectory-simulation
description: Use when a fitted GRN matrix A (e.g. from scKINETICS EM fitting) must
  be validated for recovery accuracy by simulating noisy expression trajectories from
  random start cells and refitting the model.
when_to_use_negative:
- Input is real single-cell expression data with no ground-truth network — there is
  no true A to correlate against, so recovery accuracy cannot be assessed
- The goal is to fit a GRN on real data rather than to benchmark recovery of an already-fitted
  A matrix
- The underlying dynamics are strongly nonlinear and cannot be approximated by the
  linear model x(t)=e^{At}x(0) used in this benchmark
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: scKINETICS
  role: Dynamical model refit via expectation-maximization to recover the regulatory
    network A from each simulated noisy expression matrix
  repo: http://github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Notebook environment for running the simulation, matrix exponential propagation,
    and correlation analysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:3bd4e932cc49f87701f7544082b19703c094d52bcb4f7300a0a87b061d587cc3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# regulatory-network-trajectory-simulation

## Summary

Validates an inferred gene regulatory network by propagating simulated expression states through the fitted linear dynamics x(t)=e^{At}x(0) with added Gaussian noise, refitting the model, and measuring recovery of the true A matrix — the simulation-based benchmark used by scKINETICS.

## When to use

Use when a fitted GRN matrix A (e.g. from scKINETICS EM fitting) must be validated for recovery accuracy by simulating noisy expression trajectories from random start cells and refitting the model.

## When NOT to use

- Input is real single-cell expression data with no ground-truth network — there is no true A to correlate against, so recovery accuracy cannot be assessed
- The goal is to fit a GRN on real data rather than to benchmark recovery of an already-fitted A matrix
- The underlying dynamics are strongly nonlinear and cannot be approximated by the linear model x(t)=e^{At}x(0) used in this benchmark

## Inputs

- Fitted gene regulatory network matrix A from scKINETICS EM fitting (task_002 output)
- Known simulation time points
- Starting cell expression states x(0) sampled from single-cell expression data

## Outputs

- network_recovery_correlations.csv with 15 per-experiment Pearson correlations (inferred vs true A over nonzero elements) and their mean
- 15 simulated noisy expression matrices used for refitting

## How to apply

Load the fitted A matrix (produced by the earlier scKINETICS EM fitting step) and randomly sample starting cell expression states x(0). For each of 15 independent experiments from distinct start cells, propagate expression through the linear dynamics x(t)=e^{At}x(0) at the known time points using the matrix exponential (numpy/scipy), then add Gaussian noise with SD=5 to the simulated counts. Refit scKINETICS via expectation-maximization on each simulated expression matrix to recover the regulatory network A. Compute the Pearson correlation between inferred and true A restricted to the nonzero elements of the true matrix, average the 15 per-experiment correlations, and write per-experiment and mean values to network_recovery_correlations.csv.

## Related tools

- **scKINETICS** (Dynamical model refit via expectation-maximization to recover the regulatory network A from each simulated noisy expression matrix) — http://github.com/dpeerlab/scKINETICS
- **Jupyter** (Notebook environment for running the simulation, matrix exponential propagation, and correlation analysis)

## Examples

```
from scipy.linalg import expm; Xt = [expm(A*t) @ x0 for t in timepoints]; Xnoisy = Xt + np.random.normal(0, 5, Xt.shape)
```

## Evaluation signals

- Mean Pearson correlation between inferred and true A over nonzero elements of the true matrix should be within a small tolerance of the reported benchmark r=0.665 across the 15 experiments
- All 15 per-experiment correlations are computed on the same support: nonzero elements of the true A matrix only
- Each of the 15 experiments uses a distinct randomly sampled start cell expression state x(0), not repeated seeds from the same cell
- Simulated counts must include Gaussian noise with SD=5 added to the propagated values x(t)=e^{At}x(0)
- network_recovery_correlations.csv contains both per-experiment and mean values

## Limitations

- Recovery is quantified only over nonzero elements of the true A, so false positives on truly zero entries are not captured by the correlation metric
- Benchmark reflects Gaussian noise (SD=5) on simulated linear dynamics; performance on real data with model misspecification may differ
- No changelog found for the scKINETICS repository, so version-specific behavior of the refitting step is not documented

## Evidence

- [results] 15 simulation experiments from distinct start cells with Gaussian noise (SD = 5), scKINETICS recovered the true network, correlating inferred to true A at an average r=.665 over nonzero elements: [evidence span withheld — anchor status: not_found]
- [methods] EM fitting of the dynamical model: [evidence span withheld — anchor status: not_found]
- [abstract] dynamical model fit with simultaneous learning of velocities and GRN: "a dynamical model of gene expression change which is fit with the simultaneous learning of per-cell transcriptional velocities and a governing gene regulatory network"
- [readme] README demo description: [evidence span withheld — anchor status: external_doc]
