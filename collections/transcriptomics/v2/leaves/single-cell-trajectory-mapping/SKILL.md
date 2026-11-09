---
name: single-cell-trajectory-mapping
description: Use when new or perturbed cells (e.g. Gfi1-/-, Irf8-/-, double-KO GMP
  scRNA-seq count matrices) must be assigned to branches of an existing wild-type
  reference trajectory to quantify fate-biased allocation.
when_to_use_negative:
- When the new cells have totally different fates compared to the ones present in
  the reference structure — the article states the mapping procedure may not be optimal
  in that case
- When no reference trajectory exists and the goal is to reconstruct a trajectory
  de novo from a single pooled dataset (use the STREAM reconstruction/bifurcation
  workflow instead)
- When the expected topology is cyclic or disconnected — the article notes STREAM
  is not yet adapted to properly describe these more general structures
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: STREAM
  role: Provides the mapping feature that projects new cells onto the fixed reference
    elastic principal graph using MLLE-based neighborhood weights, without re-fitting
    the trajectory
  repo: https://github.com/pinellolab/STREAM
- name: ElPiGraph
  role: Core algorithm that learned the reference branching principal graph onto which
    the new cells are mapped
  repo: sysbio-curie/ElPiGraph.R
- name: anndata
  role: Data class in which both the wild-type reference (with its learned graph)
    and the mutant count matrices are held for mapping
- name: conda
  role: Used to install the STREAM reference implementation (stream=1.0 from Bioconda)
    in the env_stream environment
provenance:
  source_task_ids:
  - task_006
  - task_005
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:94ed79ed32a03bd6fe99d7ec222f649096376b65cf95aaedc720f6a768c4ba7e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pinellolab/STREAM
  tools:
  - ElPiGraph
  - Jupyter Notebook
  - Monocle2
  - Python 3
  - STREAM
  - anndata
  - conda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# single-cell-trajectory-mapping

## Summary

Project new (e.g. genetically perturbed) single-cell populations onto a previously learned STREAM elastic principal graph without pooling cells or re-fitting the trajectory, using STREAM's mapping feature with MLLE-based neighborhood weights.

## When to use

Use when new or perturbed cells (e.g. Gfi1-/-, Irf8-/-, double-KO GMP scRNA-seq count matrices) must be assigned to branches of an existing wild-type reference trajectory to quantify fate-biased allocation.

## When NOT to use

- When the new cells have totally different fates compared to the ones present in the reference structure — the article states the mapping procedure may not be optimal in that case
- When no reference trajectory exists and the goal is to reconstruct a trajectory de novo from a single pooled dataset (use the STREAM reconstruction/bifurcation workflow instead)
- When the expected topology is cyclic or disconnected — the article notes STREAM is not yet adapted to properly describe these more general structures

## Inputs

- anndata object with wild-type reference scRNA-seq counts and the precomputed STREAM/ElPiGraph elastic principal graph
- scRNA-seq count matrices for the cells to be mapped (e.g. Gfi1-/-, Irf8-/-, Gfi1-/-Irf8-/- GMP populations) loaded as anndata objects
- Number of PCs for mapping (top_pcs feature) and MLLE neighborhood weighting settings

## Outputs

- Per-cell branch assignments of the mapped cells onto the reference graph
- Per-branch allocation summary table (fraction of cells of each genotype on each reference branch, fractions summing to 1)
- Visualization of the mapped mutant cells overlaid on the reference branch structure

## How to apply

First learn the reference trajectory: load the wild-type Olsson et al. scRNA-seq data (LSK, CMP, GMP, LKCD34+ subpopulations) into an anndata object, run STREAM (whose core algorithm is ElPiGraph) to obtain the elastic principal graph showing the HSCP → erythroid/Multi-Lin → Gran/Mono bifurcation, without supplying time points, start cell, or number of branching events. Then load each mutant count matrix into a separate anndata object and execute STREAM's mapping feature (tutorial 3-STREAM_mapping.ipynb, v1.0 redesigned mapping; neighborhood weighting via MLLE, with the `top_pcs` feature controlling which PCs are used) to project cells onto the fixed reference graph. Do NOT pool the new cells with the reference cells or recompute the graph — the point of the mapping procedure is that it is deterministic and reuses the reference structure. Tabulate the fraction of cells from each genotype assigned to each reference branch and verify the fractions sum to 1 per genotype. Visualize the mapped cells on the reference branch structure to confirm the expected allocation pattern.

## Related tools

- **STREAM** (Provides the mapping feature that projects new cells onto the fixed reference elastic principal graph using MLLE-based neighborhood weights, without re-fitting the trajectory) — https://github.com/pinellolab/STREAM
- **ElPiGraph** (Core algorithm that learned the reference branching principal graph onto which the new cells are mapped) — sysbio-curie/ElPiGraph.R
- **anndata** (Data class in which both the wild-type reference (with its learned graph) and the mutant count matrices are held for mapping)
- **conda** (Used to install the STREAM reference implementation (stream=1.0 from Bioconda) in the env_stream environment)

## Examples

```
conda create -n env_stream python=3.7 stream=1.0 jupyter; conda activate env_stream; jupyter notebook  # then run 3-STREAM_mapping.ipynb to project mutant anndata onto the fixed wild-type STREAM graph
```

## Evaluation signals

- The allocation table exists and contains all three genotypes (Gfi1-/-, Irf8-/-, Gfi1-/-Irf8-/-) with per-branch fractions summing to 1
- The mapped-cell visualization on the reference graph reproduces the expected pattern: Gfi1-/- biased to the monocyte branch, Irf8-/- to the granulocyte branch, and the double KO balanced across the diverging branches
- The reference STREAM/ElPiGraph graph was NOT re-fitted — mapping must be executed against the fixed, precomputed wild-type trajectory
- Neighborhood weights for the mapping are MLLE-based and the top_pcs feature used for mapping is explicitly set/recorded
- The reference trajectory topology matches the known HSCP → erythroid/Multi-Lin → Gran/Mono hierarchy with branch-specific markers (Gfi1/Irf8 on Gran/Mono, Gata1 on erythroid) before mapping

## Limitations

- The mapping procedure may not be optimal if the new cells have totally different fates compared to the ones present in the reference structure
- STREAM is not yet adapted to properly describe cyclic or disconnected graph topologies, so mapping onto such structures is not supported
- Scalability to very large datasets (millions of cells) is not established for STREAM

## Evidence

- [intro] no existing method provides mapping without re-fitting: "no trajectory inference method provides the possibility to map new cells to previously obtained reference trajectories without pooling cells and re-computing trajectories."
- [discussion] mapping limitation for divergent fates: "However, the mapping procedure may not be optimal if the new cells have totally different fates compared to the ones present in the reference structure"
- [methods] mapping tutorial notebook: "Example for *mapping* feature: [3-STREAM_mapping.ipynb]"
- [readme] redesigned mapping in v1.0: "redesigned all plotting-related functions
5) redesigned *mapping* procedure"
- [readme] top_pcs feature for Mapping: "We added support of feature `top_pcs` for *Mapping*"
- [methods] environment setup for STREAM: "$ conda create -n env_stream python=3.7 stream=1.0 jupyter"
