---
name: seed-controlled-in-silico-dataset-generation
description: Use when a benchmark needs multiple simulated single-cell datasets with
  known ground truth (trajectory, progression times, cell-specific GRNs) across diverse
  backbone topologies and reproducible seeds.
when_to_use_negative:
- The analysis requires real biological data rather than synthetic data with known
  ground truth (dyngen simulations assume well-mixed, independent cells and do not
  capture inter-cellular dependencies).
- The target method needs simulator features dyngen does not provide, e.g. benchmarking
  of clustering or differential-expression methods for which protocol-specific simulators
  such as Splatter, powsimR, PROSSTT, or SymSim were designed.
- Only a single arbitrary topology is needed without controlled backbone variety or
  seed-level reproducibility — a simpler one-off simulation suffices.
edam_operation: http://edamontology.org/operation_0284
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: dyngen
  role: 'Primary simulation engine: backbone constructors, initialise_model(), and
    generate_dataset() producing counts plus ground-truth trajectory and cellwise
    GRN'
  repo: https://github.com/dynverse/dyngen
- name: GillespieSSA2
  role: Executes the Gillespie stochastic simulation algorithm (SSA) step inside generate_dataset()
    to simulate individual cells at single-transcript resolution
- name: dyngen_manuscript
  role: Repository with reproduction scripts (analysis/usecase_rna_velocity, usecase_network_inference,
    usecase_trajectory_alignment) that generate and consume the simulated datasets
  repo: https://github.com/dynverse/dyngen_manuscript
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-24152-2
    title: Spearheading future omics analyses using dyngen, a multi-modal simulator
      of single cells
  source_content_hash: sha256:feb33b3505c80a61ecb286c10f38f09c981a4546c634bda7b882c0bfb826e0ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dynverse/dyngen
  tools:
  - GillespieSSA2
  - dyngen
  - dyngen_manuscript
  edam_operation: http://edamontology.org/operation_0284
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Seed-controlled in silico dataset generation

## Summary

Generate reproducible simulated single-cell datasets with dyngen by enumerating predefined trajectory backbones, instantiating each backbone into a model, and running generate_dataset() over fixed random seeds so every dataset carries its transcriptomic count matrix plus ground-truth trajectory and cellwise gene regulatory network.

## When to use

Use when a benchmark needs multiple simulated single-cell datasets with known ground truth (trajectory, progression times, cell-specific GRNs) across diverse backbone topologies and reproducible seeds.

## When NOT to use

- The analysis requires real biological data rather than synthetic data with known ground truth (dyngen simulations assume well-mixed, independent cells and do not capture inter-cellular dependencies).
- The target method needs simulator features dyngen does not provide, e.g. benchmarking of clustering or differential-expression methods for which protocol-specific simulators such as Splatter, powsimR, PROSSTT, or SymSim were designed.
- Only a single arbitrary topology is needed without controlled backbone variety or seed-level reproducibility — a simpler one-off simulation suffices.

## Inputs

- Choice of predefined dyngen backbone constructor (e.g. backbone_bifurcating, backbone_bifurcating_cycle, backbone_converging)
- Random seed(s), e.g. 3 seeds per backbone
- Module network / GRN generation parameters (default max in-degree x = 5, TF interactions sampled from FANTOM5 GRNs)
- Real reference single-cell RNA-seq dataset for experiment emulation (e.g. zenodo_1443566_real_silver_bone-marrow-mesenchyme-erythrocyte-differentiation_mca)

## Outputs

- Per-dataset transcriptomics count matrix (simulated spliced/unspliced molecule abundances)
- Ground-truth trajectory with per-cell progression times and cell states
- Cell-specific ground-truth gene regulatory network with regulatory effects bounded in [-1, 1]
- A collection of e.g. 42 complete datasets (14 backbones x 3 seeds) for benchmarking

## How to apply

Enumerate the predefined dyngen backbone constructors (linear, bifurcating, bifurcating_cycle, cyclic, converging, disconnected, etc.; 14 in the manuscript benchmark). For each backbone, call initialise_model() with the backbone's default module network, sampling target-gene regulators from FANTOM5 GRNs with a max in-degree of 5, and default kinetics. Run the full pipeline via generate_dataset(), which internally performs gold-standard simulation, a Gillespie SSA cell simulation (GillespieSSA2), ground-truth trajectory mapping by expression correlation, per-cell regulatory-effect computation, and experiment emulation of single-cell omics profiling using real reference datasets (e.g. the MCA bone-marrow/mesenchyme/erythrocyte dataset from Zenodo). Repeat for fixed random seeds — the manuscript used 3 seeds per backbone to yield exactly 42 datasets. Store, per dataset, the count matrix, ground-truth trajectory with progression times, and cell-specific GRN with regulatory effects bounded in [-1, 1], so downstream methods (RNA velocity, CSNI, trajectory alignment) can be scored against complete ground truth.

## Related tools

- **dyngen** (Primary simulation engine: backbone constructors, initialise_model(), and generate_dataset() producing counts plus ground-truth trajectory and cellwise GRN) — https://github.com/dynverse/dyngen
- **GillespieSSA2** (Executes the Gillespie stochastic simulation algorithm (SSA) step inside generate_dataset() to simulate individual cells at single-transcript resolution)
- **dyngen_manuscript** (Repository with reproduction scripts (analysis/usecase_rna_velocity, usecase_network_inference, usecase_trajectory_alignment) that generate and consume the simulated datasets) — https://github.com/dynverse/dyngen_manuscript

## Examples

```
model <- initialise_model(backbone = backbone_bifurcating()); dataset <- generate_dataset(model, seed = 1)
```

## Evaluation signals

- Exactly 42 complete datasets are produced when following the manuscript design (14 backbones x 3 seeds), each with a count matrix plus trajectory and cellwise GRN annotations.
- Every dataset contains per-cell ground-truth regulatory effects bounded within [-1, 1].
- Ground-truth trajectory progression times are present and consistent with the backbone topology (linear, bifurcating, cyclic, converging, disconnected).
- Re-running the same seed/backbone combination reproduces an identical dataset (seed-level determinism of initialise_model/generate_dataset).
- The experiment-emulation step applied a real reference dataset (e.g. the MCA bone-marrow/mesenchyme/erythrocyte dataset) rather than purely synthetic noise profiles.

## Limitations

- One major assumption of dyngen is that cells are assumed to be well-mixed and independent from each other.
- No software version numbers are reported for dyngen, GillespieSSA2, velocyto, or scVelo, which complicates exact reproduction.
- The reference single-cell RNA-seq dataset repository used for experiment emulation is not explicitly named in the manuscript methods.

## Evidence

- [methods] Full pipeline via generate_dataset covering SSA simulation and ground truth: [evidence span withheld — anchor status: not_found]
- [results] 42 datasets benchmarked for RNA velocity: "We executed velocyto and scvelo (with 2 different parameter settings, stochastic and dynamical) on 42 datasets"
- [methods] FANTOM5-sampled target-gene interactions: "To sample the interactions between target genes, one of the many FANTOM545 GRNs is sampled."
- [readme] Manuscript reproduction code location: "All source code for reproducing the results in this manuscript are available on [GitHub](https://github.com/dynverse/dyngen_manuscript)."
- [discussion] Well-mixed independent cells assumption: "One major assumption of dyngen is that cells are assumed to be well-mixed and independent from each other"
