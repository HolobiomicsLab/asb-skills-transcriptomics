---
name: single-cell-pseudo-temporal-ordering
description: Use when a cell-to-landmark trajectory graph already exists and individual
  cells must be sequenced along developmental paths between adjacent landmark states
  to recover a pseudo-time axis.
when_to_use_negative:
- The trajectory graph itself has not been built yet — this ordering is the final
  step and depends on upstream landmark construction and cell-to-landmark transition
  assignments
- Development starts from multiple independent progenitor cell types — Mpath explicitly
  does not yet support this case
- The goal is estimating landmark state transitions from neighbourhood counts rather
  than ordering cells along already-defined edges
tools:
- name: Mpath
  role: Provides the landmark trajectory graph and performs the final cell-reordering
    step to generate the pseudo-temporal ordering of individual cells
  repo: https://github.com/JinmiaoChenLab/Mpath
- name: R
  role: Implementation environment for the Mpath package used to run the ordering
    procedure and write cell_ordering.csv
  repo: https://github.com/JinmiaoChenLab/Mpath
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:928aa4be9a3d5e0c49ec79b71714779feff7150cd1c99e646baef7d0d1ed2db9
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
yes_rate: 0.8888888888888888
n_claims: 9
---

# single-cell-pseudo temporal-ordering

## Summary

Places individual single-cell RNA-seq cells in pseudo-temporal order along the multi-branching developmental trajectories constructed by Mpath, by ordering transitioning cells between neighbouring landmark states along each graph edge.

## When to use

Use when a cell-to-landmark trajectory graph already exists and individual cells must be sequenced along developmental paths between adjacent landmark states to recover a pseudo-time axis.

## When NOT to use

- The trajectory graph itself has not been built yet — this ordering is the final step and depends on upstream landmark construction and cell-to-landmark transition assignments
- Development starts from multiple independent progenitor cell types — Mpath explicitly does not yet support this case
- The goal is estimating landmark state transitions from neighbourhood counts rather than ordering cells along already-defined edges

## Inputs

- Mpath landmark graph (landmark coordinates and connecting edges) from the trajectory-construction stage
- Cell-to-landmark membership/transition assignments for the transitioning cell set
- Single-cell expression matrix used to project cells (as used by the Mpath R package)

## Outputs

- cell_ordering.csv with cell identifier, path, edge, projection position, and pseudo-time rank
- A single pseudo-temporal ordering of cells per developmental path, used for temporal gene-regulation analyses (e.g. DEG tables in Supplementary Data 2–5)

## How to apply

Load the Mpath landmark graph (landmark coordinates and connecting edges) together with the cell-to-landmark membership/transition assignments produced by the upstream trajectory-construction stage. For every pair of adjacent landmarks, select the cells assigned as transitioning between them and compute the orthogonal projection of each cell onto the straight edge connecting that landmark pair. Sort cells on each edge by their projection-point coordinate to obtain the local ordering, then concatenate the per-edge ordered segments along each developmental path to yield one pseudo-temporal ordering. Mpath places cells in order as the final step of its framework, enabling downstream analysis such as sequential waves of gene regulation along cDC development and myoblast differentiation. Write the concatenated ordering (cell identifier, path, edge, projection position, pseudo-time rank) to cell_ordering.csv.

## Related tools

- **Mpath** (Provides the landmark trajectory graph and performs the final cell-reordering step to generate the pseudo-temporal ordering of individual cells) — https://github.com/JinmiaoChenLab/Mpath
- **R** (Implementation environment for the Mpath package used to run the ordering procedure and write cell_ordering.csv) — https://github.com/JinmiaoChenLab/Mpath

## Evaluation signals

- cell_ordering.csv contains every cell from the transitioning set exactly once (no duplicates, no missing cells)
- Each cell's pseudo-time rank is consistent with its projection position along its assigned landmark edge
- Per-edge local orderings concatenate monotonically along each developmental path (e.g. MDP → CDP → preDC branches in the cDC data)
- The ordering recapitulates known biology: sequential waves of gene regulation and a switch from proliferation to differentiation during DC maturation
- Downstream DEG analyses along the ordering (e.g. early CDP → late cDC2-committed preDC contrasts) yield coherent phase-dependent gene clusters

## Limitations

- The ordering depends on signature genes of differentiated cell types (end-stage contrasts) and FACS-sorted terminal subsets for fate prediction upstream
- Utility demonstrated only on two data sets: mouse bone marrow-derived DC progenitors and primary human myoblasts
- Ordering is defined only between neighbouring landmark states; cells must first be assigned to landmark transitions by the upstream Mpath stages

## Evidence

- [abstract] Reordering cells along cDC development reveals sequential waves of gene regulation and temporal coupling between cell cycle and cDC differentiation: "Reordering cells along cDC development reveals sequential waves of gene regulation and temporal coupling between cell cycle and cDC differentiation."
- [discussion] Ordering placed individual cells in order along trajectories revealing proliferation-to-differentiation switch: "it placed individual cells in order along the developmental trajectories, revealing a switch from proliferation to differentiation during DC maturation."
- [readme] Mpath constructs multi-branching cell lineages from single-cell data (README): "Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data"
