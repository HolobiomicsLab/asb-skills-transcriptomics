---
name: hematopoietic-lineage-annotation
description: Use when scRNA-seq data from hematopoietic subpopulations (HSCP/LSK,
  CMP, GMP, LKCD34+) needs an unbiased lineage hierarchy with per-branch gene expression,
  without prior time points, start cell, or branching count.
when_to_use_negative:
- Trajectories with cyclic or disconnected graph topologies — the article states further
  improvement is necessary to adapt STREAM to these more general structures
- Datasets with millions of cells — the discussion notes many current trajectory inference
  methods, including STREAM, don't scale to recent million-cell assays
- Mapping new cells whose fates are totally different from the reference structure
  — the mapping procedure may not be optimal in that case
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: STREAM
  role: Interactive pipeline that disentangles and visualizes branching trajectories;
    used to learn the hematopoietic principal graph and detect branch-dependent genes
  repo: https://github.com/pinellolab/STREAM
- name: ElPiGraph
  role: Core algorithm behind STREAM that learns the elastic branching principal graph
    over the hematopoietic cell space
  repo: sysbio-curie/ElPiGraph.R
- name: anndata
  role: Data container class in which the hematopoietic count matrix must be loaded
    before STREAM analysis
  repo: http://anndata.rtfd.io
- name: conda
  role: Environment manager used to install the pinned stream=1.0 Bioconda package
    with Python 3.7 and Jupyter
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:11a2eee0499ee183eae5a4e079c7f9089022dbc89b6b42f97dbb3c1cad59dd8d
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# hematopoietic-lineage-annotation

## Summary

Reconstruct and validate hematopoietic cell lineage hierarchies from scRNA-seq count data (e.g. LSK, CMP, GMP, LKCD34+ subpopulations) using STREAM's branching principal-graph framework, then annotate each inferred branch by branch-specific marker expression (Gfi1, Irf8, Gata1).

## When to use

Use when scRNA-seq data from hematopoietic subpopulations (HSCP/LSK, CMP, GMP, LKCD34+) needs an unbiased lineage hierarchy with per-branch gene expression, without prior time points, start cell, or branching count.

## When NOT to use

- Trajectories with cyclic or disconnected graph topologies — the article states further improvement is necessary to adapt STREAM to these more general structures
- Datasets with millions of cells — the discussion notes many current trajectory inference methods, including STREAM, don't scale to recent million-cell assays
- Mapping new cells whose fates are totally different from the reference structure — the mapping procedure may not be optimal in that case

## Inputs

- scRNA-seq count matrix for mouse hematopoietic subpopulations (LSK, CMP, GMP, LKCD34+), e.g. the wild-type Olsson et al. dataset
- anndata-formatted single-cell object (STREAM's native input class)
- List of candidate marker genes (e.g. Gfi1, Irf8, Gata1) for branch annotation

## Outputs

- Branching principal graph (trajectory hierarchy: HSCP → erythroid; Multi-Lin → Gran/Mono)
- Density-level trajectory visualization with cells positioned along inferred branches
- Branch-dependent (branch-specific) gene expression profiles, e.g. Gfi1/Irf8 on Gran/Mono branches, Gata1 on the erythroid branch
- Pseudotime ordering along reconstructed trajectories

## How to apply

Load the wild-type count matrix (LSK, CMP, GMP, LKCD34+ cells) into an anndata object, the class STREAM is written with. Preprocess with feature selection of informative genes following the STREAM scRNA-seq bifurcation tutorial (1.1-STREAM_scRNA-seq (Bifurcation).ipynb), then learn the branching principal graph — ElPiGraph is the core algorithm — supplying no prior knowledge such as time points, start cell, or number of branching events. Inspect the density-level trajectory visualization to read out the hierarchy, and detect branch-dependent genes: expect high Gfi1 and Irf8 on the Gran/Mono branches and high Gata1 on the erythroid branch. Validate by comparing the learned graph topology against the published hierarchy (HSCP → erythroid and Multi-Lin → Gran/Mono bifurcation); agreement of topology plus branch-specific marker expression confirms the annotation. Install via the Bioconda recipe in a dedicated environment (env_stream) with Python 3.7 to keep dependencies stable.

## Related tools

- **STREAM** (Interactive pipeline that disentangles and visualizes branching trajectories; used to learn the hematopoietic principal graph and detect branch-dependent genes) — https://github.com/pinellolab/STREAM
- **ElPiGraph** (Core algorithm behind STREAM that learns the elastic branching principal graph over the hematopoietic cell space) — sysbio-curie/ElPiGraph.R
- **anndata** (Data container class in which the hematopoietic count matrix must be loaded before STREAM analysis) — http://anndata.rtfd.io
- **conda** (Environment manager used to install the pinned stream=1.0 Bioconda package with Python 3.7 and Jupyter)

## Examples

```
conda create -n env_stream python=3.7 stream=1.0 jupyter
```

## Evaluation signals

- Learned graph topology must reproduce HSCP → erythroid and Multi-Lin → Gran/Mono bifurcation as in the published Olsson et al. hierarchy
- Gfi1 and Irf8 must be highly expressed specifically on the Gran/Mono branches
- Gata1 must be highly expressed specifically on the erythroid branch
- Density-level trajectory visualization must show continuous transitions between subpopulations (LSK/CMP/GMP/LKCD34+)
- Analysis runs without supplying time points, start cell, or the number of branching events — their absence is an invariant of the unbiased run

## Limitations

- ElPiGraph inherently supports learning complex graphs, but more work is necessary to provide intuitive visualizations for sophisticated topologies
- Further improvement is necessary to adapt STREAM to cyclic or disconnected structures
- Scalability: many current trajectory inference methods don't scale to assays profiling millions of cells
- The mapping procedure may not be optimal if new cells have totally different fates compared to the reference structure

## Evidence

- [abstract] STREAM unbiasedly reconstructs lineage hierarchy: "an interactive pipeline capable of disentangling and visualizing complex branching trajectories from both single-cell transcriptomic and epigenomic data."
- [discussion] No prior knowledge required: "Our method does not require prior knowledge such as time points, start cell, or the number of branching events to reconstruct trajectories"
- [methods] anndata container class: "STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)]"
- [discussion] ElPiGraph is the core algorithm: "Although ElPiGraph, the core algorithm behind STREAM, inherently supports learning complex graphs"
- [abstract] Hematopoiesis validation scope: "We further demonstrate its utility for understanding myoblast differentiation and disentangling known heterogeneity in hematopoiesis for different organisms."
- [readme] Conda environment setup from README: "Create an environment named `env_stream` , install **stream**, **jupyter**, and activate it with the following commands"
