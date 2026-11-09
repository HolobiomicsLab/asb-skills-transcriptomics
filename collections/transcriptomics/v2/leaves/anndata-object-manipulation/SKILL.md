---
name: anndata-object-manipulation
description: Use when building or running a STREAM workflow (trajectory reconstruction
  or mapping of new cells onto a fixed reference graph) whose inputs must be represented
  as anndata objects holding count matrices and per-cell metadata.
when_to_use_negative:
- When the task requires learning a new trajectory for the mutant cells rather than
  mapping onto a fixed reference graph — STREAM mapping is designed to avoid pooling
  cells and recomputing trajectories
- When the new cells have totally different fates compared to those present in the
  reference structure — the paper notes the mapping procedure may not be optimal in
  that case
- When data are not single-cell count matrices (e.g., bulk or already-aggregated feature
  tables) that need an anndata single-cell representation
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STREAM
  role: Trajectory pipeline that operates on anndata objects; its mapping feature
    projects new cells onto a precomputed reference elastic-principal-graph trajectory
  repo: https://github.com/pinellolab/STREAM
- name: anndata
  role: Data class in which STREAM is written; holds count matrices, metadata, and
    trajectory results
- name: ElPiGraph
  role: Core algorithm behind STREAM that learns the elastic principal graph stored
    alongside the reference anndata object
  repo: sysbio-curie/ElPiGraph.R
- name: conda
  role: Environment manager used to install stream (stream=1.0, python=3.7) for running
    anndata-based workflows
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:886cc3694462229bc605a6b129f980066a6ffed7e144b1e6cd2063eda6723356
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pinellolab/STREAM
  tools:
  - ElPiGraph
  - Python 3
  - STREAM
  - anndata
  - conda
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# anndata-object-manipulation

## Summary

Manipulate single-cell scRNA-seq data as anndata objects (the in-memory class used by STREAM) so that reference trajectories and mutant/perturbed cell populations can be loaded, kept distinct, and passed to STREAM's mapping and trajectory routines.

## When to use

Use when building or running a STREAM workflow (trajectory reconstruction or mapping of new cells onto a fixed reference graph) whose inputs must be represented as anndata objects holding count matrices and per-cell metadata.

## When NOT to use

- When the task requires learning a new trajectory for the mutant cells rather than mapping onto a fixed reference graph — STREAM mapping is designed to avoid pooling cells and recomputing trajectories
- When the new cells have totally different fates compared to those present in the reference structure — the paper notes the mapping procedure may not be optimal in that case
- When data are not single-cell count matrices (e.g., bulk or already-aggregated feature tables) that need an anndata single-cell representation

## Inputs

- Wild-type reference scRNA-seq count matrix stored as an anndata object with a precomputed STREAM elastic-principal-graph trajectory
- Mutant scRNA-seq count matrices (Gfi1-/-, Irf8-/-, Gfi1-/-Irf8-/- GMP cells), each as a separate anndata object
- Per-cell metadata (genotype labels, cell type annotations) in the anndata obs slots

## Outputs

- Annotated anndata objects with mapped cells assigned to reference branches (branch labels stored in per-cell metadata)
- Per-branch allocation table of cell fractions for each genotype (fractions summing to 1)
- Visualizations of mapped mutant cells on the reference branch structure

## How to apply

Represent each scRNA-seq count matrix — e.g., the wild-type hematopoiesis reference with its precomputed STREAM elastic-principal-graph, and the Gfi1-/-, Irf8-/-, and Gfi1-/-Irf8-/- GMP mutant matrices — as a separate anndata object rather than pooling cells. Keep the reference anndata object carrying the fixed ElPiGraph-derived trajectory so STREAM's mapping feature can project new cells onto it without recomputing the graph. Load the anndata class that STREAM is written with (anndata, Wolf et al. Genome Biology 2018) inside the stream conda environment (env_stream, python=3.7, stream=1.0) or the pinellolab/stream Docker container. Use the per-genotype anndata objects as inputs to the mapping step with MLLE-based neighborhood weights, then tabulate per-branch allocation fractions from the mapped cell annotations stored in the object.

## Related tools

- **STREAM** (Trajectory pipeline that operates on anndata objects; its mapping feature projects new cells onto a precomputed reference elastic-principal-graph trajectory) — https://github.com/pinellolab/STREAM
- **anndata** (Data class in which STREAM is written; holds count matrices, metadata, and trajectory results)
- **ElPiGraph** (Core algorithm behind STREAM that learns the elastic principal graph stored alongside the reference anndata object) — sysbio-curie/ElPiGraph.R
- **conda** (Environment manager used to install stream (stream=1.0, python=3.7) for running anndata-based workflows)

## Examples

```
conda create -n env_stream python=3.7 stream=1.0 jupyter
```

## Evaluation signals

- The per-branch allocation table exists and contains the three genotypes (Gfi1-/-, Irf8-/-, Gfi1-/-Irf8-/-) with per-branch fractions summing to 1
- Mapping was executed on the fixed reference graph without pooling cells or re-fitting the trajectory (the reference graph is unchanged after mapping)
- The mapped pattern reproduces the expected biology: Gfi1-/- biased to the monocyte branch, Irf8-/- to the granulocyte branch, double KO balanced between branches
- Each mutant population was mapped as a distinct anndata object with per-cell branch assignments stored in the object metadata

## Limitations

- The mapping procedure may not be optimal if the new cells have totally different fates compared to the ones present in the reference structure
- Scalability to millions of cells is not established for STREAM and many trajectory inference methods
- Further improvement is necessary to adapt STREAM to cyclic or disconnected graph topologies, limiting the trajectories an anndata object can carry

## Evidence

- [methods] STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)]: "STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)]"
- [intro] No mapping without recomputing: "no trajectory inference method provides the possibility to map new cells to previously obtained reference trajectories without pooling cells and re-computing trajectories."
- [discussion] Mapping caveat on different fates: "However, the mapping procedure may not be optimal if the new cells have totally different fates compared to the ones present in the reference structure"
- [readme] anndata reference in README: "STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)](http://anndata.rtfd.io) and available as user-friendly open source software"
