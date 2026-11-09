---
name: velocity-recovery-benchmarking
description: Use when assessing whether scKINETICS-inferred per-cell velocities and
  GRNs are stable to user-defined hyperparameter choices (initiating sigma) or perturbation
  of prior inputs (coexpression matrix A^), before trusting downstream regulator calls.
when_to_use_negative:
- The goal is accuracy benchmarking of scKINETICS against other velocity methods (e.g.
  RNA velocity baselines) rather than internal parameter-sensitivity testing.
- The coexpression prior A^ or manifold-constraint sigma will not be perturbed because
  a single standard fit suffices for the analysis at hand.
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scKINETICS
  role: Dynamical model refit by expectation-maximization at each sigma value and
    after each round of Gaussian noise on the coexpression prior A^; source of per-cell
    velocities and the inferred gene regulatory network.
  repo: http://github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Notebook environment used to load the demonstration dataset and run the sigma-sweep
    and prior-noise experiments.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:6854efb941de7fb6f7ab018489c791220d656e70ea72e66f536fb9e9fe330b53
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# velocity-recovery-benchmarking

## Summary

A robustness-benchmarking procedure for scKINETICS that quantifies how inferred per-cell velocities and the governing gene regulatory network respond to perturbation of the manifold-constraint width sigma and to Gaussian noise added to the coexpression prior A^, using cosine distance of refitted velocities against a reference fit.

## When to use

Use when assessing whether scKINETICS-inferred per-cell velocities and GRNs are stable to user-defined hyperparameter choices (initiating sigma) or perturbation of prior inputs (coexpression matrix A^), before trusting downstream regulator calls.

## When NOT to use

- The goal is accuracy benchmarking of scKINETICS against other velocity methods (e.g. RNA velocity baselines) rather than internal parameter-sensitivity testing.
- The coexpression prior A^ or manifold-constraint sigma will not be perturbed because a single standard fit suffices for the analysis at hand.

## Inputs

- Demonstration single-cell dataset used in the scKINETICS Jupyter notebook (with epigenetic and coexpression priors)
- Reference scKINETICS fit (per-cell velocities and inferred GRN) at default parameter values
- Coexpression prior matrix A^ with nonzero elements of ~1e-4 magnitude
- Grid of initiating sigma values for the manifold-constraint prior

## Outputs

- sensitivity_analysis_results.csv tabulating velocity deviation (cosine distance) per sigma value and noise round
- Per-sigma refitted velocity estimates for comparison against the reference fit
- Per-noise-round network-alignment metrics between inferred GRN and perturbed prior

## How to apply

Load the demonstration dataset from the scKINETICS Jupyter notebook and fix all model inputs except the parameter under perturbation. Sweep the manifold-constraint width sigma over a grid of initiating values, refitting scKINETICS by expectation-maximization at each sigma while keeping the epigenetic and coexpression priors unchanged. Separately perturb the nonzero elements of the coexpression prior A^ with successive rounds of Gaussian noise whose magnitude exceeds the ~1e-4 scale of the true A^ values, refitting the full model after each round. For every sigma value and noise round, compute the deviation of refitted per-cell velocities from the reference fit (cosine distance) and the alignment of the inferred network to the perturbed prior; tabulate results to sensitivity_analysis_results.csv. The method is grounded in the EM scheme that simultaneously learns per-cell transcriptional velocities and a governing gene regulatory network, so each refit must be a complete EM run under the new sigma or prior.

## Related tools

- **scKINETICS** (Dynamical model refit by expectation-maximization at each sigma value and after each round of Gaussian noise on the coexpression prior A^; source of per-cell velocities and the inferred gene regulatory network.) — http://github.com/dpeerlab/scKINETICS
- **Jupyter** (Notebook environment used to load the demonstration dataset and run the sigma-sweep and prior-noise experiments.)

## Examples

```
python run_sensitivity.py --sigma_grid 0.1,1,10,100 --noise_rounds 5 --noise_scale 1e-3 --prior A_hat.h5ad --out sensitivity_analysis_results.csv
```

## Evaluation signals

- Tabulated velocity deviation (cosine distance) stays below 0.6 across the entire initiating-sigma range, including extreme sigma values.
- Nearly perfect velocity recovery (cosine distance near 0) is observed over the wide central range of initiating sigma values.
- Inferred-network alignment to the perturbed coexpression prior remains consistent under every round of Gaussian noise applied to A^.
- Noise magnitude added to nonzero A^ elements demonstrably exceeds the ~1e-4 scale of the true prior values, confirming the perturbation is meaningful.
- sensitivity_analysis_results.csv is produced with one row per (sigma value, noise round) combination and populated deviation/alignment metrics.

## Limitations

- Robustness is demonstrated only on the demonstration dataset from the accompanying scKINETICS notebook; behavior on other datasets or prior configurations is not guaranteed by this benchmark.
- Even at extreme sigma values, cosine distance can approach but stay under 0.6, so moderate velocity deviation at parameter extremes is an expected outcome rather than a failure.
- The benchmark perturbs only the coexpression prior A^ and sigma; sensitivity to the epigenetic prior is not exercised.

## Evidence

- [supplementary] nearly perfect velocity recovery is observed over a wide range of initiating σ values, with only modest deviation (cosine distance < 0.6) even at extreme σ: [evidence span withheld — anchor status: not_found]
- [abstract] an expectation-maximization approach derived to learn the impact of each regulator on its target genes: "This is accomplished through an expectation-maximization approach derived to learn the impact of each regulator on its target genes"
- [intro] leverage biologically-motivated priors from epigenetic data, gene-gene co-expression, and constraints on cells' future states imposed by the phenotypic manifold: "leveraging biologically-motivated priors from epigenetic data, gene–gene co-expression, and constraints on cells’ future states imposed by the phenotypic manifold"
- [readme] The demo shows the main functionalities of the package including input file generation, GRN construction, EM estimation, visualizations, and the TF ablation experiment.: [evidence span withheld — anchor status: external_doc]
