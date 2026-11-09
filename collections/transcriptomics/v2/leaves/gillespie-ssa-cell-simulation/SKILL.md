---
name: gillespie-ssa-cell-simulation
description: Use when dynamic cellular processes must be simulated in silico from
  a known GRN with ground-truth molecule counts (pre-mRNA, mRNA, protein) in the low-copy-number
  regime typical of single-cell omics.
when_to_use_negative:
- When simulating bulk data with high molecule counts, where ODE-based models are
  appropriate and SSA stochasticity is unnecessary
- When the goal is only technical protocol emulation without cell dynamics (e.g. Splatter-style
  simulation of library-size and dropout properties)
- When spatial or inter-cellular effects matter — dyngen assumes cells are well-mixed
  and independent from each other
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: dyngen
  role: 'Orchestrates the simulation pipeline: GRN-to-reaction conversion, SSA-based
    cell simulation, sampling, and experiment emulation'
  repo: https://github.com/dynverse/dyngen
- name: GillespieSSA2
  role: Executes Gillespie's stochastic simulation algorithm (e.g. ssa_etl with a
    census interval) to simulate individual cells
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-24152-2
    title: Spearheading future omics analyses using dyngen, a multi-modal simulator
      of single cells
  source_content_hash: sha256:4aa811133592c8dcba98e0a9f67884152c8965c487066b7c9e965a423f656e81
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dynverse/dyngen
  tools:
  - GillespieSSA2
  - dyngen
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# Gillespie SSA cell simulation

## Summary

Simulate individual cells at single-transcript resolution by running Gillespie's stochastic simulation algorithm (SSA) over reaction sets (transcription, splicing, translation, degradation) derived from a gene regulatory network, as implemented in dyngen via GillespieSSA2.

## When to use

Use when dynamic cellular processes must be simulated in silico from a known GRN with ground-truth molecule counts (pre-mRNA, mRNA, protein) in the low-copy-number regime typical of single-cell omics.

## When NOT to use

- When simulating bulk data with high molecule counts, where ODE-based models are appropriate and SSA stochasticity is unnecessary
- When the goal is only technical protocol emulation without cell dynamics (e.g. Splatter-style simulation of library-size and dropout properties)
- When spatial or inter-cellular effects matter — dyngen assumes cells are well-mixed and independent from each other

## Inputs

- Gene regulatory network (TFs, target subnetwork, housekeeping subnetwork) from a dyngen module-network backbone
- Reaction network: transcription, splicing, translation, and degradation reactions with Hill-model parameters (ba, sy)
- Simulation configuration: burn-phase modules, census interval, ssa_etl algorithm settings, sampling mode (snapshot or time series)
- Number of cells N to sample per dataset

## Outputs

- Per-cell, per-gene abundance trajectories of pre-mRNA (xG), mature mRNA (yG), and protein (zG)
- Sampled cell states with progression times along the ground-truth trajectory
- Molecule abundances passed on to the experiment-emulation step for generating simulated count matrices (e.g. dyngen_simulated_counts.csv)

## How to apply

First convert the gene regulatory network into a set of reactions per gene (transcription, splicing, translation, degradation), where the transcription propensity follows a thermodynamic Hill model parameterized by basal expression ba in [0,1] and synergism sy in [0,1]. Then run Gillespie's SSA through GillespieSSA2 (e.g. the ssa_etl algorithm with a census interval) on each cell, choosing SSA because ODE solvers are ill-suited for low-molecule simulations. Track the abundance of pre-mRNA (xG), mature mRNA (yG), and protein (zG) for every gene at every simulation state. After the burn phase (modules marked 'active during the burn phase' generate initial expression), sample N cells either as an unsynchronised snapshot (N_i = N/(L_i/sum L_i) per transition edge) or as a time series (e.g. k=8 intervals with gap g=0.75). The rationale is that SSA yields exact stochastic trajectories of molecule counts that downstream experiment emulation can convert into realistic count matrices.

## Related tools

- **dyngen** (Orchestrates the simulation pipeline: GRN-to-reaction conversion, SSA-based cell simulation, sampling, and experiment emulation) — https://github.com/dynverse/dyngen
- **GillespieSSA2** (Executes Gillespie's stochastic simulation algorithm (e.g. ssa_etl with a census interval) to simulate individual cells)

## Examples

```
library(GillespieSSA2); sim <- simulate_cells(model, num_simulations = 1000, census_interval = 1, ssa_algorithm = "ssa_etl")
```

## Evaluation signals

- The SSA output contains pre-mRNA (xG), mature mRNA (yG), and protein (zG) abundances for every gene at each simulated state
- After sampling, the dataset contains the requested number of cells and genes with a ground-truth trajectory and progression times
- Downstream simulated counts match the reference dataset's library size, fraction of zeros, and mean-variance behaviour as assessed by countsimQC (Supplementary Note 1)
- The chosen SSA algorithm is a low-molecule-capable exact method (e.g. ssa_etl with a census interval), not a deterministic ODE solver

## Limitations

- Cells are assumed to be well-mixed and independent from each other, excluding spatial and inter-cellular signalling effects
- SSA is needed because ODEs are ill-suited for performing low-molecule simulations, so the step is computationally heavier than deterministic simulation
- No software version numbers are reported for dyngen or GillespieSSA2 in the manuscript, complicating exact reproduction

## Evidence

- [intro] SSA designed for low-molecule simulations: [evidence span withheld — anchor status: not_found]
- [methods] dyngen uses Gillespie SSA: [evidence span withheld — anchor status: not_found]
- [methods] Reactions derived from GRN: "For every gene in the GRN a set of reactions are defined, namely transcription, splicing, translation, and degradation."
- [readme] dyngen multi-modal simulation engine: "dyngen is a novel, multi-modal simulation engine for studying dynamic cellular processes at single-cell resolution."
- [methods] Experiment emulation from SSA abundances: "From the SSA simulation we obtain the abundance levels of all the molecules at every state."
