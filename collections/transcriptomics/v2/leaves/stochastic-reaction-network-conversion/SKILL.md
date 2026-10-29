---
name: stochastic-reaction-network-conversion
description: Use when a defined gene regulatory network must be turned into stochastic
  biochemical reactions to simulate single-cell, single-transcript resolution dynamics
  with known ground truth.
when_to_use_negative:
- The goal is to emulate only the technical sequencing protocol rather than underlying
  biological dynamics — the article notes such simulators' applicability and reusability
  is limited
- Data are bulk high-molecule measurements where ODE-based simulation is adequate;
  SSA is motivated by low library sizes and high dropout of single-cell data
- Cells must be modeled as spatially arranged or interacting — dyngen assumes cells
  are well-mixed and independent
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_3170
tools:
- name: dyngen
  role: Multi-modal simulation engine that translates the GRN into reactions and orchestrates
    the SSA-based single-cell simulation
  repo: https://github.com/dynverse/dyngen
- name: GillespieSSA2
  role: R package executing Gillespie's SSA (e.g. ssa_etl with a census interval)
    to simulate pre-mRNA, mRNA, and protein abundances
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-24152-2
    title: Spearheading future omics analyses using dyngen, a multi-modal simulator
      of single cells
  source_content_hash: sha256:a8ee8a0ba81b831b5068ca363948f9d27c554ab38d88097083f3be1a9d50f0a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dynverse/dyngen
  tools:
  - GillespieSSA2
  - dyngen
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Stochastic Reaction Network Conversion

## Summary

Converts a gene regulatory network into a set of biochemical reactions (transcription, splicing, translation, degradation) and simulates single-cell molecule counts with Gillespie's stochastic simulation algorithm, as implemented in dyngen's simulation pipeline.

## When to use

Use when a defined gene regulatory network must be turned into stochastic biochemical reactions to simulate single-cell, single-transcript resolution dynamics with known ground truth.

## When NOT to use

- The goal is to emulate only the technical sequencing protocol rather than underlying biological dynamics — the article notes such simulators' applicability and reusability is limited
- Data are bulk high-molecule measurements where ODE-based simulation is adequate; SSA is motivated by low library sizes and high dropout of single-cell data
- Cells must be modeled as spatially arranged or interacting — dyngen assumes cells are well-mixed and independent

## Inputs

- dyngen module network backbone (e.g. backbone_bifurcating_loop) with basal expression, burn-phase activity flags, and module interaction strengths
- FANTOM5 GRN used to sample target-gene interactions (max 5 regulators per target)
- Reaction parameters: basal expression ba in [0,1], synergism sy in [0,1]

## Outputs

- Per-gene reaction set (transcription, splicing, translation, degradation) with Hill-model transcription propensities
- SSA-simulated molecule abundances per gene per cell: pre-mRNA xG, mature mRNA yG, protein zG
- Simulated single-cell count matrix (e.g. dyngen_simulated_counts.csv)

## How to apply

First define a module network backbone (e.g. backbone_bifurcating_loop) specifying basal expression, burn-phase activity, and up/down-regulating interactions with strengths between modules, then generate the feature network by assigning TFs (at least 1 TF per module), inheriting TF interactions from the backbone, and sampling target and housekeeping subnetworks from a FANTOM5 GRN with maximum in-degree 5. Convert the GRN into reactions for every gene — transcription, splicing, translation, and degradation — parameterizing transcription propensity with a thermodynamic Hill model using basal expression ba in [0,1] and synergism sy in [0,1]. Simulate individual cells with Gillespie's SSA via GillespieSSA2 (e.g. ssa_etl with a census interval), because SSA is designed to work well in low-molecule simulations where ODEs are ill-suited given single-cell library sizes; track pre-mRNA (xG), mature mRNA (yG), and protein (zG) abundances for every gene. The assumption is that cells are well-mixed and independent of each other.

## Related tools

- **dyngen** (Multi-modal simulation engine that translates the GRN into reactions and orchestrates the SSA-based single-cell simulation) — https://github.com/dynverse/dyngen
- **GillespieSSA2** (R package executing Gillespie's SSA (e.g. ssa_etl with a census interval) to simulate pre-mRNA, mRNA, and protein abundances)

## Examples

```
install.packages("dyngen"); library(dyngen); backbone <- backbone_bifurcating_loop(); model <- initialise_model(backbone) # then generate_tf_network / generate_feature_network and simulate via GillespieSSA2
```

## Evaluation signals

- Output count matrix contains the requested number of cells and genes
- Simulated data show characteristic single-cell omics features — library size, fraction of zeros, mean-variance relationship — consistent with the chosen reference dataset as assessed by countsimQC (Supplementary Note 1)
- Every gene in the GRN has a complete reaction set (transcription, splicing, translation, degradation)
- Each module in the backbone has at least one assigned TF, and sampled target/housekeeping subnetworks respect the maximum in-degree of 5
- Tracked SSA state includes all three molecule species (xG pre-mRNA, yG mature mRNA, zG protein) per gene

## Limitations

- One major assumption of dyngen is that cells are assumed to be well-mixed and independent from each other
- ODE-based simulation is ill-suited for low-molecule single-cell conditions, motivating SSA but constraining scalability
- No software version numbers were reported for any tool, complicating reproducibility

## Evidence

- [intro] SSA motivation: "individual cells are simulated using Gillespie’s stochastic simulation algorithm (SSA)12, which is designed to work well in low-molecule simulations"
- [methods] reaction conversion: "For every gene in the GRN a set of reactions are defined, namely transcription, splicing, translation, and degradation."
- [methods] GRN generation steps: "The GRN is generated based on the given module network in four main steps (Supplementary Fig. 6)."
- [readme] README positioning: "dyngen is a novel, multi-modal simulation engine for studying dynamic
cellular processes at single-cell resolution."
