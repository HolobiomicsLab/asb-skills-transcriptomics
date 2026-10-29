---
name: r-based-cell-ordering-implementation
description: Use when an Mpath landmark graph (landmark coordinates, edges, and cell-to-landmark
  transition assignments) exists and cells must be assigned a pseudo-temporal ordering
  along each edge between adjacent landmarks.
when_to_use_negative:
- When no Mpath landmark graph or cell-to-landmark transition assignments exist yet
  — order cells only after the neighborhood-based trajectory-construction stage completes
- When the goal is trajectory construction itself rather than reordering cells along
  an already-built trajectory
- When the developmental process starts from multiple independent progenitor cell
  types — Mpath does not yet support this scenario
tools:
- name: Mpath
  role: R package implementing landmark-based trajectory construction and the final
    cell reordering step
  repo: JinmiaoChenLab/Mpath
- name: R
  role: Runtime environment for executing the Mpath ordering routine
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:9e8c302886779cc8d07ea32db828562f8a825c28d7e14e9cd9c6f8b4824645a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JinmiaoChenLab/Mpath
  tools:
  - Mpath
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# r-based-cell-ordering-implementation

## Summary

Final step of the Mpath framework: places individual cells in pseudo-temporal order along the developmental trajectories between every pair of neighbouring landmark states, implemented as an R routine in the Mpath package.

## When to use

Use when an Mpath landmark graph (landmark coordinates, edges, and cell-to-landmark transition assignments) exists and cells must be assigned a pseudo-temporal ordering along each edge between adjacent landmarks.

## When NOT to use

- When no Mpath landmark graph or cell-to-landmark transition assignments exist yet — order cells only after the neighborhood-based trajectory-construction stage completes
- When the goal is trajectory construction itself rather than reordering cells along an already-built trajectory
- When the developmental process starts from multiple independent progenitor cell types — Mpath does not yet support this scenario

## Inputs

- Mpath landmark graph: landmark coordinates and connecting edges (R object from upstream Mpath trajectory construction)
- Cell-to-landmark membership/transition assignments for transitioning cells
- Single-cell expression matrix used by the Mpath R package

## Outputs

- cell_ordering.csv containing per-cell identifier, path, edge, projection position, and pseudo-time rank
- A single concatenated pseudo-temporal ordering of cells along each developmental path

## How to apply

Load the Mpath landmark graph and the cell-to-landmark membership/transition assignments produced by the upstream neighborhood-based trajectory-construction stage in R. For each pair of adjacent landmarks, take the cells assigned as transitioning between them and compute the orthogonal projection of each cell onto the straight edge connecting that landmark pair. Sort cells on each edge by their projection coordinate to obtain a local ordering, then concatenate the per-edge ordered segments along each developmental path into a single pseudo-temporal ordering. Write the result (cell identifier, path, edge, projection position, pseudo-time rank) to cell_ordering.csv and confirm every transitioning cell appears exactly once. The rationale is that Mpath 'places individual cells in order along the developmental trajectories', which downstream analysis (e.g. detecting waves of gene regulation and the proliferation-to-differentiation switch during DC maturation) depends on.

## Related tools

- **Mpath** (R package implementing landmark-based trajectory construction and the final cell reordering step) — JinmiaoChenLab/Mpath
- **R** (Runtime environment for executing the Mpath ordering routine)

## Examples

```
Rscript -e "library(Mpath); ordering <- read.csv('cell_ordering.csv'); stopifnot(!any(duplicated(ordering$cell_id)))"
```

## Evaluation signals

- cell_ordering.csv contains every cell from the transitioning set exactly once (no duplicates, no missing cells)
- Each cell's projection position lies on (or within tolerance of) the straight edge between its two adjacent landmarks
- Per-edge orderings are monotonic in projection coordinate before concatenation
- Concatenated ordering reproduces the expected path structure: ordered segments follow the landmark edge sequence of each developmental path
- Downstream sanity check: reordering of cDC development cells reveals the expected sequential waves of gene regulation and proliferation-to-differentiation switch

## Limitations

- Mpath depends on signature genes of differentiated cell types (end-stage contrasts) and FACS-sorted terminal subsets for fate prediction, which constrains the trajectories that cells can be ordered along
- Mpath does not yet support developmental processes starting from multiple independent progenitor cell types (planned future extension)
- Demonstrated utility is limited to two data sets: mouse bone marrow-derived DC progenitors and primary human myoblasts

## Evidence

- [discussion] Final-step reordering finding: "it placed individual cells in order along the developmental trajectories, revealing a switch from proliferation to differentiation during DC maturation"
- [abstract] Reordering utility: "Reordering cells along cDC development reveals sequential waves of gene regulation and temporal coupling between cell cycle and cDC differentiation"
- [abstract] Trajectory basis: "Mpath, an algorithm that derives multi-branching developmental trajectories using neighborhood-based cell state transitions"
- [supplementary] Package documentation: "Mpath reference manual provides detailed documentation of Mpath R package."
- [readme] README: "Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data"
