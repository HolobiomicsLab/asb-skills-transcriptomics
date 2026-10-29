---
name: ground-truth-trajectory-extraction
description: Use when benchmarking methods that require known ground truth (RNA velocity
  via velocity correlation/arrow cosine, trajectory inference, CSNI) on simulated
  single-cell data where the true trajectory is available by construction.
when_to_use_negative:
- Working with real experimental single-cell data, where the ground-truth trajectory
  and velocity are not known and cannot be extracted by construction
- Benchmarking methods that do not use trajectory information (e.g. clustering or
  differential expression), for which simulators like Splatter, powsimR, PROSSTT,
  or SymSim may suffice
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: dyngen
  role: Multi-modal simulation engine that generates the count matrix together with
    the ground-truth trajectory and cellwise GRN annotations from a backbone-defined
    module network
  repo: https://github.com/dynverse/dyngen
- name: GillespieSSA2
  role: Executes Gillespie's stochastic simulation algorithm producing molecule abundances
    at every state, from which progression times are derived
- name: scVelo
  role: Downstream RNA velocity method (stochastic and dynamical settings) whose outputs
    are evaluated against the extracted ground-truth velocities
- name: velocyto
  role: Downstream RNA velocity method evaluated against the extracted ground-truth
    velocities
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-24152-2
    title: Spearheading future omics analyses using dyngen, a multi-modal simulator
      of single cells
  source_content_hash: sha256:65684168ca276ec9a644b2d5d40b64903eca7c0b3dc0e32ba7c94fe0c4acddad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dynverse/dyngen
  tools:
  - GillespieSSA2
  - dyngen
  - scVelo
  - velocyto
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Ground-truth trajectory extraction from dyngen simulations

## Summary

Extracts per-cell ground-truth trajectories (cell states, progression/pseudotime, and ground-truth RNA velocity vectors) from dyngen single-cell simulations so that computational methods such as RNA velocity, trajectory inference, and trajectory alignment can be quantitatively benchmarked against completely known ground truth.

## When to use

Use when benchmarking methods that require known ground truth (RNA velocity via velocity correlation/arrow cosine, trajectory inference, CSNI) on simulated single-cell data where the true trajectory is available by construction.

## When NOT to use

- Working with real experimental single-cell data, where the ground-truth trajectory and velocity are not known and cannot be extracted by construction
- Benchmarking methods that do not use trajectory information (e.g. clustering or differential expression), for which simulators like Splatter, powsimR, PROSSTT, or SymSim may suffice

## Inputs

- dyngen model initialised with a predefined backbone (e.g. via backbone_bifurcating or backbone_converging constructors)
- Gene regulatory network sampled from FANTOM5 GRNs (target-regulator interactions, max in-degree ~5)
- Random seed(s) controlling the Gillespie SSA stochastic simulation
- Real reference single-cell RNA-seq dataset used for experiment emulation (e.g. zenodo_1443566_real_silver_bone-marrow-mesenchyme-erythrocyte-differentiation_mca)

## Outputs

- Per-dataset transcriptomics count matrix (spliced/unspliced abundances for RNA velocity evaluation)
- Ground-truth trajectory with cell states and progression (pseudotime) per cell
- Cell-specific ground-truth regulatory network with regulatory effects bounded in [-1, 1]
- Ground-truth RNA velocity vectors for computing velocity correlation and velocity arrow cosine

## How to apply

Build a dyngen model with a predefined backbone (e.g. backbone_bifurcating, backbone_bifurcating_cycle, backbone_converging) whose module network is generated from a sampled FANTOM5 GRN, then run generate_dataset(), which internally executes gold-standard simulation, a Gillespie SSA cell simulation (GillespieSSA2), ground-truth trajectory mapping by expression correlation, per-cell regulatory-effect computation, and experiment emulation. From the SSA simulation, the abundance levels of all molecules at every state are known, so the true trajectory with progression times can be attached to every simulated cell alongside the transcriptomics count matrix. For each backbone, repeat with multiple random seeds (the manuscript benchmark used 14 backbones x 3 seeds = 42 datasets) and store the count matrix, ground-truth trajectory with progression times, and cellwise regulatory network with effects bounded in [-1, 1]. The ground-truth velocity vector is then compared to method outputs (velocyto, scVelo stochastic/dynamical) using velocity correlation and velocity arrow cosine metrics, and trajectory alignment is scored with ABWAP against the worst possible alignment.

## Related tools

- **dyngen** (Multi-modal simulation engine that generates the count matrix together with the ground-truth trajectory and cellwise GRN annotations from a backbone-defined module network) — https://github.com/dynverse/dyngen
- **GillespieSSA2** (Executes Gillespie's stochastic simulation algorithm producing molecule abundances at every state, from which progression times are derived)
- **scVelo** (Downstream RNA velocity method (stochastic and dynamical settings) whose outputs are evaluated against the extracted ground-truth velocities)
- **velocyto** (Downstream RNA velocity method evaluated against the extracted ground-truth velocities)

## Examples

```
model <- initialise_model(backbone = backbone_bifurcating()); dataset <- generate_dataset(model, seed = 1)
```

## Evaluation signals

- Dataset inventory check: exactly 14 backbones x 3 seeds = 42 complete datasets produced, each containing a count matrix plus ground-truth trajectory and cellwise GRN annotations
- Regulatory effects in every extracted cellwise GRN are bounded in [-1, 1]
- Every simulated cell is mapped to the ground-truth trajectory with a progression time derived from the SSA states
- Velocity benchmark yields finite velocity correlation and velocity arrow cosine values for velocyto and scVelo (stochastic and dynamical) on all 42 datasets
- Trajectory alignment runs yield ABWAP scores comparable against the worst possible alignment path

## Limitations

- dyngen assumes cells are well-mixed and independent from each other, which may not reflect real cellular populations
- Trajectory-alignment evaluation (as in the manuscript) was demonstrated on linear trajectories, so ground-truth alignment for complex backbones may need extra care
- No software version numbers were reported for dyngen, velocyto, or scVelo in the manuscript, complicating exact reproduction

## Evidence

- [methods] SSA abundance states: "From the SSA simulation we obtain the abundance levels of all the molecules at every state."
- [methods] SSA-based cell simulation: [evidence span withheld — anchor status: not_found]
- [intro] Benchmarking scope: "dyngen can cater to a wide range of benchmarking applications, including trajectory inference, trajectory alignment, and trajectory differential expression"
- [results] 42-dataset velocity benchmark: "We executed velocyto and scvelo (with 2 different parameter settings, stochastic and dynamical) on 42 datasets"
- [readme] Reproducibility scripts: [evidence span withheld — anchor status: external_doc]
