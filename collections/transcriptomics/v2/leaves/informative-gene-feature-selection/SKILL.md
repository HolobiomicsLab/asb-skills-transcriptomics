---
name: informative-gene-feature-selection
description: Use when preparing a scRNA-seq expression matrix (loaded into an AnnData
  object) for STREAM trajectory inference, so that only informative genes drive the
  low-dimensional embedding and subsequent branching-tree reconstruction.
when_to_use_negative:
- the data contain cyclic or disconnected graph topologies — the paper states STREAM
  needs further improvement to properly describe these more general structures
- the goal is mapping new cells with totally different fates onto a previously obtained
  reference trajectory — the mapping procedure may not be optimal in that case
- the dataset scales to millions of cells — the paper notes many trajectory inference
  methods, including STREAM, do not scale to such datasets
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: STREAM
  role: pipeline whose feature-selection step computes informative genes and the low-dimensional
    embedding prior to tree learning
  repo: https://github.com/pinellolab/STREAM
- name: ElPiGraph
  role: core elastic principal graph algorithm that learns the branching trajectory
    on the selected informative features
  repo: https://github.com/pinellolab/STREAM
- name: anndata
  role: data class in which the expression matrix is held during feature selection
  repo: http://anndata.rtfd.io
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:6f252248549d8c94a3ad52c5086a0fa690987e4259b614fdd8febc64896dd8c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pinellolab/STREAM
  tools:
  - ElPiGraph
  - Python 3
  - STREAM
  - anndata
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# informative-gene-feature-selection

## Summary

Select informative genes (informative features) from a single-cell expression matrix as the pre-processing step that STREAM uses before fitting its low-dimensional embedding and learning the branching trajectory with the ElPiGraph elastic principal graph algorithm.

## When to use

Use when preparing a scRNA-seq expression matrix (loaded into an AnnData object) for STREAM trajectory inference, so that only informative genes drive the low-dimensional embedding and subsequent branching-tree reconstruction.

## When NOT to use

- the data contain cyclic or disconnected graph topologies — the paper states STREAM needs further improvement to properly describe these more general structures
- the goal is mapping new cells with totally different fates onto a previously obtained reference trajectory — the mapping procedure may not be optimal in that case
- the dataset scales to millions of cells — the paper notes many trajectory inference methods, including STREAM, do not scale to such datasets

## Inputs

- single-cell RNA-seq expression matrix (gene x cell counts, e.g., 1656 Nestorowa mouse hematopoietic cells)
- AnnData object as used by STREAM
- optional list of expected lineage marker genes (e.g., Car2, Epx, Coro1a, Blvrb) for validation

## Outputs

- subset of informative genes/features used for trajectory learning
- low-dimensional embedding fitted on the informative genes
- branching trajectory learned by ElPiGraph with diverging and transition gene assignments

## How to apply

Load the expression matrix (e.g., the Nestorowa et al. mouse hematopoiesis matrix of 1656 sorted cells) into an AnnData object as used by STREAM, then run STREAM's informative-gene selection over the gene-by-cell matrix to obtain a reduced feature space. Fit the low-dimensional embedding that STREAM computes on these informative genes before tree learning, and feed the embedding to the ElPiGraph elastic principal graph algorithm. Do not supply prior knowledge such as time points, a start cell, or a preset number of branching events — the STREAM paper stresses the method does not require these. Validate the chosen features downstream: diverging genes (e.g., expected erythroid markers Car2 and Epx) and transition genes (e.g., Coro1a, Blvrb) should be recovered along the reconstructed branches, and the tree should recapitulate the known lymphoid/myeloid/erythroid bifurcations with multipotent progenitors placed before the first bifurcation (Fig. 1b-d).

## Related tools

- **STREAM** (pipeline whose feature-selection step computes informative genes and the low-dimensional embedding prior to tree learning) — https://github.com/pinellolab/STREAM
- **ElPiGraph** (core elastic principal graph algorithm that learns the branching trajectory on the selected informative features) — https://github.com/pinellolab/STREAM
- **anndata** (data class in which the expression matrix is held during feature selection) — http://anndata.rtfd.io

## Examples

```
conda create -n env_stream python=3.7 stream=1.0 jupyter && conda activate env_stream && jupyter notebook
```

## Evaluation signals

- the reconstructed tree exhibits the reported lymphoid/myeloid/erythroid bifurcations, consistent with Figure 1 of the STREAM paper
- multipotent progenitors are placed before the first bifurcation in the learned tree
- expected erythroid-diverging markers (Car2, Epx) appear in the detected diverging-gene output
- transition genes (Coro1a, Blvrb) are recovered in the marker detection along branches
- no prior knowledge (time points, start cell, number of branching events) was supplied during reconstruction

## Limitations

- further improvement is necessary to adapt STREAM to cyclic or disconnected graph topologies
- scalability to millions of cells is not established for STREAM and many other trajectory inference methods
- although ElPiGraph inherently supports learning complex graphs, more work is necessary to provide intuitive visualizations
- the mapping procedure may not be optimal if new cells have totally different fates compared to the reference structure

## Evidence

- [discussion] no prior knowledge claim: "Our method does not require prior knowledge such as time points, start cell, or the number of branching events to reconstruct trajectories"
- [abstract] scope of pipeline: "an interactive pipeline capable of disentangling and visualizing complex branching trajectories from both single-cell transcriptomic and epigenomic data"
- [methods] data container: "STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)]"
- [readme] installation env: "$ conda create -n env_stream python=3.7 stream=1.0 jupyter"
- [discussion] core algorithm: "Although ElPiGraph, the core algorithm behind STREAM, inherently supports learning complex graphs, more work is necessary to provide intuitive visualizations"
