---
name: branch-dependent-gene-detection
description: Use when a branching single-cell trajectory has been inferred (e.g.,
  STREAM on scRNA-seq) and you need to identify marker genes enriched on each branch
  to validate the topology or annotate lineage identities.
when_to_use_negative:
- The trajectory is cyclic or disconnected — the article notes further improvement
  is necessary to adapt STREAM to these more general structures before branch testing
  is meaningful
- Data are single-cell chromatin accessibility and the goal is TF binding dynamics
  rather than gene-expression branch markers — use the scATAC-seq mode/features instead
- You need to map new cells with potentially different fates onto a precomputed trajectory
  — the mapping procedure may not be optimal for cells with totally different fates
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: STREAM
  role: Interactive pipeline used to learn the branching graph and detect/visualize
    branch-dependent genes; written using the anndata class
  repo: https://github.com/pinellolab/STREAM
- name: ElPiGraph
  role: Core algorithm inside STREAM that learns the elastic principal graph whose
    branches define the gene-testing units
  repo: https://github.com/sysbio-curie/ElPiGraph.R
- name: anndata
  role: Data class in which the scRNA-seq count matrix must be loaded before STREAM
    branch detection
  repo: http://anndata.rtfd.io
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:7a25a0f54388e0707870dd1446342f6c2b2d680cf27eae601fee8cc256a56d0a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pinellolab/STREAM
  tools:
  - ElPiGraph
  - Python 3
  - STREAM
  - anndata
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Branch-dependent gene detection

## Summary

Detect and visualize genes whose expression is significantly enriched on specific branches of a trajectory reconstructed with STREAM, using the branch-specific differential testing built into STREAM (which runs ElPiGraph as its core principal-graph algorithm). This validates lineage branching by confirming that known fate markers (e.g., Gfi1, Irf8, Gata1 in mouse hematopoiesis) localize to the correct inferred branches.

## When to use

Use when a branching single-cell trajectory has been inferred (e.g., STREAM on scRNA-seq) and you need to identify marker genes enriched on each branch to validate the topology or annotate lineage identities.

## When NOT to use

- The trajectory is cyclic or disconnected — the article notes further improvement is necessary to adapt STREAM to these more general structures before branch testing is meaningful
- Data are single-cell chromatin accessibility and the goal is TF binding dynamics rather than gene-expression branch markers — use the scATAC-seq mode/features instead
- You need to map new cells with potentially different fates onto a precomputed trajectory — the mapping procedure may not be optimal for cells with totally different fates

## Inputs

- STREAM-fitted anndata object with an inferred branching principal graph (e.g., wild-type Olsson et al. mouse hematopoietic scRNA-seq count matrix of LSK, CMP, GMP, LKCD34+ cells)
- Preprocessed expression matrix with informative genes selected per the STREAM scRNA-seq bifurcation tutorial

## Outputs

- Branch-dependent gene lists scored per branch/edge of the learned principal graph
- Density-level trajectory plots with branch-specific gene expression overlays (e.g., Gfi1, Irf8 on Gran/Mono branches; Gata1 on the erythroid branch)

## How to apply

After learning the branching principal graph with STREAM (built on ElPiGraph, no prior knowledge of time points, start cell, or number of branching events required), run STREAM's branch detection on the fitted object to score genes for branch dependence. In the Olsson et al. mouse hematopoiesis case (LSK, CMP, GMP, LKCD34+ subpopulations), the tutorial workflow 1.1-STREAM_scRNA-seq (Bifurcation).ipynb is followed: preprocess with STREAM's informative-gene feature selection, fit the graph, then detect branch-dependent genes and visualize them along the density-level trajectory. Judge correctness by checking that branch-specific markers recover expected biology: Gfi1 and Irf8 high on the Gran/Mono branches and Gata1 high on the erythroid branch, matching the HSCP -> erythroid / Multi-Lin -> Gran/Mono hierarchy of Fig. 2a-c.

## Related tools

- **STREAM** (Interactive pipeline used to learn the branching graph and detect/visualize branch-dependent genes; written using the anndata class) — https://github.com/pinellolab/STREAM
- **ElPiGraph** (Core algorithm inside STREAM that learns the elastic principal graph whose branches define the gene-testing units) — https://github.com/sysbio-curie/ElPiGraph.R
- **anndata** (Data class in which the scRNA-seq count matrix must be loaded before STREAM branch detection) — http://anndata.rtfd.io

## Examples

```
conda create -n env_stream python=3.7 stream=1.0 jupyter; conda activate env_stream; jupyter notebook  # then run 1.1-STREAM_scRNA-seq (Bifurcation).ipynb on the Olsson et al. count matrix
```

## Evaluation signals

- Learned graph topology reproduces the Olsson et al. Fig. 2a-c hierarchy: HSCP -> erythroid and Multi-Lin -> Gran/Mono bifurcation
- Gfi1 and Irf8 expression is high on the Gran/Mono branches of the reconstructed trajectory
- Gata1 expression is high on the erythroid branch
- Trajectory was inferred without supplying prior knowledge (time points, start cell, or number of branching events)
- Branch-specific plots are produced with STREAM's density-level trajectory visualization rather than single-cell or cluster-only pseudotime displays

## Limitations

- ElPiGraph inherently supports learning complex graphs, but more work is necessary to provide intuitive visualizations for sophisticated topologies
- STREAM is not yet adapted to cyclic or disconnected graph topologies
- Scalability to millions of cells is not established for STREAM
- The mapping procedure may not be optimal if new cells have totally different fates compared to the reference structure

## Evidence

- [discussion] STREAM does not require prior knowledge of branches: "Our method does not require prior knowledge such as time points, start cell, or the number of branching events to reconstruct trajectories"
- [methods] anndata class requirement: "STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)]"
- [discussion] ElPiGraph as core algorithm: "Although ElPiGraph, the core algorithm behind STREAM, inherently supports learning complex graphs, more work is necessary to provide intuitive visualizations"
- [discussion] Density-level visualization innovation: "Additionally STREAM introduces four innovations compared to other existing methods: (1) a density-level trajectory visualization"
- [readme] Tutorial notebook for scRNA-seq bifurcation workflow: "Example for scRNA-seq: [1.1-STREAM_scRNA-seq (Bifurcation).ipynb]"
- [methods] Conda environment creation command: "$ conda create -n env_stream python=3.7 stream=1.0 jupyter"
