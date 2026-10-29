---
name: trajectory-graph-edge-traversal
description: Use when a landmark-based trajectory graph (landmark coordinates, connecting
  edges, and cell-to-landmark transition assignments) is already built and cells must
  be given a pseudo-temporal ordering along each edge of the tree.
when_to_use_negative:
- No landmark graph exists yet — this ordering step presupposes Mpath's landmark states
  and edges derived by neighborhood-based cell state transition mapping
- The developmental process starts from multiple independent progenitor cell types
  — Mpath does not yet support this (planned future extension)
- The goal is de novo discovery of subset identity in highly heterogeneous data where
  specific subset information is unknown — Mpath relies on signature genes of differentiated
  cell types and FACS-sorted terminal subsets for fate prediction
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Mpath
  role: Provides the landmark graph and cell state transitions whose edges are traversed
    and projected onto to produce the pseudo-temporal cell ordering
  repo: https://github.com/JinmiaoChenLab/Mpath
- name: R
  role: Implementation environment for the Mpath R package used to run the ordering
    step
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:1afdfa3f9658120d76e91a80bfd1c2c7162dced299fa8c7245a09a28e415bef0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JinmiaoChenLab/Mpath
  tools:
  - Mpath
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# trajectory-graph-edge-traversal

## Summary

Final step of the Mpath framework: individual single-cell RNA-seq cells are placed in order along the developmental trajectories between every pair of neighbouring landmark states, producing the pseudo-temporal ordering used to reveal sequential waves of gene regulation during cDC development and myoblast differentiation.

## When to use

Use when a landmark-based trajectory graph (landmark coordinates, connecting edges, and cell-to-landmark transition assignments) is already built and cells must be given a pseudo-temporal ordering along each edge of the tree.

## When NOT to use

- No landmark graph exists yet — this ordering step presupposes Mpath's landmark states and edges derived by neighborhood-based cell state transition mapping
- The developmental process starts from multiple independent progenitor cell types — Mpath does not yet support this (planned future extension)
- The goal is de novo discovery of subset identity in highly heterogeneous data where specific subset information is unknown — Mpath relies on signature genes of differentiated cell types and FACS-sorted terminal subsets for fate prediction

## Inputs

- Mpath landmark graph: landmark coordinates and connecting edges
- Cell-to-landmark membership/transition assignments from the trajectory-construction stage
- Single-cell expression matrix (single-cell RNA-sequencing data, e.g. mouse cDC progenitors or human myoblasts)

## Outputs

- Pseudo-temporal ordering of individual cells along each developmental path between neighbouring landmark states
- Per-cell ordering table (cell identifier, path, edge, projection position, pseudo-time rank), e.g. cell_ordering.csv

## How to apply

Load the Mpath landmark graph (landmark coordinates and connecting edges) together with the cell-to-landmark membership/transition assignments produced by the neighborhood-based trajectory-construction stage. For each pair of adjacent landmarks, select the cells assigned as transitioning between them and compute the orthogonal projection of each cell onto the straight edge connecting the landmark pair. Sort cells on each edge by their projection-point coordinate to obtain a local ordering between the two landmarks, then concatenate the per-edge ordered segments along each developmental path (travelling back in time from the differentiated/terminal subsets toward progenitors) to yield a single pseudo-temporal ordering. The rationale is that Mpath resolves multi-branching trajectories with multiple fates from one progenitor type, so ordering must be done per edge and then stitched along paths rather than globally. Write the result (cell identifier, path, edge, projection position, pseudo-time rank) and verify every transitioning cell appears exactly once.

## Related tools

- **Mpath** (Provides the landmark graph and cell state transitions whose edges are traversed and projected onto to produce the pseudo-temporal cell ordering) — https://github.com/JinmiaoChenLab/Mpath
- **R** (Implementation environment for the Mpath R package used to run the ordering step)

## Evaluation signals

- Every cell in the transitioning set appears exactly once in the concatenated ordering (no duplicates or omissions)
- Each per-edge cell segment is monotonically sorted by projection-point coordinate between its two adjacent landmarks
- Ordering recapitulates known biology: e.g. placement of cells along cDC development should reveal the documented switch from proliferation to differentiation during DC maturation
- Ordered cells align with end-stage/FACS-defined terminal subsets when the algorithm travels back in time from differentiated cell subsets
- Concatenated paths connect through shared landmark states without gaps between per-edge segments

## Limitations

- Mpath does not yet support development processes starting from multiple independent progenitor cell types (planned future extension)
- Mpath is not yet extended to highly heterogeneous single-cell data sets where specific subset information is unknown (planned future extension)
- Mpath depends on signature genes of differentiated cell types (end-stage contrasts) and FACS-sorted terminal subsets for fate prediction, so ordering quality is bounded by the quality of those references

## Evidence

- [discussion] it placed individual cells in order along the developmental trajectories, revealing a switch from proliferation to differentiation during DC maturation.: "it placed individual cells in order along the developmental trajectories, revealing a switch from proliferation to differentiation during DC maturation."
- [abstract] Reordering cells along cDC development reveals sequential waves of gene regulation and temporal coupling between cell cycle and cDC differentiation.: "Reordering cells along cDC development reveals sequential waves of gene regulation and temporal coupling between cell cycle and cDC differentiation."
- [discussion] The algorithm travels back in time from differentiated cell subsets and maps the developmental path from known terminal differentiated cell subsets back to its progenitors.: "The algorithm travels back in time from differentiated cell subsets and maps the developmental path from known terminal differentiated cell subsets back to its progenitors."
- [discussion] Mpath uses count of neighbouring cells to estimate potential state transition. Via neighborhood-based cell state transition mapping: "Mpath uses count of neighbouring cells to estimate potential state transition. Via neighborhood-based cell state transition mapping"
- [readme] Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data: "Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data"
