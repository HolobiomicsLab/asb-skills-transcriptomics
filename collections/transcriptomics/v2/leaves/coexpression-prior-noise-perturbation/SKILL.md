---
name: coexpression-prior-noise-perturbation
description: Use when assessing robustness of scKINETICS-derived velocity estimates
  and gene regulatory networks to variation in sigma and to noise injected into the
  coexpression prior A^, holding all other inputs fixed.
when_to_use_negative:
- The goal is primary velocity/GRN inference rather than sensitivity analysis — run
  scKINETICS once with the chosen priors instead of perturbing them.
- The perturbation target is the epigenetic prior or phenotypic manifold rather than
  the coexpression prior A^ and sigma; this workflow fixes those inputs unchanged.
- Reference per-cell velocities from an unperturbed scKINETICS fit are unavailable,
  so cosine-distance deviation cannot be computed.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scKINETICS
  role: Dynamical model refit by expectation-maximization at each sigma value and
    after each Gaussian-noise perturbation of the coexpression prior A^; produces
    per-cell velocities and the gene regulatory network
  repo: github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Notebook providing the demonstration dataset and main functionalities (input
    file generation, GRN construction, EM estimation) used as the fixed reference
    environment for the sweep
  repo: github.com/dpeerlab/scKINETICS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:16507eb64f5b277d8d6e7c4afe2f0bc041dd0238ba2e951a3bcaaeec1abec6dc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# coexpression-prior-noise-perturbation

## Summary

A sensitivity-analysis procedure for scKINETICS that perturbs the coexpression prior A^ with successive rounds of Gaussian noise and sweeps the manifold-constraint parameter sigma, quantifying how inferred per-cell velocities and network alignment deviate from a reference fit.

## When to use

Use when assessing robustness of scKINETICS-derived velocity estimates and gene regulatory networks to variation in sigma and to noise injected into the coexpression prior A^, holding all other inputs fixed.

## When NOT to use

- The goal is primary velocity/GRN inference rather than sensitivity analysis — run scKINETICS once with the chosen priors instead of perturbing them.
- The perturbation target is the epigenetic prior or phenotypic manifold rather than the coexpression prior A^ and sigma; this workflow fixes those inputs unchanged.
- Reference per-cell velocities from an unperturbed scKINETICS fit are unavailable, so cosine-distance deviation cannot be computed.

## Inputs

- Demonstration dataset from the scKINETICS Jupyter notebook
- Reference (unperturbed) scKINETICS fit: per-cell velocities and inferred gene regulatory network
- Coexpression prior A^ (nonzero elements ~1e-4 scale)

## Outputs

- Sigma-sweep table of velocity cosine-distance deviations per initiating sigma value
- Noise-sweep table of velocity deviation and network alignment per Gaussian-noise round
- Summary table written to sensitivity_analysis_results.csv

## How to apply

Load the demonstration dataset from the scKINETICS Jupyter notebook and fix all model inputs except the parameters under perturbation. Sweep the manifold-constraint width sigma over a grid of initiating values, refitting the expectation-maximization model at each sigma while keeping epigenetic and coexpression priors unchanged. Separately perturb the nonzero elements of the coexpression prior A^ with successive rounds of Gaussian noise whose magnitude exceeds the ~1e-4 scale of true A^ values, refitting the full EM model after each round. For every sigma and noise round, compute the cosine-distance deviation of refitted per-cell velocities from the reference fit and the alignment of the inferred network to the perturbed prior, then tabulate results (e.g., sensitivity_analysis_results.csv).

## Related tools

- **scKINETICS** (Dynamical model refit by expectation-maximization at each sigma value and after each Gaussian-noise perturbation of the coexpression prior A^; produces per-cell velocities and the gene regulatory network) — github.com/dpeerlab/scKINETICS
- **Jupyter** (Notebook providing the demonstration dataset and main functionalities (input file generation, GRN construction, EM estimation) used as the fixed reference environment for the sweep) — github.com/dpeerlab/scKINETICS

## Examples

```
python run_sensitivity.py --sigma-grid 0.1,1,10,100 --noise-rounds 5 --noise-scale 1e-3 --output sensitivity_analysis_results.csv
```

## Evaluation signals

- Velocity cosine-distance deviation stays below 0.6 across the sigma sweep, including at extreme sigma values.
- Per-cell vectors remain consistently aligned to the reference fit under every round of Gaussian noise added to A^.
- Noise magnitude injected into nonzero A^ elements exceeds the ~1e-4 scale of the true A^ values.
- Epigenetic and coexpression priors remain unchanged across all sigma-sweep refits (only sigma varies).
- Summary table (sensitivity_analysis_results.csv) contains one row per sigma value and per noise round with computed deviations.

## Limitations

- Robustness is demonstrated on the notebook demonstration dataset; conclusions about sigma and prior-noise insensitivity may not transfer to other tissues or manifolds.
- The workflow fixes epigenetic and manifold inputs constant, so it does not assess sensitivity to those priors.
- No changelog was found for the scKINETICS repository, so version-specific behavior of the EM refits is unverified.

## Evidence

- [results] robustness finding: [evidence span withheld — anchor status: not_found]
- [results] prior-noise alignment: [evidence span withheld — anchor status: not_found]
- [abstract] EM fitting approach: "fit with the simultaneous learning of per-cell transcriptional velocities and a governing gene regulatory network"
- [intro] coexpression prior role: "leveraging biologically-motivated priors from epigenetic data, gene–gene co-expression, and constraints on cells’ future states imposed by the phenotypic manifold"
- [readme] demo availability: [evidence span withheld — anchor status: external_doc]
