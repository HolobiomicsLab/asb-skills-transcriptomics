---
name: savitzky-golay-width-curve-smoothing
description: Use when rendering branching single-cell trajectories as stream plots
  whose branch width must be proportional to the total number of cells at a given
  pseudotime, and when gene-expression overlays along branches are needed.
when_to_use_negative:
- The input is raw single-cell data without an already-inferred branching graph and
  pseudotime — first run trajectory reconstruction with STREAM/ElPiGraph before visualizing.
- The goal is a trajectory for cyclic or disconnected graph topologies, which STREAM
  does not properly describe.
- The analysis target is motif/accessibility feature selection for scATAC-seq rather
  than density-level trajectory visualization.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3318
tools:
- name: STREAM
  role: Source pipeline whose density-level stream plot encodes branch width proportional
    to total cell count at a given pseudotime; this skill reimplements its plotting
    module
  repo: https://github.com/pinellolab/STREAM
- name: anndata
  role: Data container class used to load the single-cell dataset with branching graph
    and pseudotime prior to width-curve computation
  repo: http://anndata.rtfd.io
- name: Python 3
  role: Implementation language for the sliding-window counting, interpolation, Savitzky-Golay
    smoothing, and polygon rendering
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:26317a31e6a3ec21c6c709865a7f2a9c33e5229cd1793e1d5175b8132d3328e8
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# savitzky-golay-width-curve-smoothing

## Summary

Reconstructs STREAM's density-level stream plot by converting per-branch sliding-window cell counts into smoothed, normalized branch-width curves (via linear interpolation plus Savitzky-Golay filtering) and rendering polygons whose half-width encodes local cell density, with gene expression overlaid after bicubic smoothing.

## When to use

Use when rendering branching single-cell trajectories as stream plots whose branch width must be proportional to the total number of cells at a given pseudotime, and when gene-expression overlays along branches are needed.

## When NOT to use

- The input is raw single-cell data without an already-inferred branching graph and pseudotime — first run trajectory reconstruction with STREAM/ElPiGraph before visualizing.
- The goal is a trajectory for cyclic or disconnected graph topologies, which STREAM does not properly describe.
- The analysis target is motif/accessibility feature selection for scATAC-seq rather than density-level trajectory visualization.

## Inputs

- anndata object holding a cells x features matrix plus an inferred branching graph and pseudotime values
- Per-branch cell assignments along the trajectory graph
- Optional gene-of-interest expression values to overlay on the stream plot

## Outputs

- stream_plot.png — a PNG figure in which each branch is a polygon whose half-width is the smoothed, normalized local cell density
- Gene-expression overlay along each branch on a dense interpolated grid

## How to apply

Load the single-cell dataset (cells x features, inferred branching graph, pseudotime) using the anndata class that STREAM is written with. For each branch, compute sliding-window cell counts along the branch, including windows that span parent and child branches near each branching node. Normalize the per-branch width curves by the longest path in the trajectory graph so widths are comparable across the plot. Linearly interpolate the width curves and apply a Savitzky-Golay filter to remove sampling noise while preserving branch-shape trends, then render each branch as a polygon whose half-width at each position equals the smoothed, normalized cell density. Compute the gene-expression signal along each branch, interpolate on a dense grid, smooth it bicubically, and overlay it on the rendered polygons.

## Related tools

- **STREAM** (Source pipeline whose density-level stream plot encodes branch width proportional to total cell count at a given pseudotime; this skill reimplements its plotting module) — https://github.com/pinellolab/STREAM
- **anndata** (Data container class used to load the single-cell dataset with branching graph and pseudotime prior to width-curve computation) — http://anndata.rtfd.io
- **Python 3** (Implementation language for the sliding-window counting, interpolation, Savitzky-Golay smoothing, and polygon rendering)

## Evaluation signals

- The output file stream_plot.png exists and is a valid PNG figure (file_exists / file_format checks).
- An llm_judge review confirms branch widths visually scale with local cell counts along pseudotime.
- Width curves are normalized by the longest path in the trajectory graph so branch widths are comparable across the plot.
- Smoothed width curves preserve branch-shape trends (peaks near branching nodes and windows spanning parent/child branches) despite sampling noise removal.
- Gene-expression overlay values change smoothly along each branch, consistent with bicubic smoothing on a dense grid.

## Limitations

- The density-level visualization is designed for tree-like branching trajectories; further improvement is necessary to adapt STREAM to properly describing cyclic or disconnected graph structures.
- Although ElPiGraph, the core algorithm behind STREAM, inherently supports learning complex graphs, more work is necessary to provide intuitive visualizations for such sophisticated topologies.
- Width curves derived from sliding-window counts are sensitive to sampling noise, which is why interpolation and Savitzky-Golay smoothing are required before rendering.

## Evidence

- [discussion] density-level visualization innovation: "Additionally STREAM introduces four innovations compared to other existing methods: (1) a density-level trajectory visualization"
- [methods] anndata data container: "STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)]"
- [readme] STREAM pipeline purpose: "is an interactive pipeline capable of disentangling and visualizing complex branching trajectories from both single-cell transcriptomic and epigenomic data"
- [discussion] visualization limitation for complex topologies: "Although ElPiGraph, the core algorithm behind STREAM, inherently supports learning complex graphs, more work is necessary to provide intuitive visualizations"
