---
name: subpopulation-defined-trajectory-endpoint-selection
description: Use when reconstructing single-cell trajectories between biologically
  defined cell states (e.g., CMP to erythroid vs GMP in Paul et al. GSE72857) and
  prior algorithms cannot target arbitrary endpoints.
when_to_use_negative:
- When the goal is a purely linear pseudotime ordering over the whole dataset without
  specified endpoints — Monocle 1 or Waterfall reconstruct linear trajectories
- When the number of branches is known and a global branching model suffices — DPT
  requires a priori knowledge of the number of branches, which may be acceptable in
  that setting
- When no meaningful subpopulation labels exist and endpoints cannot be defined biologically
edam_operation: http://edamontology.org/operation_2437
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CellRouter
  role: Implements the flow-network algorithm that reconstructs multi-state trajectories
    between any two chosen subpopulation endpoints on a subpopulation-aware graph
  repo: https://github.com/edroaldo/cellrouter
- name: R
  role: Implementation environment; CellRouter class is sourced in R with the libdir
    variable pointing to the Java libraries
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-03214-y
    title: Reconstruction of complex single-cell trajectories using CellRouter
  source_content_hash: sha256:0f258f5d9ec807f1cc9603f9b6dede6fca5403712aa2ed1009cb9598cbad2181
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edroaldo/cellrouter
  tools:
  - CellRouter
  - Monocle 2
  - R
  - Waterfall
  - Wishbone
  edam_operation: http://edamontology.org/operation_2437
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Subpopulation-defined trajectory endpoint selection

## Summary

Selects trajectory start and end states as named subpopulations (clusters or user-supplied metadata labels) on a subpopulation-aware graph, letting CellRouter's flow-network algorithm reconstruct multi-state transitions between any two chosen cell states, including intermediate or rare ones.

## When to use

Use when reconstructing single-cell trajectories between biologically defined cell states (e.g., CMP to erythroid vs GMP in Paul et al. GSE72857) and prior algorithms cannot target arbitrary endpoints.

## When NOT to use

- When the goal is a purely linear pseudotime ordering over the whole dataset without specified endpoints — Monocle 1 or Waterfall reconstruct linear trajectories
- When the number of branches is known and a global branching model suffices — DPT requires a priori knowledge of the number of branches, which may be acceptable in that setting
- When no meaningful subpopulation labels exist and endpoints cannot be defined biologically

## Inputs

- Preprocessed scRNA-seq count matrix (raw count/UMI data supported per README update), e.g., Paul et al. GSE72857 mouse myeloid progenitor data
- Subpopulation assignments from CellRouter community detection, or user-defined cluster labels / sorted-population metadata
- Named start and target subpopulation identifiers (e.g., subpopulation 20 as source; subpopulations 13 and 9 as targets)

## Outputs

- Multi-state transition trajectories between the chosen subpopulation pairs
- Per-trajectory cell orderings and gene expression dynamics along each trajectory progression

## How to apply

First identify subpopulations by combining learning the structure of the data with community detection algorithms (or supply user-defined clusters/sorted-population metadata, as supported in the May 2018 CellRouter update). Then designate a start subpopulation and one or more target subpopulations — e.g., in the Paul et al. myeloid progenitor tutorial, trajectories are reconstructed from the CMP subpopulation (subpopulation 20) to the erythroid (subpopulation 13) and GMP (subpopulation 9) subpopulations. CellRouter's flow-network algorithm then finds high-resolution paths between these endpoints on the subpopulation-aware graph, requiring no a priori knowledge of the number of branches or fates. Because endpoints are explicit, you can deliberately query transitions from early states to intermediate or transitory states, which is where Monocle 2, DPT, Wishbone, and Waterfall are less robust.

## Related tools

- **CellRouter** (Implements the flow-network algorithm that reconstructs multi-state trajectories between any two chosen subpopulation endpoints on a subpopulation-aware graph) — https://github.com/edroaldo/cellrouter
- **R** (Implementation environment; CellRouter class is sourced in R with the libdir variable pointing to the Java libraries)

## Examples

```
source('path/to/CellRouter_Class.R'); libdir <- 'path/to/CellRouter/'
```

## Evaluation signals

- The chosen endpoints resolve to the intended subpopulations (e.g., CMP = subpopulation 20, erythroid = 13, GMP = 9 in the Paul et al. tutorial) and trajectories connect them
- Reconstructed paths pass through intermediate/transitory cell states rather than only mature types, since trajectories are identified from any subpopulation to any other
- Trajectory smoothness metrics (significantly correlated genes and lag-1 autocorrelation of branch markers such as Klf1, Car2, Cited4 and Mpo, Prtn3, Ctsg) support robust reconstruction from early states
- Running CellRouter with user-defined labels (e.g., Paul et al. cluster labels as subpopulation annotations) reproduces comparable endpoint-to-endpoint trajectories, confirming user-supplied metadata is honored

## Limitations

- CellRouter was tested only in Unix-based systems and was not tested in Windows, per the README
- Trajectory quality depends on subpopulation identification; endpoints must correspond to real subpopulations or valid user-defined metadata
- Requires sourcing the CellRouter class with libdir correctly set to the folder containing the Java libraries inside 'CellRouter'

## Evidence

- [methods] High-resolution multi-state trajectories are identified from any subpopulation to any other, including intermediate and mature cell types: "High-resolution multi-state trajectories are identified from any subpopulation to any other, including intermediate and mature cell types"
- [intro] Monocle 2 not designed for arbitrary endpoints: "Monocle 2 addresses many of these challenges but is not designed to reconstruct trajectories between any two chosen cell states"
- [methods] Subpopulations identified via structure learning and community detection: "Subpopulations are then identified by a combination of learning the structure of the data and community detection algorithms"
- [readme] User-defined clusters or sorted-population metadata can serve as subpopulations: "Enhanced flexibility for trajectory reconstruction (User-defined clusters or other metadata can be used, such as sorted populations)"
- [abstract] Flow-network reconstruction of multi-state trajectories: "identifies complex cell-state transition trajectories by using flow networks to explore the subpopulation structure of multi-dimensional, single-cell omics data"
