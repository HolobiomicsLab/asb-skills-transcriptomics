---
name: gene-regulatory-network-simulation
description: Use when benchmarking trajectory inference, RNA velocity, trajectory
  alignment, or cell-specific network inference methods requires simulated single-cell
  data with completely known ground truth for GRN dynamics and cellular progression.
when_to_use_negative:
- The analysis goal is benchmarking on real biological data without ground truth rather
  than synthetic data where 'the actual ground truth is completely known'
- The simulator must model cell-to-cell interactions or spatial effects — dyngen assumes
  'cells are assumed to be well-mixed and independent from each other'
- Only bulk omics data is needed — ODE-based bulk simulators are a different regime;
  dyngen targets low-molecule single-cell simulations via SSA
edam_operation: http://edamontology.org/operation_3947
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: dyngen
  role: 'Primary simulation engine: backbone/module-network definition, GRN generation,
    reaction conversion, and experiment emulation via generate_dataset()'
  repo: https://github.com/dynverse/dyngen
- name: GillespieSSA2
  role: Executes Gillespie's stochastic simulation algorithm for the low-molecule
    single-cell dynamics inside dyngen
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-24152-2
    title: Spearheading future omics analyses using dyngen, a multi-modal simulator
      of single cells
  source_content_hash: sha256:22c2dd9e6c8ca1189a1d8fe94f9c67bf64cf636569a251c3cd09aa0e00d120a8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dynverse/dyngen
  tools:
  - GillespieSSA2
  - dyngen
  edam_operation: http://edamontology.org/operation_3947
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# gene-regulatory-network-simulation

## Summary

Simulate dynamic single-cell gene regulatory network activity by generating a GRN from a module network/backbone, translating it into regulation, transcription, splicing, translation and degradation reactions, and running Gillespie's SSA to produce cells with ground-truth trajectory and cellwise GRN annotations (as implemented in dyngen).

## When to use

Use when benchmarking trajectory inference, RNA velocity, trajectory alignment, or cell-specific network inference methods requires simulated single-cell data with completely known ground truth for GRN dynamics and cellular progression.

## When NOT to use

- The analysis goal is benchmarking on real biological data without ground truth rather than synthetic data where 'the actual ground truth is completely known'
- The simulator must model cell-to-cell interactions or spatial effects — dyngen assumes 'cells are assumed to be well-mixed and independent from each other'
- Only bulk omics data is needed — ODE-based bulk simulators are a different regime; dyngen targets low-molecule single-cell simulations via SSA

## Inputs

- Module network definition or predefined dyngen backbone (e.g. backbone_bifurcating, backbone_bifurcating_cycle, backbone_converging)
- GRN generation parameters (e.g. max in-degree x = 5 for target regulators sampled from FANTOM5 GRNs)
- Random seed(s) for stochastic simulation
- Real reference single-cell RNA-seq dataset for experiment emulation

## Outputs

- Transcriptomics count matrix for simulated cells
- Ground-truth trajectory with progression times and cell states
- Cell-specific ground-truth regulatory network with regulatory effects bounded in [-1, 1]
- Molecule abundances (pre-mRNA, mRNA, protein) from the SSA simulation states

## How to apply

Define a module network or pick one of dyngen's 14 predefined backbones (e.g. linear, bifurcating, bifurcating_cycle, converging, cyclic, disconnected) via backbone constructors like backbone_bifurcating(). Generate the GRN from the module network in four steps (sampling TFs, generating TF interactions, sampling the target subnetwork with in-degree capped at ~5 from FANTOM5 GRNs, and sampling the housekeeping subnetwork), then convert the GRN into reactions for transcription, splicing, translation, and degradation. Simulate individual cells with Gillespie's SSA (GillespieSSA2), which is designed for low-molecule simulations, optionally including a burn phase for modules marked active during warm-up. Finally, emulate single-cell omics profiling using a real reference dataset (e.g. FANTOM5-sampled interactions or a downloaded real-counts dataset such as zenodo_1443566_real_silver_bone-marrow-mesenchyme-erythrocyte-differentiation_mca) and run generate_dataset() per seed. Store the count matrix, ground-truth trajectory with progression times, and per-cell regulatory network with regulatory effects bounded in [-1, 1].

## Related tools

- **dyngen** (Primary simulation engine: backbone/module-network definition, GRN generation, reaction conversion, and experiment emulation via generate_dataset()) — https://github.com/dynverse/dyngen
- **GillespieSSA2** (Executes Gillespie's stochastic simulation algorithm for the low-molecule single-cell dynamics inside dyngen)

## Examples

```
model <- initialise_model(backbone = backbone_bifurcating()); dataset <- generate_dataset(model, seed = 1)
```

## Evaluation signals

- Exactly 42 complete datasets are produced when running the benchmark design (14 predefined backbones x 3 seeds)
- Each dataset contains a count matrix plus ground-truth trajectory and cellwise GRN annotations as required for downstream method evaluation
- Regulatory effects in the cellwise ground-truth GRN are bounded within [-1, 1]
- The GRN generation completes all four steps: TF sampling, TF interactions per the backbone, target subnetwork sampling (max in-degree x = 5 from FANTOM5), and housekeeping subnetwork sampling
- Trajectory inference, RNA velocity (velocity correlation, velocity arrow cosine), and CSNI (mean AUROC/AUPR per cell) metrics can be computed against the stored ground truth

## Limitations

- One major assumption of dyngen is that cells are assumed to be well-mixed and independent from each other, so cell-cell communication and spatial structure are not modeled
- No software version numbers for dyngen or downstream tools are reported in the manuscript, complicating exact reproduction
- Existing single-cell simulators focused on technical protocol rather than underlying biology have limited reusability — dyngen addresses this but requires specifying backbone/module networks appropriate to the question

## Evidence

- [methods] Module network definition: "To start a dyngen simulation, the user needs to define a module network."
- [methods] SSA simulation: [evidence span withheld — anchor status: not_found]
- [results] RNA velocity benchmark scale: "We executed velocyto and scvelo (with 2 different parameter settings, stochastic and dynamical) on 42 datasets"
- [results] CSNI ground-truth evaluation: "For each dataset, we computed the mean Area Under the Receiver Operating Characteristic-curve (AUROC) and Area Under the Precision-Recall curve (AUPR) scores of the individual cells"
- [readme] Tool description from README: "dyngen is a novel, multi-modal simulation engine for studying dynamic cellular processes at single-cell resolution."
