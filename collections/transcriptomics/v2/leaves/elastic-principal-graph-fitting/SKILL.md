---
name: elastic-principal-graph-fitting
description: Use when scRNA-seq (or scATAC-seq) expression data in an AnnData object
  must be ordered into a branching trajectory/pseudotime and the number of bifurcations
  is unknown a priori.
when_to_use_negative:
- The underlying structure is cyclic or disconnected — the article states further
  improvement is necessary to adapt STREAM to these more general graph structures
- The dataset profiles millions of cells — scalability of many trajectory inference
  methods including STREAM is not established
- New cells have totally different fates than the reference structure — the mapping
  procedure may not be optimal
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: STREAM
  role: Interactive pipeline that invokes ElPiGraph to fit branching principal graphs
    and provides density-level visualization and marker detection on anndata objects
  repo: https://github.com/pinellolab/STREAM
- name: ElPiGraph
  role: Core elastic principal graph algorithm that learns the complex branching graph
    underlying the fitted trajectory
  repo: https://github.com/sysbio-curie/ElPiGraph.R
- name: anndata
  role: Class used by STREAM to hold the expression matrix and cell metadata during
    graph fitting
- name: conda
  role: Installs stream 1.0 with Python 3.7 and Jupyter in the env_stream environment
    via Bioconda
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:d7fcc8426332ccb14db3a2ee2c8b02379c88b0f16ac70917806c702f765925de
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# elastic-principal-graph-fitting

## Summary

Fits an elastic principal graph (ElPiGraph) to a low-dimensional embedding of single-cell data to reconstruct branching developmental trajectories, without requiring time points, a start cell, or the number of branching events.

## When to use

Use when scRNA-seq (or scATAC-seq) expression data in an AnnData object must be ordered into a branching trajectory/pseudotime and the number of bifurcations is unknown a priori.

## When NOT to use

- The underlying structure is cyclic or disconnected — the article states further improvement is necessary to adapt STREAM to these more general graph structures
- The dataset profiles millions of cells — scalability of many trajectory inference methods including STREAM is not established
- New cells have totally different fates than the reference structure — the mapping procedure may not be optimal

## Inputs

- Single-cell expression count matrix (scRNA-seq qPCR or RNA-seq; also scATAC-seq peaks/k-mers/motifs) loaded into an anndata object
- Cell type or subpopulation labels (optional, e.g., LSK, CMP, GMP, LKCD34+)

## Outputs

- Learned branching principal graph (trajectory tree) with inferred pseudotime
- Density-level trajectory visualization with cell type labels (e.g., stream_nestorowa_trajectory.png)
- Detected diverging genes and transition genes along branches
- Mapped cells on previously obtained reference trajectories (via the mapping feature)

## How to apply

Load the count matrix (e.g., the Nestorowa 1656-cell mouse HSC scRNA-seq dataset or wild-type Olsson LSK/CMP/GMP/LKCD34+ data) into an AnnData object, the class STREAM is written with. Select informative genes and fit the low-dimensional embedding STREAM computes before tree learning, then fit the branching principal graph with ElPiGraph — the core algorithm underlying STREAM — supplying no prior knowledge such as time points, start cell, or number of branching events. Render the learned graph as a density-level trajectory and detect diverging and transition genes along branches (e.g., erythroid markers Car2/Epx or Gata1; transition genes Coro1a/Blvrb; Gfi1/Irf8 on Gran/Mono branches). Validate by comparing the learned topology to expected lineage hierarchies such as HSCP → erythroid / Multi-Lin → Gran/Mono.

## Related tools

- **STREAM** (Interactive pipeline that invokes ElPiGraph to fit branching principal graphs and provides density-level visualization and marker detection on anndata objects) — https://github.com/pinellolab/STREAM
- **ElPiGraph** (Core elastic principal graph algorithm that learns the complex branching graph underlying the fitted trajectory) — https://github.com/sysbio-curie/ElPiGraph.R
- **anndata** (Class used by STREAM to hold the expression matrix and cell metadata during graph fitting)
- **conda** (Installs stream 1.0 with Python 3.7 and Jupyter in the env_stream environment via Bioconda)

## Examples

```
conda create -n env_stream python=3.7 stream=1.0 jupyter && conda activate env_stream && jupyter notebook
```

## Evaluation signals

- The reconstructed tree exhibits the reported lymphoid/myeloid/erythroid bifurcations consistent with Figure 1 of the STREAM paper on Nestorowa data
- Expected erythroid-diverging markers (Car2, Epx, Gata1) and transition genes (Coro1a, Blvrb) appear in the detected marker output
- On Olsson wild-type data, the topology shows HSCP → erythroid / Multi-Lin → Gran/Mono with Gfi1 and Irf8 high on Gran/Mono branches and Gata1 high on the erythroid branch, matching Fig. 2a-c
- The trajectory was fit without supplying time points, a start cell, or a preset number of branching events (no prior-knowledge parameters were used)

## Limitations

- Further improvement is necessary to adapt STREAM/ElPiGraph to cyclic or disconnected graph topologies
- More work is necessary to provide intuitive visualizations for sophisticated topologies that ElPiGraph inherently supports
- Scalability to millions of cells is not established for many trajectory inference methods including STREAM

## Evidence

- [discussion] ElPiGraph is the core graph-learning algorithm: "Although ElPiGraph, the core algorithm behind STREAM, inherently supports learning complex graphs, more work is necessary to provide intuitive visualizations"
- [discussion] No prior knowledge required for trajectory reconstruction: "Our method does not require prior knowledge such as time points, start cell, or the number of branching events to reconstruct trajectories"
- [methods] STREAM is written with anndata and installable via conda: "STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)]"
- [readme] Conda environment setup for STREAM: "conda create -n env_stream python=3.7 stream=1.0 jupyter"
- [abstract] STREAM tested across organisms and technologies: "We have tested STREAM on several synthetic and real datasets generated with different single-cell technologies."
