---
name: hematopoietic-lineage-marker-interpretation
description: Use when a fitted branching trajectory on sorted hematopoietic scRNA-seq
  data (e.g., Nestorowa HSC dataset) needs biological validation via marker detection
  at bifurcations into lymphoid, myeloid, and erythroid fates.
when_to_use_negative:
- The data are single-cell chromatin accessibility (scATAC-seq) with motifs/peaks
  rather than transcriptomic markers — gene-level marker interpretation would need
  TF binding dynamics instead
- The reconstructed structure is cyclic or disconnected rather than a branching tree;
  the paper notes further work is needed to adapt STREAM to such topologies
- The goal is mapping new cells onto a reference trajectory, not interpreting lineage
  markers on the fitted tree
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: STREAM
  role: Interactive pipeline that fits the branching trajectory, detects diverging
    and transition genes, and renders the labeled trajectory figure
  repo: https://github.com/pinellolab/STREAM
- name: ElPiGraph
  role: Core elastic principal graph algorithm underlying STREAM used to learn the
    branching structure whose branches are annotated with markers
  repo: https://github.com/sysbio-curie/ElPiGraph.R
- name: anndata
  role: Class in which the hematopoietic scRNA-seq expression matrix is loaded before
    marker detection in STREAM
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:3734f534c740d0e818ba2eded14472a530a4ea7425cb6eb5a7541e1fec994d02
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pinellolab/STREAM
  tools:
  - ElPiGraph
  - Python 3
  - STREAM
  - anndata
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# hematopoietic-lineage-marker-interpretation

## Summary

Interprets a reconstructed hematopoietic branching trajectory by detecting diverging and transition genes along branches and validating lineage assignments against known mouse hematopoiesis markers (e.g., erythroid Car2/Epx, transition Coro1a/Blvrb), as demonstrated on the Nestorowa 1656-cell scRNA-seq dataset with STREAM.

## When to use

Use when a fitted branching trajectory on sorted hematopoietic scRNA-seq data (e.g., Nestorowa HSC dataset) needs biological validation via marker detection at bifurcations into lymphoid, myeloid, and erythroid fates.

## When NOT to use

- The data are single-cell chromatin accessibility (scATAC-seq) with motifs/peaks rather than transcriptomic markers — gene-level marker interpretation would need TF binding dynamics instead
- The reconstructed structure is cyclic or disconnected rather than a branching tree; the paper notes further work is needed to adapt STREAM to such topologies
- The goal is mapping new cells onto a reference trajectory, not interpreting lineage markers on the fitted tree

## Inputs

- STREAM-fitted branching trajectory (elastic principal graph) computed from a hematopoietic scRNA-seq expression matrix loaded in an AnnData object (e.g., Nestorowa et al. 1656 sorted cells)
- Gene expression matrix with informative genes selected for trajectory learning
- Prior list of canonical lineage markers (e.g., Car2, Epx for erythroid; Coro1a, Blvrb as transition genes)

## Outputs

- Detected diverging genes per branch (e.g., Car2, Epx on the erythroid branch)
- Detected transition genes along the trajectory (e.g., Coro1a, Blvrb)
- Labeled branching trajectory figure (e.g., stream_nestorowa_trajectory.png) with lymphoid, myeloid, and erythroid fates
- Pseudotime ordering placing multipotent progenitors before the first bifurcation

## How to apply

After fitting the branching trajectory with STREAM's ElPiGraph elastic principal graph (without supplying time points, a start cell, or a preset number of branching events), run STREAM's marker detection to identify diverging genes along each branch and transition genes near bifurcation points. Compare detected diverging markers against expected fate-specific markers — e.g., Car2 and Epx for the erythroid branch — and check transition genes such as Coro1a and Blvrb that mark branch points. Annotate branches with cell type labels corresponding to lymphoid, myeloid, and erythroid fates following the STREAM scRNA-seq bifurcation tutorial, and render the trajectory with these labels. Validate that multipotent progenitors are placed before the first bifurcation and that the marker output contains the expected erythroid-diverging and transition markers, consistent with Figure 1 of the STREAM paper.

## Related tools

- **STREAM** (Interactive pipeline that fits the branching trajectory, detects diverging and transition genes, and renders the labeled trajectory figure) — https://github.com/pinellolab/STREAM
- **ElPiGraph** (Core elastic principal graph algorithm underlying STREAM used to learn the branching structure whose branches are annotated with markers) — https://github.com/sysbio-curie/ElPiGraph.R
- **anndata** (Class in which the hematopoietic scRNA-seq expression matrix is loaded before marker detection in STREAM)

## Examples

```
conda create -n env_stream python=3.7 stream=1.0 jupyter && conda activate env_stream && jupyter notebook
```

## Evaluation signals

- The reconstructed tree exhibits the reported lymphoid/myeloid/erythroid bifurcations
- Expected erythroid markers Car2 and Epx appear in the detected diverging-gene output for the erythroid branch
- Transition genes Coro1a and Blvrb appear in the detected marker output near branch points
- Multipotent progenitors are placed before the first bifurcation, consistent with Figure 1 of the STREAM paper
- No prior knowledge (time points, start cell, number of branching events) was supplied to obtain the structure

## Limitations

- ElPiGraph inherently supports learning complex graphs, but more work is necessary to provide intuitive visualizations for sophisticated topologies
- Trajectory inference methods including STREAM may not scale to datasets with millions of cells
- Validation depends on prior biological knowledge of expected markers; failure to detect Car2/Epx or Coro1a/Blvrb may indicate misfit rather than wrong biology

## Evidence

- [discussion] No prior knowledge required for trajectory fitting: "Our method does not require prior knowledge such as time points, start cell, or the number of branching events to reconstruct trajectories"
- [discussion] Core algorithm behind STREAM: "Although ElPiGraph, the core algorithm behind STREAM, inherently supports learning complex graphs, more work is necessary to provide intuitive visualizations"
- [methods] STREAM is written using anndata: "STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)]"
- [abstract] Hematopoietic heterogeneity application: "We further demonstrate its utility for understanding myoblast differentiation and disentangling known heterogeneity in hematopoiesis for different organisms."
- [readme] Conda install command: "$ conda create -n env_stream python=3.7 stream=1.0 jupyter"
