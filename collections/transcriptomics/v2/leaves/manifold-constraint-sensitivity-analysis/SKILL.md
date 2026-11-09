---
name: manifold-constraint-sensitivity-analysis
description: Use when scKINETICS velocity outputs must be stress-tested against variation
  in the manifold-constraint parameter sigma or against noise injected into the coexpression
  prior A^, with all other inputs held fixed.
when_to_use_negative:
- You need to fit scKINETICS on a new dataset rather than assess robustness of an
  existing fit's parameters
- The goal is perturbing epigenetic priors or other inputs — this skill holds those
  fixed and only varies sigma and A^ noise
- Input lacks a converged reference fit to serve as the baseline for computing velocity
  deviations
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scKINETICS
  role: Dynamical model refit by expectation-maximization at each sigma value and
    after each Gaussian perturbation of the coexpression prior A^
  repo: http://github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Notebook providing the demonstration dataset and workflow in which the sigma
    sweep and prior-noise perturbation are executed
  repo: http://github.com/dpeerlab/scKINETICS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:ca5de0ed0bbb76d445ab06db5cc9a3a0ab6bbbf62fe8de3f1e0c0f21de4b7525
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# manifold-constraint-sensitivity-analysis

## Summary

A robustness-probing procedure for scKINETICS that sweeps the user-defined phenotypic manifold-constraint width sigma across a grid and perturbs the coexpression prior A^ with Gaussian noise, refitting the EM model at each setting to quantify deviation of per-cell velocities and network alignment.

## When to use

Use when scKINETICS velocity outputs must be stress-tested against variation in the manifold-constraint parameter sigma or against noise injected into the coexpression prior A^, with all other inputs held fixed.

## When NOT to use

- You need to fit scKINETICS on a new dataset rather than assess robustness of an existing fit's parameters
- The goal is perturbing epigenetic priors or other inputs — this skill holds those fixed and only varies sigma and A^ noise
- Input lacks a converged reference fit to serve as the baseline for computing velocity deviations

## Inputs

- Demonstration single-cell dataset from the scKINETICS Jupyter notebook (reference fit inputs)
- Reference scKINETICS fit (per-cell velocities and inferred GRN) with all priors fixed
- Coexpression prior matrix A^ (nonzero elements ~1e-4 scale) to be perturbed
- Grid of initiating sigma values for the manifold-constraint sweep

## Outputs

- Per-sigma velocity deviation metrics (cosine distance vs. reference fit)
- Per-noise-round network alignment statistics against the perturbed prior
- Sensitivity summary table written to sensitivity_analysis_results.csv

## How to apply

Load the demonstration dataset from the scKINETICS Jupyter notebook and fix every model input except the parameter under perturbation. First sweep sigma over a grid of initiating values, refitting scKINETICS via expectation-maximization at each sigma while keeping the epigenetic and coexpression priors unchanged. Independently, perturb the nonzero elements of the coexpression prior A^ with successive rounds of Gaussian noise whose magnitude exceeds the ~1e-4 scale of true A^ values, refitting the full model after each round. For each sigma value and noise round, compute the deviation of refitted per-cell velocities from the reference fit (e.g., cosine distance) and check whether the inferred network remains aligned to the perturbed prior; the paper reports nearly perfect velocity recovery across the sigma range with cosine distance < 0.6 even at extreme sigma, and consistent network alignment under prior noise.

## Related tools

- **scKINETICS** (Dynamical model refit by expectation-maximization at each sigma value and after each Gaussian perturbation of the coexpression prior A^) — http://github.com/dpeerlab/scKINETICS
- **Jupyter** (Notebook providing the demonstration dataset and workflow in which the sigma sweep and prior-noise perturbation are executed) — http://github.com/dpeerlab/scKINETICS

## Examples

```
import scKINETICS; # load demo notebook dataset, then for each sigma in sweep: fit via EM with manifold constraint sigma, perturb A^ with Gaussian noise > 1e-4 scale, write sensitivity_analysis_results.csv
```

## Evaluation signals

- Tabulated velocity deviations stay below 0.6 cosine distance across the full sigma sweep range
- Nearly perfect velocity recovery observed over a wide range of initiating sigma values
- Inferred network remains consistently aligned to the perturbed A^ under Gaussian noise exceeding the ~1e-4 true-prior scale
- All other model inputs (epigenetic prior, coexpression prior, dataset) unchanged between reference and perturbation runs
- Summary results persisted to sensitivity_analysis_results.csv with per-sigma and per-noise-round rows

## Limitations

- Robustness claims hold for the demonstration dataset in the accompanying notebook; other datasets may not reproduce the <0.6 cosine-distance behavior
- Even at extreme sigma, deviation is modest but nonzero (cosine distance < 0.6), so velocities are not strictly invariant
- No changelog found for the scKINETICS repository, so version-specific behavior of the sensitivity workflow is untracked

## Evidence

- [abstract] robustness to sigma sweep: [evidence span withheld — anchor status: not_found]
- [intro] EM fitting of the model: "This is accomplished through an expectation-maximization approach derived to learn the impact of each regulator on its target genes"
- [readme] demo notebook coverage: [evidence span withheld — anchor status: external_doc]
- [abstract] EM learning of velocities and GRN: "a dynamical model of gene expression change which is fit with the simultaneous learning of per-cell transcriptional velocities and a governing gene regulatory network"
