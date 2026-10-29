---
name: noisy-single-cell-count-simulation
description: Use when benchmarking whether a GRN inference method (e.g., scKINETICS
  EM fitting) can recover a known ground-truth regulatory matrix A from simulated
  expression trajectories with added noise.
when_to_use_negative:
- You already have real single-cell expression data with a validated ground-truth
  network — simulate only when a true A is needed for controlled recovery benchmarks
- You need velocity or manifold-constrained inference on observed data rather than
  a linear-systems simulation study — use the scKINETICS fitting workflow directly
  instead
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scKINETICS
  role: Refits the dynamical model via expectation-maximization on each simulated
    expression matrix to recover the regulatory network A
  repo: http://github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Notebook environment for running the simulation, matrix-exponential propagation,
    correlation analysis, and CSV output
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:75beb0f6b01c518bd47f35f6562933b629bd85b1517e982dc71340e5342b7100
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# noisy-single-cell-count-simulation

## Summary

Simulates noisy single-cell expression data by propagating sampled starting cell states through linear GRN dynamics x(t)=e^{At}x(0) and adding Gaussian noise (SD=5), enabling benchmarking of GRN recovery accuracy for methods like scKINETICS.

## When to use

Use when benchmarking whether a GRN inference method (e.g., scKINETICS EM fitting) can recover a known ground-truth regulatory matrix A from simulated expression trajectories with added noise.

## When NOT to use

- You already have real single-cell expression data with a validated ground-truth network — simulate only when a true A is needed for controlled recovery benchmarks
- You need velocity or manifold-constrained inference on observed data rather than a linear-systems simulation study — use the scKINETICS fitting workflow directly instead

## Inputs

- Ground-truth gene regulatory network matrix A (e.g., scKINETICS-fitted A matrix from an upstream fitting run)
- Starting cell expression states x(0) sampled from single-cell expression data
- Known simulation time points
- Gaussian noise level (SD = 5 in the benchmark)

## Outputs

- 15 simulated noisy expression matrices (one per independent experiment)
- Refitted regulatory network matrices A inferred by scKINETICS EM for each simulation
- network_recovery_correlations.csv with per-experiment and mean Pearson correlations between inferred and true A over nonzero elements

## How to apply

Start from a fitted ground-truth regulatory network matrix A (e.g., the scKINETICS-fitted A produced in an upstream task) and randomly sample starting cell expression states x(0) from real cells. For each independent experiment (the benchmark used 15 distinct start cells), propagate expression through the linear dynamics x(t)=e^{At}x(0) at the known time points using the matrix exponential (numpy/scipy), then add Gaussian noise with SD=5 to the simulated counts. Refit scKINETICS via its expectation-maximization fitting of the dynamical model on each simulated expression matrix to recover the regulatory network. For each refit, compute the Pearson correlation between the inferred A and the true A restricted to the nonzero elements of the true matrix, then average the 15 per-experiment correlations and write per-experiment and mean values to a CSV (network_recovery_correlations.csv).

## Related tools

- **scKINETICS** (Refits the dynamical model via expectation-maximization on each simulated expression matrix to recover the regulatory network A) — http://github.com/dpeerlab/scKINETICS
- **Jupyter** (Notebook environment for running the simulation, matrix-exponential propagation, correlation analysis, and CSV output)

## Examples

```
from scipy.linalg import expm; X_t = np.array([expm(A * t) @ x0 for t in time_points]) + np.random.normal(0, 5, X_t.shape)
```

## Evaluation signals

- Mean Pearson correlation between inferred and true A over nonzero elements should be within a small tolerance of the benchmark r=0.665
- Each of the 15 independent experiments yields a per-experiment correlation recorded in network_recovery_correlations.csv
- Correlations are computed only on nonzero elements of the true A, matching the benchmark definition
- Simulated counts contain Gaussian noise with SD=5 and are generated at the known time points via x(t)=e^{At}x(0)

## Limitations

- Simulation assumes linear dynamics x(t)=e^{At}x(0), so recovery benchmarks do not capture nonlinear regulatory effects
- Recovery accuracy depends on the noise level (benchmark used Gaussian SD=5) and on the choice of starting cell states; other noise regimes or start cells may not reproduce r=0.665

## Evidence

- [abstract] dynamical model of gene expression change which is fit with the simultaneous learning of per-cell transcriptional velocities and a governing gene regulatory network: "a dynamical model of gene expression change which is fit with the simultaneous learning of per-cell transcriptional velocities and a governing gene regulatory network"
- [intro] EM approach to learn regulator-target impacts: "This is accomplished through an expectation-maximization approach derived to learn the impact of each regulator on its target genes"
- [readme] README demo covers GRN construction, EM estimation and visualizations: [evidence span withheld — anchor status: external_doc]
