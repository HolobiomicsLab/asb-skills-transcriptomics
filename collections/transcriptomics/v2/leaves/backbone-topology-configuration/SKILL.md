---
name: backbone-topology-configuration
description: Use when generating dyngen simulation datasets and a specific trajectory
  topology (linear, bifurcating, cyclic, converging, disconnected) must be configured
  to benchmark trajectory, RNA velocity, or CSNI methods.
when_to_use_negative:
- The input is real experimental single-cell data rather than a simulation requirement
  — dyngen backbones only define synthetic trajectory topologies
- A custom, non-predefined topology is needed — in that case construct a custom backbone
  instead of using the predefined backbone_* constructors
- Benchmarking methods that do not depend on trajectory structure (e.g. purely static
  differential expression), where backbone choice is not the controlling variable
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: dyngen
  role: Simulation engine whose predefined backbone constructors (backbone_bifurcating,
    backbone_bifurcating_cycle, backbone_converging, etc.) define the module network
    and trajectory topology of each simulated dataset
  repo: https://github.com/dynverse/dyngen
- name: GillespieSSA2
  role: Executes the Gillespie SSA cell simulation step downstream of backbone configuration,
    within generate_dataset()
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-24152-2
    title: Spearheading future omics analyses using dyngen, a multi-modal simulator
      of single cells
  source_content_hash: sha256:22b58b81de3888d11259ec537f1ab3527dcdf7c60d730a19178ad3ccb704330c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dynverse/dyngen
  tools:
  - GillespieSSA2
  - dyngen
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

# backbone-topology-configuration

## Summary

Configure dyngen's predefined trajectory backbones (linear, bifurcating, cyclic, converging, disconnected, etc.) to control the topology of simulated single-cell datasets used for benchmarking RNA velocity, CSNI, and trajectory alignment methods with full ground truth.

## When to use

Use when generating dyngen simulation datasets and a specific trajectory topology (linear, bifurcating, cyclic, converging, disconnected) must be configured to benchmark trajectory, RNA velocity, or CSNI methods.

## When NOT to use

- The input is real experimental single-cell data rather than a simulation requirement — dyngen backbones only define synthetic trajectory topologies
- A custom, non-predefined topology is needed — in that case construct a custom backbone instead of using the predefined backbone_* constructors
- Benchmarking methods that do not depend on trajectory structure (e.g. purely static differential expression), where backbone choice is not the controlling variable

## Inputs

- Choice of predefined dyngen backbone (e.g. linear, bifurcating, cyclic, converging, disconnected via backbone_* constructors)
- Module network definition embedded in the backbone
- GRN generation parameters (e.g. max in-degree = 5, regulators sampled from FANTOM5 GRNs)
- Random seeds (3 per backbone in the manuscript benchmark)

## Outputs

- Initialised dyngen model objects with the desired trajectory topology
- Complete simulated datasets (42 = 14 backbones x 3 seeds) each containing a transcriptomics count matrix
- Ground-truth trajectory with progression times per dataset
- Cell-specific ground-truth regulatory networks with regulatory effects bounded in [-1, 1]

## How to apply

Start a dyngen simulation by defining a module network, which the backbone constructor encapsulates (e.g. backbone_bifurcating, backbone_bifurcating_cycle, backbone_converging); the backbone determines how transcription factors are wired and how modules transition through states. Pass the backbone to initialise_model() together with default GRN generation parameters (e.g. max in-degree 5 for target regulators sampled from FANTOM5 GRNs) and default kinetics. Run generate_dataset() per backbone per random seed to execute the full pipeline: gold-standard simulation, GillespieSSA2 Gillespie SSA cell simulation, ground-truth trajectory mapping by expression correlation, per-cell regulatory-effect computation (bounded in [-1, 1]), and experiment emulation from real reference data. The manuscript benchmark combined 14 predefined backbones x 3 seeds into 42 datasets on which velocyto and scvelo (stochastic and dynamical) were evaluated with velocity correlation and velocity arrow cosine metrics. The dyngen_manuscript repository's analysis/showcase_backbones script generates visualisations of every predefined backbone, useful to confirm the chosen topology.

## Related tools

- **dyngen** (Simulation engine whose predefined backbone constructors (backbone_bifurcating, backbone_bifurcating_cycle, backbone_converging, etc.) define the module network and trajectory topology of each simulated dataset) — https://github.com/dynverse/dyngen
- **GillespieSSA2** (Executes the Gillespie SSA cell simulation step downstream of backbone configuration, within generate_dataset())

## Examples

```
model <- initialise_model(backbone = backbone_bifurcating()); dataset <- generate_dataset(model, seed = 1)
```

## Evaluation signals

- Exactly 42 complete datasets are produced for the full manuscript benchmark (14 predefined backbones x 3 random seeds)
- Each generated dataset contains all three modalities: count matrix, ground-truth trajectory with progression times, and cellwise GRN with regulatory effects bounded in [-1, 1]
- Backbone visualisations (via analysis/showcase_backbones) match the intended topology before running the pipeline
- Downstream velocity evaluation yields finite velocity correlation and velocity arrow cosine scores for velocyto and scvelo (stochastic and dynamical settings)
- Each backbone's default module network and GRN parameters (e.g. max in-degree 5 from FANTOM5 regulators) are retained unless deliberately overridden

## Limitations

- dyngen assumes cells are well-mixed and independent from each other
- Only the predefined backbones shipped with the package cover the manuscript's topology variety; custom topologies require manual backbone construction
- Experiment emulation depends on a real reference single-cell RNA-seq dataset (e.g. the MCA bone-marrow/mesenchyme/erythrocyte differentiation dataset from Zenodo), so simulated counts inherit reference-specific properties

## Evidence

- [methods] module network is entry point: "To start a dyngen simulation, the user needs to define a module network."
- [methods] GRN generated from module network in four steps: "The GRN is generated based on the given module network in four main steps (Supplementary Fig. 6)."
- [methods] SSA simulation step: [evidence span withheld — anchor status: not_found]
- [results] 42-dataset velocity benchmark: "We executed velocyto and scvelo (with 2 different parameter settings, stochastic and dynamical) on 42 datasets"
- [readme] showcase backbones script: [evidence span withheld — anchor status: external_doc]
