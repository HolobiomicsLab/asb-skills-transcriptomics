---
name: sliding-window-cell-count-estimation
description: Use when a single-cell dataset (cells x features in an anndata object)
  already has an inferred branching graph and pseudotime, and you need to render a
  stream plot whose branch width is proportional to the total number of cells at each
  pseudotime position.
when_to_use_negative:
- No branching trajectory graph or pseudotime has yet been inferred — STREAM's trajectory
  reconstruction (ElPiGraph) must be run first
- The goal is to display single cells or clusters along pseudotime rather than subpopulation
  composition and its continuous transition — STREAM's density-level visualization
  was developed precisely because current methods mainly focus on displaying single
  cells or clusters along pseudotime
- The trajectory topology is cyclic or disconnected — the article notes further improvement
  is necessary to adapt STREAM to properly describing these more general structures
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0092
tools:
- name: STREAM
  role: Provides the density-level trajectory visualization (stream plot) that this
    skill reimplements; branch width is proportional to the total number of cells
    at a given pseudotime
  repo: https://github.com/pinellolab/STREAM
- name: anndata
  role: Data container class STREAM is written with; used to load the cells x features
    matrix with branching graph and pseudotime
- name: Python 3
  role: Runtime environment for the reimplementation (installed via Python3 Anaconda
    / conda env env_stream with stream=1.0)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:5e78c8e9a22cb35c88e2c7edea353e2b5a75bb3d39108601e566ccf6803e96cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pinellolab/STREAM
  tools:
  - Python 3
  - STREAM
  - anndata
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# sliding-window-cell-count-estimation

## Summary

Reconstructs STREAM's density-level trajectory visualization by computing sliding-window cell counts along each branch of an inferred branching graph, normalizing widths by the longest path, and smoothing them with a Savitzky-Golay filter so that branch width encodes local cell density.

## When to use

Use when a single-cell dataset (cells x features in an anndata object) already has an inferred branching graph and pseudotime, and you need to render a stream plot whose branch width is proportional to the total number of cells at each pseudotime position.

## When NOT to use

- No branching trajectory graph or pseudotime has yet been inferred — STREAM's trajectory reconstruction (ElPiGraph) must be run first
- The goal is to display single cells or clusters along pseudotime rather than subpopulation composition and its continuous transition — STREAM's density-level visualization was developed precisely because current methods mainly focus on displaying single cells or clusters along pseudotime
- The trajectory topology is cyclic or disconnected — the article notes further improvement is necessary to adapt STREAM to properly describing these more general structures

## Inputs

- Single-cell dataset as an anndata object (cells x features, e.g. scRNA-seq or scATAC-seq counts)
- Inferred branching trajectory graph with branch structure (from ElPiGraph, the core algorithm behind STREAM)
- Per-cell pseudotime coordinates
- Optional: gene expression values for a gene of interest to overlay on the stream plot

## Outputs

- Per-branch smoothed, normalized cell-density (width) curves on a dense interpolated grid
- Polygon geometry for each branch whose half-width at each position equals the smoothed, normalized cell density
- stream_plot.png density-level trajectory visualization, optionally with a bicubically smoothed gene-expression overlay

## How to apply

Load the cells x features matrix, inferred branching graph, and pseudotime using the anndata class that STREAM is written with. For each branch, bin cells into sliding windows along the branch coordinate, including windows that span parent and child branches near each branching node so that density is continuous across bifurcations. Normalize the per-branch width curves by the longest path in the trajectory graph so widths are comparable across the plot. Linearly interpolate the width curves onto a dense grid and smooth them with a Savitzky-Golay filter to remove sampling noise while preserving branch-shape trends. Finally, render each branch as a polygon whose half-width at each position equals the smoothed, normalized cell density; the same scaffolding supports overlaying a gene-expression signal smoothed bicubically along each branch.

## Related tools

- **STREAM** (Provides the density-level trajectory visualization (stream plot) that this skill reimplements; branch width is proportional to the total number of cells at a given pseudotime) — https://github.com/pinellolab/STREAM
- **anndata** (Data container class STREAM is written with; used to load the cells x features matrix with branching graph and pseudotime)
- **Python 3** (Runtime environment for the reimplementation (installed via Python3 Anaconda / conda env env_stream with stream=1.0))

## Examples

```
conda create -n env_stream python=3.7 stream=1.0 jupyter; conda activate env_stream; jupyter notebook
```

## Evaluation signals

- stream_plot.png exists and is a valid PNG figure
- Polygon half-widths scale with local cell counts: branches (or branch segments) traversed by more cells are visibly wider (verifiable via llm_judge review)
- Width curves are smooth and free of window-sampling noise, indicating the Savitzky-Golay smoothing step ran with branch-shape trends preserved
- Relative branch widths agree with per-branch cell totals after normalization by the longest path in the trajectory graph
- Sliding windows spanning branching nodes produce continuous density across parent and child branches (no discontinuities at bifurcations)

## Limitations

- STREAM does not require prior knowledge such as time points, start cell, or number of branching events, but further improvement is necessary to adapt it to cyclic or disconnected graph structures
- Although ElPiGraph, the core algorithm behind STREAM, inherently supports learning complex graphs, more work is necessary to provide intuitive visualizations for such topologies
- Scalability to very large datasets is a concern: recent single-cell assays can profile millions of cells, but many current trajectory inference methods, including those in this space, don't scale
- Efforts on trajectory visualization remain limited; the width encoding shows subpopulation composition and its continuous transition but is one projection of a more complex graph

## Evidence

- [discussion] STREAM introduces four innovations compared to existing methods: (1) a density-level trajectory visualization: "Additionally STREAM introduces four innovations compared to other existing methods: (1) a density-level trajectory visualization"
- [intro] Current visualization limits: "Current methods mainly focus on displaying single cells or clusters (or stable states) along the pseudotime, which makes it difficult to study"
- [methods] STREAM is written with anndata: "STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)]"
- [readme] Stream plot feature is accessible through the interactive web panels: [evidence span withheld — anchor status: external_doc]
- [readme] Install environment for reproducing the visualization: "$ conda create -n env_stream python=3.7 stream=1.0 jupyter"
