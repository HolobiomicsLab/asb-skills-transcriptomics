---
name: inferred-vs-true-network-correlation-analysis
description: Use when a fitted GRN matrix A is available and you need a quantitative
  recovery benchmark via noise-added in-silico expression simulations refit through
  the same model.
when_to_use_negative:
- When no ground-truth network matrix is available — the correlation metric is only
  defined against a known true A
- When evaluating velocity predictions rather than regulatory network structure —
  this skill targets A recovery, not per-cell velocity accuracy
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_2258
tools:
- name: scKINETICS
  role: Dynamical model refit by expectation-maximization on simulated expression
    matrices to recover the regulatory network A
  repo: github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Notebook environment for running the simulation, refitting, and correlation
    analysis workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:ba7929da7c97cca31eb4a20606242feed0e8b4a21795434498da6bae5f99ab39
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# inferred-vs-true-network-correlation-analysis

## Summary

Benchmarking protocol that quantifies how accurately scKINETICS recovers a known gene regulatory network by simulating expression trajectories from a fitted A matrix and correlating the refit A against the true A over its nonzero elements.

## When to use

Use when a fitted GRN matrix A is available and you need a quantitative recovery benchmark via noise-added in-silico expression simulations refit through the same model.

## When NOT to use

- When no ground-truth network matrix is available — the correlation metric is only defined against a known true A
- When evaluating velocity predictions rather than regulatory network structure — this skill targets A recovery, not per-cell velocity accuracy

## Inputs

- Fitted gene regulatory network matrix A from a prior scKINETICS EM fit (task_002 output)
- Known simulation time points
- Starting cell expression states x(0) sampled from single-cell expression data

## Outputs

- network_recovery_correlations.csv containing per-experiment Pearson correlations and the mean correlation over nonzero elements of the true A

## How to apply

Load the fitted regulatory network matrix A produced by an initial scKINETICS expectation-maximization fit, then randomly sample starting cell expression states x(0). For each of 15 independent experiments, propagate expression through the linear dynamics x(t)=e^{At}x(0) using a matrix exponential (numpy/scipy) at the known time points, adding Gaussian noise with SD=5 to the simulated counts. Refit scKINETICS via expectation-maximization on each of the 15 simulated expression matrices to recover A, and for each refit compute the Pearson correlation between inferred and true A restricted to the nonzero elements of the true matrix. Average the 15 per-experiment correlations; the benchmark expects a mean r of 0.665 within a small tolerance. Write per-experiment and mean values to network_recovery_correlations.csv.

## Related tools

- **scKINETICS** (Dynamical model refit by expectation-maximization on simulated expression matrices to recover the regulatory network A) — github.com/dpeerlab/scKINETICS
- **Jupyter** (Notebook environment for running the simulation, refitting, and correlation analysis workflow)

## Examples

```
from scipy.linalg import expm; Xt = x0 @ expm(A*t); Xt_noisy = Xt + np.random.normal(0, 5, Xt.shape)
```

## Evaluation signals

- Mean Pearson correlation over 15 experiments should reproduce the reported benchmark r=0.665 within a small tolerance
- Correlations must be restricted to nonzero elements of the true A matrix, not computed over all entries
- network_recovery_correlations.csv must contain both per-experiment values and the mean across 15 experiments
- All 15 experiments should use distinct randomly sampled starting cells x(0) with Gaussian noise SD=5 applied to simulated counts

## Limitations

- Recovery is evaluated on a linear dynamical model x(t)=e^{At}x(0); performance on nonlinear dynamics is not established by this benchmark
- The benchmark value r=0.665 was obtained with a fixed noise level (Gaussian SD=5); other noise regimes are not validated
- Only 15 experiments from distinct start cells were used, so the mean correlation estimate carries sampling variability

## Evidence

- [full_text] In 15 simulation experiments from distinct start cells with Gaussian noise (SD = 5), scKINETICS recovered the true network, correlating inferred to true A at an average r=.665 over nonzero elements.: [evidence span withheld — anchor status: not_found]
- [abstract] a dynamical model of gene expression change which is fit with the simultaneous learning of per-cell transcriptional velocities and a governing gene regulatory network: "a dynamical model of gene expression change which is fit with the simultaneous learning of per-cell transcriptional velocities and a governing gene regulatory network"
- [readme] The demo shows the main functionalities of the package including input file generation, GRN construction, EM estimation, visualizations, and the TF ablation experiment.: [evidence span withheld — anchor status: external_doc]
