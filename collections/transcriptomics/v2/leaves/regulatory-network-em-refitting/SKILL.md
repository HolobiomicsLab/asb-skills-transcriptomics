---
name: regulatory-network-em-refitting
description: Use when you need to test robustness of scKINETICS velocity/network outputs
  by sweeping sigma over an initiating grid or perturbing nonzero A^ coexpression-prior
  elements with Gaussian noise while holding all other inputs fixed.
when_to_use_negative:
- You are fitting scKINETICS for the first time to derive velocities/networks rather
  than stress-testing an existing reference fit
- You are benchmarking scKINETICS against other velocity methods rather than assessing
  its internal robustness to sigma and prior perturbation
- No epigenetic/coexpression priors or manifold constraint are defined, so there is
  no A^ to perturb and no sigma to sweep
tools:
- name: scKINETICS
  role: Dynamical model refit with expectation-maximization at each sigma value and
    each Gaussian-noise perturbation of A^
  repo: http://github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Source of the demonstration dataset and accompanying notebook used to fix
    reference inputs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:eea1cc75daa66cb53e8cdf998f269bc0ad7615d66de521cb52ae0ecbd3e20291
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# regulatory-network-em-refitting

## Summary

Refits the scKINETICS expectation-maximization model across perturbations — sweeping the manifold-constraint width sigma and adding Gaussian noise to the coexpression prior A^ — to quantify how sensitive inferred per-cell velocities and the governing gene regulatory network are to prior/parameter choices.

## When to use

Use when you need to test robustness of scKINETICS velocity/network outputs by sweeping sigma over an initiating grid or perturbing nonzero A^ coexpression-prior elements with Gaussian noise while holding all other inputs fixed.

## When NOT to use

- You are fitting scKINETICS for the first time to derive velocities/networks rather than stress-testing an existing reference fit
- You are benchmarking scKINETICS against other velocity methods rather than assessing its internal robustness to sigma and prior perturbation
- No epigenetic/coexpression priors or manifold constraint are defined, so there is no A^ to perturb and no sigma to sweep

## Inputs

- Demonstration single-cell dataset from the scKINETICS Jupyter notebook
- Reference scKINETICS fit (per-cell transcriptional velocities and inferred gene regulatory network)
- Coexpression prior matrix A^ (nonzero elements at ~1e-4 scale)
- Grid of initiating sigma values for the phenotypic manifold constraint

## Outputs

- sensitivity_analysis_results.csv — per-sigma and per-noise-round deviation and network-alignment summary table

## How to apply

Load the demonstration dataset from the accompanying scKINETICS Jupyter notebook and fix every model input except the parameter under perturbation. First, sweep the manifold-constraint width sigma over a grid of initiating values, refitting scKINETICS with expectation-maximization at each sigma while keeping the epigenetic and coexpression priors unchanged. Second, perturb the nonzero elements of the coexpression prior A^ with successive rounds of Gaussian noise whose magnitude exceeds the ~1e-4 scale of the true A^ values, refitting the full EM model after each round. For every sigma value and noise round, compute the deviation of refitted per-cell velocities from the reference fit (cosine distance) and the alignment of the inferred network to the perturbed prior. Tabulate both sweeps and accept the run if deviations stay below 0.6 across the sigma range and network alignment remains consistent under noise above the ~1e-4 prior scale.

## Related tools

- **scKINETICS** (Dynamical model refit with expectation-maximization at each sigma value and each Gaussian-noise perturbation of A^) — http://github.com/dpeerlab/scKINETICS
- **Jupyter** (Source of the demonstration dataset and accompanying notebook used to fix reference inputs)

## Evaluation signals

- Velocity deviation from the reference fit, measured as cosine distance, stays below 0.6 across the entire sigma-sweep grid
- Inferred network remains consistently aligned to the perturbed prior even when Gaussian noise magnitude exceeds the ~1e-4 scale of true A^ values
- sensitivity_analysis_results.csv contains rows for every sigma value and noise round with deviation and alignment columns populated
- Each EM refit is run with epigenetic and coexpression priors held unchanged, so only sigma or A^ varies between fits

## Limitations

- Robustness was verified on the demonstration dataset from the accompanying notebook; generalization to other datasets is not established by this experiment
- Even at extreme sigma values deviation can approach the 0.6 cosine-distance bound, so near-perfect recovery is not guaranteed at the edges of the sweep
- Perturbation noise deliberately exceeds the true ~1e-4 magnitude of A^ elements, making this a stress test rather than a realistic prior-misspecification scenario
- No changelog is available for the scKINETICS repository, so version-specific behavior of the EM refitting is not documented

## Evidence

- [abstract] a dynamical model of gene expression change which is fit with the simultaneous learning of per-cell transcriptional velocities and a governing gene regulatory network: "a dynamical model of gene expression change which is fit with the simultaneous learning of per-cell transcriptional velocities and a governing gene regulatory network"
- [intro] an expectation-maximization approach derived to learn the impact of each regulator on its target genes: "an expectation-maximization approach derived to learn the impact of each regulator on its target genes"
- [readme] The demo shows the main functionalities of the package including input file generation, GRN construction, EM estimation, visualizations, and the TF ablation experiment.: [evidence span withheld — anchor status: external_doc]
