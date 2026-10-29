---
name: cell-fate-branch-allocation-quantification
description: Use when scRNA-seq cells from perturbed (e.g. knockout GMP) populations
  must be allocated to branches of an already-inferred wild-type branching trajectory
  to measure fate bias between, e.g., monocyte and granulocyte branches.
when_to_use_negative:
- When the new cells have totally different fates compared to the ones present in
  the reference structure — the mapping procedure may not be optimal in this case
- When the reference trajectory itself must be inferred or updated (this skill requires
  a fixed, precomputed STREAM graph); use the full STREAM trajectory inference workflow
  instead
- When the underlying topology is cyclic or disconnected — STREAM does not yet properly
  describe these more general structures
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STREAM
  role: Provides the mapping feature that projects mutant cells onto the fixed reference
    elastic principal graph using MLLE-based neighborhood weights without re-fitting
  repo: https://github.com/pinellolab/STREAM
- name: ElPiGraph
  role: Core algorithm that learned the reference elastic principal graph onto which
    mutant cells are mapped
  repo: https://github.com/Albluca/ElPiGraph.R
- name: anndata
  role: Data container class used to hold the reference and mutant scRNA-seq count
    matrices during mapping
  repo: http://anndata.rtfd.io
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:8210603ab87bbacf8699f92a13ff6db7fa753e2e97ce53a03fd6f43d06fc87c3
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
yes_rate: 0.9
n_claims: 10
---

# cell-fate-branch-allocation-quantification

## Summary

Quantifies how genetically perturbed cells are diverted among the branches of a fixed reference trajectory by projecting them, without re-fitting, onto a precomputed STREAM elastic principal graph and tabulating per-genotype branch fractions.

## When to use

Use when scRNA-seq cells from perturbed (e.g. knockout GMP) populations must be allocated to branches of an already-inferred wild-type branching trajectory to measure fate bias between, e.g., monocyte and granulocyte branches.

## When NOT to use

- When the new cells have totally different fates compared to the ones present in the reference structure — the mapping procedure may not be optimal in this case
- When the reference trajectory itself must be inferred or updated (this skill requires a fixed, precomputed STREAM graph); use the full STREAM trajectory inference workflow instead
- When the underlying topology is cyclic or disconnected — STREAM does not yet properly describe these more general structures

## Inputs

- Wild-type reference scRNA-seq dataset with a precomputed STREAM elastic principal graph (ElPiGraph core)
- scRNA-seq count matrices for perturbed GMP populations (e.g. Gfi1-/-, Irf8-/-, Gfi1-/-Irf8-/-) in anndata format

## Outputs

- Per-branch allocation table: fraction of cells from each genotype assigned to each reference branch, per-branch fractions summing to 1
- Visualization of mapped mutant cells on the reference branch structure (Fig. 2f-g style plot)

## How to apply

Load the wild-type reference scRNA-seq dataset together with its precomputed STREAM elastic principal graph (learned with the ElPiGraph core) into anndata objects, then load the count matrices for each mutant population (e.g. Gfi1-/-, Irf8-/-, Gfi1-/-Irf8-/- GMPs). Execute the STREAM mapping feature to project each mutant population onto the fixed reference graph using MLLE-based neighborhood weights, explicitly avoiding pooling cells with the reference or re-fitting the trajectory. Tabulate the fraction of cells from each genotype assigned to each reference branch, verifying the per-branch fractions sum to 1 for each genotype. Visualize the mapped mutant cells on the reference branch structure to confirm the allocation pattern; the reference graph is held fixed because STREAM uniquely offers deterministic mapping of new cells onto an existing trajectory.

## Related tools

- **STREAM** (Provides the mapping feature that projects mutant cells onto the fixed reference elastic principal graph using MLLE-based neighborhood weights without re-fitting) — https://github.com/pinellolab/STREAM
- **ElPiGraph** (Core algorithm that learned the reference elastic principal graph onto which mutant cells are mapped) — https://github.com/Albluca/ElPiGraph.R
- **anndata** (Data container class used to hold the reference and mutant scRNA-seq count matrices during mapping) — http://anndata.rtfd.io

## Examples

```
conda create -n env_stream python=3.7 stream=1.0 jupyter && conda activate env_stream && jupyter notebook 3-STREAM_mapping.ipynb
```

## Evaluation signals

- Allocation table exists and contains all three genotypes with per-branch fractions summing exactly to 1
- Pattern check: Gfi1-/- cells are biased to the monocyte-committed branch, Irf8-/- cells to the granulocyte-committed branch, and Gfi1-/-Irf8-/- cells show no imbalance between the diverging branches
- Mapping was performed without pooling mutant cells with the reference or re-fitting the trajectory (fixed reference graph)
- Mapped-cell overlay on the reference branch structure visually reproduces the reported branch-assignment pattern (Fig. 2f-g)

## Limitations

- The mapping procedure may not be optimal if the new cells have totally different fates compared to the ones present in the reference structure
- Mapping quality depends on the neighborhood weighting; no re-fitting of the reference trajectory is performed, so branch assignments reflect the fixed wild-type structure
- No support for cyclic or disconnected graph topologies; further improvement is necessary to adapt STREAM to these more general structures

## Evidence

- [intro] Unique mapping capability without re-fitting: "no trajectory inference method provides the possibility to map new cells to previously obtained reference trajectories without pooling cells and re-computing trajectories."
- [discussion] Mapping caveat for divergent fates: "However, the mapping procedure may not be optimal if the new cells have totally different fates compared to the ones present in the reference structure"
- [readme] STREAM mapping tutorial notebook: "STREAM is an interactive pipeline capable of disentangling and visualizing complex branching trajectories from both single-cell transcriptomic and epigenomic data."
