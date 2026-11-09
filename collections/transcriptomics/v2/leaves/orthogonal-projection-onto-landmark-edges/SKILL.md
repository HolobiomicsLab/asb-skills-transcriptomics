---
name: orthogonal-projection-onto-landmark-edges
description: Use when a Mpath landmark graph (landmark coordinates and connecting
  edges) and cell-to-landmark transition assignments exist and individual cells must
  be placed in pseudo-temporal order along developmental trajectories between neighbouring
  landmarks.
when_to_use_negative:
- Development process starts from multiple independent progenitor cell types — the
  article states Mpath does not yet support this (planned future extension)
- Highly heterogeneous single-cell data sets where specific subset information is
  unknown — Mpath is not yet extended to this setting
- You need a global diffusion-pseudotime value per cell rather than a per-edge ordering
  concatenated along landmarks; this skill orders cells only along Mpath landmark
  edges
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Mpath
  role: R package providing the landmark graph and cell-ordering step that places
    individual cells in order along developmental trajectories
  repo: JinmiaoChenLab/Mpath
- name: R
  role: Runtime environment in which the Mpath reference implementation of projection
    and ordering is executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:d475462afe8a64cad510cdffe93bc6c7a519d99e4cbcf42aaf25a091725024c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JinmiaoChenLab/Mpath
  tools:
  - Mpath
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# orthogonal-projection-onto-landmark-edges

## Summary

Final step of the Mpath framework: cells transitioning between pairs of adjacent landmark states are orthogonally projected onto the straight edge connecting the landmark pair, and sorted by projection coordinate to produce a pseudo-temporal ordering of single cells along each developmental trajectory.

## When to use

Use when a Mpath landmark graph (landmark coordinates and connecting edges) and cell-to-landmark transition assignments exist and individual cells must be placed in pseudo-temporal order along developmental trajectories between neighbouring landmarks.

## When NOT to use

- Development process starts from multiple independent progenitor cell types — the article states Mpath does not yet support this (planned future extension)
- Highly heterogeneous single-cell data sets where specific subset information is unknown — Mpath is not yet extended to this setting
- You need a global diffusion-pseudotime value per cell rather than a per-edge ordering concatenated along landmarks; this skill orders cells only along Mpath landmark edges

## Inputs

- Mpath landmark graph: landmark state coordinates and connecting edges between neighbouring landmark states
- Cell-to-landmark membership/transition assignments from Mpath's neighborhood-based cell state transition stage
- Single-cell expression matrix or dimensionality-reduced cell coordinates underlying the landmark space

## Outputs

- Pseudo-temporal ordering of individual cells along each developmental trajectory between neighbouring landmark states
- cell_ordering.csv with per-cell records: cell identifier, path, edge, projection position, pseudo-time rank

## How to apply

Load the Mpath landmark graph (landmark coordinates plus connecting edges) and the cell-to-landmark membership/transition assignments produced by the upstream neighborhood-based trajectory-construction stage. For each pair of adjacent landmarks, select the cells assigned as transitioning between them and compute the orthogonal projection of each cell onto the straight edge connecting that landmark pair. Sort cells on each edge by their projection point coordinate to obtain a local ordering, then concatenate the per-edge ordered segments along each developmental path into a single pseudo-time ranking. The rationale, per the article, is that Mpath derives multi-branching trajectories via neighborhood-based cell state transitions and, as the last step, places individual cells in order along those trajectories so that downstream analyses (e.g., waves of gene regulation, cell cycle coupling) can be read off the ordering.

## Related tools

- **Mpath** (R package providing the landmark graph and cell-ordering step that places individual cells in order along developmental trajectories) — JinmiaoChenLab/Mpath
- **R** (Runtime environment in which the Mpath reference implementation of projection and ordering is executed)

## Evaluation signals

- cell_ordering.csv contains every cell from the transitioning set exactly once — no duplicates and no dropped cells
- Within each landmark edge, cell order is strictly non-decreasing in the recorded projection position along that edge
- Concatenated ordering respects trajectory topology: cells on a branch appear only along the edges of that path (e.g., cDP → preDC branches), not on unrelated edges
- Downstream sanity check: reordering along cDC development should recapitulate the reported switch from proliferation to differentiation during DC maturation
- Per-edge segment lengths are bounded by the number of cells assigned as transitioning between that adjacent landmark pair

## Limitations

- Mpath does not yet support development processes starting from multiple independent progenitor cell types
- Mpath is not yet extended to highly heterogeneous single-cell data sets where specific subset information is unknown
- Utility demonstrated only on mouse bone marrow-derived DC progenitors and primary human myoblasts data sets
- Ordering is local to each landmark edge and depends on the quality of the upstream landmark graph and transition assignments

## Evidence

- [abstract] Reordering cells along cDC development reveals sequential waves of gene regulation and temporal coupling between cell cycle and cDC differentiation.: "Reordering cells along cDC development reveals sequential waves of gene regulation and temporal coupling between cell cycle and cDC differentiation."
- [discussion] it placed individual cells in order along the developmental trajectories, revealing a switch from proliferation to differentiation during DC maturation.: "it placed individual cells in order along the developmental trajectories, revealing a switch from proliferation to differentiation during DC maturation."
- [abstract] Mpath, an algorithm that derives multi-branching developmental trajectories using neighborhood-based cell state transitions: "Mpath, an algorithm that derives multi-branching developmental trajectories using neighborhood-based cell state transitions"
- [readme] Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data: "Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data"
