---
name: bicubic-gene-expression-overlay-interpolation
description: Use when a single-cell dataset already has an inferred branching graph
  and pseudotime (e.g. from STREAM/ElPiGraph stored in anndata) and you must render
  a density-level trajectory visualization with gene-expression overlays.
when_to_use_negative:
- The trajectory is cyclic or disconnected rather than a branching tree — the article
  notes STREAM needs further improvement to adapt to these more general structures
- No inferred branching graph or pseudotime exists yet — this skill visualizes reconstructed
  trajectories, it does not infer them
- The goal is displaying single cells or clusters along pseudotime rather than a density-level
  visualization of subpopulation composition along branches
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STREAM
  role: Provides the density-level stream plot visualization being reconstructed;
    branch widths encode cell counts and gene-expression overlays
  repo: https://github.com/pinellolab/STREAM
- name: anndata
  role: Data class STREAM is written with; holds the cells x features matrix, branching
    graph, and pseudotime consumed by the plotting routine
  repo: http://anndata.rtfd.io
- name: Python 3
  role: Runtime environment (installed via Python3 Anaconda / conda env) in which
    the stream-plot rendering module is implemented
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-09670-4
    title: Single-cell trajectories reconstruction, exploration and mapping of omics
      data with STREAM
  source_content_hash: sha256:106ce6a3adab3a32dff9b3149666b8c03521cea143ee900fc659c12a25a7d6e7
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# bicubic-gene-expression-overlay-interpolation

## Summary

Reconstructs STREAM's density-level stream plot module: per-branch cell-density width encoding (sliding-window counts, Savitzky-Golay smoothing, longest-path normalization) with a continuously interpolated gene-expression overlay rendered on branch polygons.

## When to use

Use when a single-cell dataset already has an inferred branching graph and pseudotime (e.g. from STREAM/ElPiGraph stored in anndata) and you must render a density-level trajectory visualization with gene-expression overlays.

## When NOT to use

- The trajectory is cyclic or disconnected rather than a branching tree — the article notes STREAM needs further improvement to adapt to these more general structures
- No inferred branching graph or pseudotime exists yet — this skill visualizes reconstructed trajectories, it does not infer them
- The goal is displaying single cells or clusters along pseudotime rather than a density-level visualization of subpopulation composition along branches

## Inputs

- Single-cell dataset as an anndata object (cells x features matrix)
- Inferred branching trajectory graph (e.g. ElPiGraph principal graph as used by STREAM)
- Pseudotime values per cell along the branching graph
- Gene-expression values for a gene of interest to overlay on the stream plot

## Outputs

- stream_plot.png — a valid PNG stream plot with branch polygons whose widths encode local cell density
- Continuous gene-expression overlay along each branch of the stream plot

## How to apply

Load the single-cell dataset (cells x features plus inferred branching graph and pseudotime) into the anndata class that STREAM is written with. For each branch, compute sliding-window cell counts along the branch, including windows that span parent and child branches near each branching node, then normalize the per-branch width curves by the longest path in the trajectory graph so widths are comparable across the plot. Linearly interpolate the width curves and smooth them with a Savitzky-Golay filter to remove sampling noise while preserving branch-shape trends. Compute the gene-expression signal along each branch, interpolate it on a dense grid, and smooth it bicubically to obtain the continuous expression overlay. Render each branch as a polygon whose half-width at each position equals the smoothed, normalized cell density — in the stream plot, at a given pseudotime the width of each branch is proportional to the total number of cells — and overlay the bicubic-smoothed gene-expression signal on the polygons, saving the result as stream_plot.png.

## Related tools

- **STREAM** (Provides the density-level stream plot visualization being reconstructed; branch widths encode cell counts and gene-expression overlays) — https://github.com/pinellolab/STREAM
- **anndata** (Data class STREAM is written with; holds the cells x features matrix, branching graph, and pseudotime consumed by the plotting routine) — http://anndata.rtfd.io
- **Python 3** (Runtime environment (installed via Python3 Anaconda / conda env) in which the stream-plot rendering module is implemented)

## Examples

```
conda create -n env_stream python=3.7 stream=1.0 jupyter
```

## Evaluation signals

- Output stream_plot.png exists and is a valid PNG figure
- Branch widths in the rendered polygons are proportional to local cell counts at each pseudotime (verifiable by llm_judge review against raw sliding-window counts)
- Width curves are normalized by the longest path in the trajectory graph, so branch widths are comparable across the plot
- Savitzky-Golay smoothing removes sampling noise from width curves while preserving branch-shape trends (no spurious kinks at branching nodes)
- Gene-expression overlay is continuous along each branch after dense-grid interpolation and bicubic smoothing

## Limitations

- Although ElPiGraph, the core algorithm behind STREAM, inherently supports learning complex graphs, more work is necessary to provide intuitive visualizations
- Further improvement is necessary to adapt STREAM to properly describing cyclic or disconnected graph structures
- Width encoding depends on the branching graph and pseudotime; windows spanning parent and child branches near branching nodes require special handling or widths will be discontinuous

## Evidence

- [methods] anndata data class: "STREAM is written using the class `anndata` [Wolf et al. Genome Biology (2018)]"
- [discussion] density-level visualization innovation: "Additionally STREAM introduces four innovations compared to other existing methods: (1) a density-level trajectory visualization"
- [intro] motivation for density-level stream plots: "efforts on trajectory visualization remain limited. Current methods mainly focus on displaying single cells or clusters (or stable states) along the pseudotime"
- [readme] stream plot scope: "an interactive pipeline capable of disentangling and visualizing complex branching trajectories from both single-cell transcriptomic and epigenomic data."
