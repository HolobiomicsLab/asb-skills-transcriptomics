---
name: gene-regulatory-network-backbone-design
description: Use when simulating single-cell dynamics with dyngen and the user must
  define or choose a module network backbone that drives the topology (e.g. bifurcation,
  loop) of the resulting gene regulatory network.
when_to_use_negative:
- The input is already a real single-cell count matrix and the goal is technical-protocol
  emulation rather than designing an underlying biological regulatory network (simulators
  like Splatter, powsimR, PROSSTT, or SymSim may be more appropriate)
- The backbone is fully predefined and the task is running the downstream SSA simulation
  or experiment emulation rather than specifying the module network itself
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: dyngen
  role: R package providing predefined backbones, backbone construction, and generate_feature_network
    to convert the module network into a GRN and reaction set
  repo: https://github.com/dynverse/dyngen
- name: GillespieSSA2
  role: Executes Gillespie's SSA on the reactions derived from the designed backbone,
    tracking pre-mRNA (xG), mature mRNA (yG), and protein (zG)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-24152-2
    title: Spearheading future omics analyses using dyngen, a multi-modal simulator
      of single cells
  source_content_hash: sha256:e5ea66b739534d53185a384916b82b65b747140689301bfe1f37e22a0fa54d2c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dynverse/dyngen
  tools:
  - GillespieSSA2
  - dyngen
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# gene-regulatory-network-backbone-design

## Summary

Design and configure a dyngen module network backbone (e.g. backbone_bifurcating_loop) that specifies basal expression, burn-phase activity, and up/down-regulating inter-module interactions, from which dyngen generates the full gene regulatory network for in silico single-cell simulation.

## When to use

Use when simulating single-cell dynamics with dyngen and the user must define or choose a module network backbone that drives the topology (e.g. bifurcation, loop) of the resulting gene regulatory network.

## When NOT to use

- The input is already a real single-cell count matrix and the goal is technical-protocol emulation rather than designing an underlying biological regulatory network (simulators like Splatter, powsimR, PROSSTT, or SymSim may be more appropriate)
- The backbone is fully predefined and the task is running the downstream SSA simulation or experiment emulation rather than specifying the module network itself

## Inputs

- Chosen or custom dyngen backbone object (e.g. backbone_bifurcating_loop) defining modules, basal expression, burn-phase activity, and up/down-regulating interactions with strengths
- One of the predefined FANTOM5 GRNs used to sample target-gene interactions
- User-defined numbers of TFs, target genes, and housekeeping genes

## Outputs

- dyngen feature network (GRN) with TF, target, and housekeeping subnetworks
- Reaction set per gene (transcription, splicing, translation, degradation) ready for Gillespie SSA simulation

## How to apply

Start a dyngen simulation by defining a module network backbone, such as backbone_bifurcating_loop, specifying for each module its basal expression level (ba in [0,1]), whether it is active during the initial warm-up phase (burn phase), and directed up- or down-regulating interactions with strengths between modules. Next, generate the feature network with generate_feature_network: sample transcription factors so each module has at least 1 TF, have TFs inherit their regulation from the backbone interactions, sample the target subnetwork from one of the FANTOM5 GRNs with at most 5 regulators per target, and sample a separate housekeeping subnetwork with maximum in-degree 5. The backbone then seeds conversion of the GRN into reactions (transcription, splicing, translation, degradation), with transcription propensity parameterized by a thermodynamic Hill model using the basal expression ba and synergism sy in [0,1]. Choose a backbone whose topology matches the biological process under study (e.g. bifurcating loop for lineage commitment dynamics), since downstream Gillespie SSA simulation and the resulting cell trajectories inherit the backbone structure.

## Related tools

- **dyngen** (R package providing predefined backbones, backbone construction, and generate_feature_network to convert the module network into a GRN and reaction set) — https://github.com/dynverse/dyngen
- **GillespieSSA2** (Executes Gillespie's SSA on the reactions derived from the designed backbone, tracking pre-mRNA (xG), mature mRNA (yG), and protein (zG))

## Examples

```
library(dyngen); backbone <- backbone_bifurcating_loop(); model <- initialise_model(backbone, num_tfs = 20, num_targets = 100, num_hks = 100); model <- generate_feature_network(model)
```

## Evaluation signals

- Every module in the backbone has at least 1 assigned transcription factor after generate_feature_network
- Target genes in the GRN have at most 5 regulators per target, matching the FANTOM5 sampling rule
- The separate housekeeping subnetwork has maximum in-degree 5 and is disconnected from TFs and target genes
- Basal expression ba and synergism sy parameters lie within [0,1] and the resulting transcription propensities are finite and positive
- Modules marked as burn-phase active actually generate expression during the initial warm-up phase of the SSA simulation

## Limitations

- dyngen assumes cells are well-mixed and independent from each other, which the design of the backbone cannot relax
- The resulting simulation is a stochastic approximation; backbone topology choices (e.g. bifurcating loop) impose structure on trajectories rather than deriving it from real data

## Evidence

- [methods] Module network definition: "To start a dyngen simulation, the user needs to define a module network."
- [methods] Burn phase semantics: "A module marked as “active during the burn phase” means that this module will be allowed to generate expression of its genes during an initial warm-up phase."
- [methods] Target subnetwork sampling: "To sample the interactions between target genes, one of the many FANTOM545 GRNs is sampled."
- [methods] GRN to reactions: "For every gene in the GRN a set of reactions are defined, namely transcription, splicing, translation, and degradation."
- [intro] Simulation engine: "Second, individual cells are simulated using Gillespie’s stochastic simulation algorithm (SSA)12, which is designed to work well in low-molecule simulations"
- [readme] Backbone showcase in README: "Showcase different
  backbones"
