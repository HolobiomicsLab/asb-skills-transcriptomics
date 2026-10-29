---
name: pseudotime-branch-visualization
description: Use when scRNA-seq cells from a new condition (e.g., Gfi1-/-, Irf8-/-,
  double-knockout GMPs) must be projected onto an existing wild-type branching trajectory
  to read off branch-level fate allocation.
when_to_use_negative:
- The query cells have totally different fates from any present in the reference structure
  — the mapping procedure may not be optimal in that case.
- You need to learn a new trajectory from scratch rather than project cells onto an
  existing one — use STREAM's trajectory reconstruction workflow instead of the mapping
  feature.
- The data are cyclic or disconnected graph topologies — the paper notes STREAM requires
  further improvement to properly describe these more general structures.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STREAM
  role: Provides the mapping feature that projects query cells onto a precomputed
    elastic-principal-graph reference trajectory and the density-level branch visualization
  repo: https://github.com/pinellolab/STREAM
- name: ElPiGraph
  role: Core algorithm behind STREAM that learns the complex elastic principal graph
    on which cells are mapped and branches visualized
  repo: sysbio-curie/ElPiGraph.R
- name: anndata
  role: Data class in which STREAM stores reference and query scRNA-seq datasets for
    mapping
- name: Python 3
  role: Runtime environment for STREAM's mapping and visualization notebooks
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:e70cc3011ee4e4461ca740c05d6d754dbaa157bc08f2c66b3b7b25e11ff0e876
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pinellolab/STREAM
  tools:
  - ElPiGraph
  - Python 3
  - STREAM
  - anndata
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# pseudotime-branch-visualization

## Summary

Visualize and quantify how single cells (e.g., genetically perturbed GMPs) distribute over the branching structure of a precomputed STREAM elastic-principal-graph trajectory, using STREAM's mapping feature to project new cells onto the fixed reference graph instead of re-fitting the trajectory.

## When to use

Use when scRNA-seq cells from a new condition (e.g., Gfi1-/-, Irf8-/-, double-knockout GMPs) must be projected onto an existing wild-type branching trajectory to read off branch-level fate allocation.

## When NOT to use

- The query cells have totally different fates from any present in the reference structure — the mapping procedure may not be optimal in that case.
- You need to learn a new trajectory from scratch rather than project cells onto an existing one — use STREAM's trajectory reconstruction workflow instead of the mapping feature.
- The data are cyclic or disconnected graph topologies — the paper notes STREAM requires further improvement to properly describe these more general structures.

## Inputs

- Wild-type reference scRNA-seq dataset with a precomputed STREAM elastic-principal-graph trajectory (ElPiGraph core)
- scRNA-seq count matrices for query/perturbed populations (e.g., Gfi1-/-, Irf8-/-, Gfi1-/-Irf8-/- GMP cells)
- anndata objects holding reference and query data

## Outputs

- Per-branch allocation table of mapped cells per genotype (fractions summing to 1 per genotype)
- Visualizations of mapped query cells overlaid on the reference branch structure
- Branch-fate assignment per cell (e.g., monocyte-committed vs. granulocyte-committed branch)

## How to apply

Load the wild-type reference scRNA-seq dataset together with its precomputed STREAM elastic-principal-graph (learned with the ElPiGraph core) into an anndata object, then load the count matrices for the query populations (e.g., Gfi1-/-, Irf8-/-, Gfi1-/-Irf8-/- GMPs). Run STREAM's mapping feature to project each query population onto the fixed reference graph using MLLE-based neighborhood weights, deliberately without pooling cells or re-fitting the trajectory — the paper emphasizes that no prior method supports this without recomputation. Tabulate the fraction of cells from each genotype assigned to each reference branch so per-branch fractions sum to 1, and visualize the mapped cells on the reference branch structure to reveal fate biases (e.g., Gfi1-/- diverted to the monocyte branch, Irf8-/- to the granulocyte branch, double KO balanced). Judge success by whether the allocation table exists, contains all genotypes with fractions summing to 1, and the visual pattern matches the expected branch bias.

## Related tools

- **STREAM** (Provides the mapping feature that projects query cells onto a precomputed elastic-principal-graph reference trajectory and the density-level branch visualization) — https://github.com/pinellolab/STREAM
- **ElPiGraph** (Core algorithm behind STREAM that learns the complex elastic principal graph on which cells are mapped and branches visualized) — sysbio-curie/ElPiGraph.R
- **anndata** (Data class in which STREAM stores reference and query scRNA-seq datasets for mapping)
- **Python 3** (Runtime environment for STREAM's mapping and visualization notebooks)

## Examples

```
conda create -n env_stream python=3.7 stream=1.0 jupyter
```

## Evaluation signals

- Allocation table exists and contains all expected genotypes (e.g., Gfi1-/-, Irf8-/-, Gfi1-/-Irf8-/-) with per-branch fractions summing to 1 per genotype.
- Reference trajectory is fixed during mapping — confirm cells were projected without pooling or re-fitting the graph.
- Mapped-cell visualization reproduces the expected branch pattern: Gfi1-/- biased to the monocyte branch, Irf8-/- to the granulocyte branch, double KO balanced between diverging branches.
- Mapping uses MLLE-based neighborhood weights and the same feature space as the reference graph (e.g., support for top_pcs added in v0.4.1).
- Expert or LLM review confirms the tabulated pattern matches the reported figure-level result.

## Limitations

- The mapping procedure may not be optimal if the new cells have totally different fates compared to those present in the reference structure.
- Further improvement is necessary to adapt STREAM to properly describe more general graph structures such as cyclic or disconnected topologies.
- ElPiGraph inherently supports learning complex graphs, but more work is necessary to provide intuitive visualizations for such topologies.
- The redesigned mapping procedure and removal of the STREAM command-line interface in v1.0 mean older v0.4.x workflow scripts may not transfer directly.

## Evidence

- [intro] no prior method maps new cells without refitting: "no trajectory inference method provides the possibility to map new cells to previously obtained reference trajectories without pooling cells and re-computing trajectories."
- [methods] anndata data class: "STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)]"
- [discussion] mapping caveat for novel fates: "the mapping procedure may not be optimal if the new cells have totally different fates compared to the ones present in the reference structure"
- [discussion] ElPiGraph core and visualization limits: "Although ElPiGraph, the core algorithm behind STREAM, inherently supports learning complex graphs, more work is necessary to provide intuitive visualizations"
- [readme] mapping tutorial notebook: "Example for *mapping* feature: [3-STREAM_mapping.ipynb]"
