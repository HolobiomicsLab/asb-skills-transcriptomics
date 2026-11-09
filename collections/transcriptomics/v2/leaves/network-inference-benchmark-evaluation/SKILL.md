---
name: network-inference-benchmark-evaluation
description: Use when you must quantitatively validate a GRN inference method against
  ground truth — specifically when you have a fitted network matrix A and need to
  test whether refitting on simulated expression recovers the true regulator-target
  edges.
when_to_use_negative:
- Input is real single-cell transcriptomics data with no known ground-truth network
  — recovery cannot be scored and qualitative validation (e.g., TF ablation, known
  transdifferentiation axis recapitulation) is appropriate instead
- The goal is velocity or trajectory inference itself rather than quantitative assessment
  of network recovery accuracy
- The dynamical model is not linear (x(t)=e^{At}x(0) does not describe the fitted
  system), so the matrix-exponential simulation step is invalid
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scKINETICS
  role: Dynamical model refit via expectation-maximization on each of the 15 simulated
    expression matrices to recover the regulatory network A
  repo: http://github.com/dpeerlab/scKINETICS
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
  source_content_hash: sha256:78064b0319318007bd0ebb31e1a5ec94f961561fa1a7866977f7a0b56c0f57d6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# network-inference-benchmark-evaluation

## Summary

A simulation-based benchmark for scKINETICS-style gene regulatory network inference: propagate expression through linear dynamics from a known network matrix A, refit the model, and score recovery by Pearson correlation between inferred and true A over the true network's nonzero elements.

## When to use

Use when you must quantitatively validate a GRN inference method against ground truth — specifically when you have a fitted network matrix A and need to test whether refitting on simulated expression recovers the true regulator-target edges.

## When NOT to use

- Input is real single-cell transcriptomics data with no known ground-truth network — recovery cannot be scored and qualitative validation (e.g., TF ablation, known transdifferentiation axis recapitulation) is appropriate instead
- The goal is velocity or trajectory inference itself rather than quantitative assessment of network recovery accuracy
- The dynamical model is not linear (x(t)=e^{At}x(0) does not describe the fitted system), so the matrix-exponential simulation step is invalid

## Inputs

- Fitted gene regulatory network matrix A from a prior scKINETICS EM fit (e.g., output of the initial fitting task)
- Randomly sampled starting cell expression states x(0)
- Known simulation time points
- Gaussian noise specification (SD = 5)

## Outputs

- Per-experiment Pearson correlations between inferred and true A over nonzero elements of the true matrix
- network_recovery_correlations.csv containing per-experiment and mean correlation values
- Mean recovery correlation (benchmark: r = 0.665)

## How to apply

Load the fitted A matrix from a prior scKINETICS EM fit and randomly sample starting cell expression states x(0). For each of 15 independent experiments from distinct start cells, propagate expression through the linear dynamics x(t)=e^{At}x(0) at the known time points using a matrix exponential (numpy/scipy), then add Gaussian noise with SD=5 to the simulated counts. Refit scKINETICS via its expectation-maximization fitting of the dynamical model on each of the 15 simulated expression matrices. For each refit, compute the Pearson correlation between the inferred A and the true A restricted to the nonzero elements of the true matrix. Average the 15 per-experiment correlations and write per-experiment and mean values to network_recovery_correlations.csv; acceptance requires the mean r to fall within a small tolerance of the reported benchmark r=0.665.

## Related tools

- **scKINETICS** (Dynamical model refit via expectation-maximization on each of the 15 simulated expression matrices to recover the regulatory network A) — http://github.com/dpeerlab/scKINETICS
- **Jupyter** (Notebook environment for running the simulation, refitting, and correlation analysis workflow)

## Examples

```
import numpy as np; from scipy.linalg import expm; X_t = expm(A * t) @ x0 + np.random.normal(scale=5, size=x0.shape)
```

## Evaluation signals

- Mean Pearson correlation over the 15 experiments falls within a small tolerance of the benchmark r=0.665
- Correlations are computed only over nonzero elements of the true A matrix (not the full matrix)
- Exactly 15 independent experiments are run, each starting from a distinct randomly sampled start cell
- network_recovery_correlations.csv contains both per-experiment and mean values
- Simulated counts are generated via x(t)=e^{At}x(0) with additive Gaussian noise of SD=5

## Limitations

- The benchmark assumes linear gene expression dynamics (x(t)=e^{At}x(0)); recovery accuracy is only meaningful under this simulated regime
- Results are specific to the benchmark noise level (Gaussian, SD=5); other noise regimes may not reproduce r=0.665
- No changelog found for the scKINETICS repository, so version drift in the EM fitting cannot be tracked from the repo alone

## Evidence

- [abstract] dynamical model fit: "a dynamical model of gene expression change which is fit with the simultaneous learning of per-cell transcriptional velocities and a governing gene regulatory network"
- [intro] EM learning of regulator impact: "This is accomplished through an expectation-maximization approach derived to learn the impact of each regulator on its target genes"
- [abstract] benchmarking claim: "scKINETICS successfully extends and improves existing velocity approaches to generate interpretable, mechanistic models of gene regulatory dynamics"
- [readme] demo functionalities: [evidence span withheld — anchor status: external_doc]
