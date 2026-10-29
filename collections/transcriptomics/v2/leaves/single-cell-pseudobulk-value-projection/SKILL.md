---
name: single-cell-pseudobulk-value-projection
description: Use when pseudotime and branch probabilities were computed at the pseudobulk
  (neighbourhood) level via vdjPseudobulk and must be mapped back to individual cells
  in a miloR SingleCellExperiment, especially when some cells are unassigned to any
  neighbourhood.
when_to_use_negative:
- Values are already at single-cell resolution (no pseudobulk milo object exists)
  — projecting again would double-aggregate.
- The trajectory has a cyclic topology (e.g. B cell receptor editing), which dandelionR's
  absorbing Markov chain pseudotime cannot represent.
- The dataset has too few cells for miloR neighbourhood construction (e.g. only ~972
  cells), leaving most cells without a neighbourhood assignment.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: dandelionR
  role: Provides projectPseudotimeToCell, the function that projects pseudobulk pseudotime
    and branch probabilities back to individual cells
  repo: https://github.com/tuonglab/dandelionR
- name: miloR
  role: Builds the neighbourhood and pseudobulk (vdjPseudobulk) objects whose values
    are projected to cells
- name: Dandelion
  role: Original Python implementation whose result-projecting function this step
    reproduces in R
  repo: https://www.github.com/zktuong/dandelion
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2025.06.047
    title: '<i>dandelionR</i> : Single-cell immune repertoire trajectory analysis
      in R'
  source_content_hash: sha256:cc392fa1e7796e3a2c26fa9bf066d935987d3cc62de05db6abfa836b5b979b92
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tuonglab/dandelionR
  tools:
  - Dandelion
  - R
  - dandelionR
  - miloR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# single-cell-pseudobulk-value-projection

## Summary

Projects pseudobulk-level trajectory results (pseudotime and branching probabilities such as 'CD8+T' and 'CD4+T') from a miloR pseudobulk object back to individual cells using dandelionR::projectPseudotimeToCell, removing cells that belong to no neighbourhood.

## When to use

Use when pseudotime and branch probabilities were computed at the pseudobulk (neighbourhood) level via vdjPseudobulk and must be mapped back to individual cells in a miloR SingleCellExperiment, especially when some cells are unassigned to any neighbourhood.

## When NOT to use

- Values are already at single-cell resolution (no pseudobulk milo object exists) — projecting again would double-aggregate.
- The trajectory has a cyclic topology (e.g. B cell receptor editing), which dandelionR's absorbing Markov chain pseudotime cannot represent.
- The dataset has too few cells for miloR neighbourhood construction (e.g. only ~972 cells), leaving most cells without a neighbourhood assignment.

## Inputs

- miloR neighbourhood object (SingleCellExperiment/Milo) with per-cell metadata and UMAP
- pseudobulk milo object (pb.milo) from vdjPseudobulk with pseudotime and branch probability columns (e.g. 'pseudotime', 'CD8+T', 'CD4+T') in colData
- value_key character vector naming the pseudobulk columns to project

## Outputs

- Projected single-cell SingleCellExperiment with per-cell 'pseudotime' and branch probability columns in colData, unassigned cells removed
- projected_cell_trajectory.rds (saved RDS of the projected object)

## How to apply

Load the miloR neighbourhood object (with per-cell metadata and UMAP from miloUmap) and the pseudobulk milo object (pb.milo) whose colData carries columns such as 'pseudotime', 'CD8+T' and 'CD4+T' from the absorbing Markov chain trajectory step. Call dandelionR::projectPseudotimeToCell(milo_object, pb.milo, value_key = c('pseudotime', 'CD8+T', 'CD4+T')). Cells not belonging to any pseudobulk are removed (39 cells in the Suo et al. fetal immune atlas run), and cells belonging to multiple pseudobulks receive each projected value as a weighted average with weights equal to the inverse of the pseudobulk size. Inspect the resulting colData to confirm all requested value_key columns are present, then save the projected SingleCellExperiment as projected_cell_trajectory.rds. This step reproduces the result-projecting function of the original Python Dandelion software in R.

## Related tools

- **dandelionR** (Provides projectPseudotimeToCell, the function that projects pseudobulk pseudotime and branch probabilities back to individual cells) — https://github.com/tuonglab/dandelionR
- **miloR** (Builds the neighbourhood and pseudobulk (vdjPseudobulk) objects whose values are projected to cells)
- **Dandelion** (Original Python implementation whose result-projecting function this step reproduces in R) — https://www.github.com/zktuong/dandelion

## Examples

```
projected <- dandelionR::projectPseudotimeToCell(milo_object, pb.milo, value_key = c('pseudotime', 'CD8+T', 'CD4+T'))
```

## Evaluation signals

- The output RDS file exists and its colData contains the projected columns for every requested value_key entry ('pseudotime', 'CD8+T', 'CD4+T').
- The retained cell count is consistent with the paper's reported dandelionR output scale (~17,281 cells from the Suo et al. dataset).
- Cells not belonging to any neighbourhood were removed (39 cells removed in the reference run), and the retained count equals original cells minus unassigned cells.
- For cells in multiple pseudobulks, projected values equal the weighted average with inverse-pseudobulk-size weights — spot-check a multi-assignment cell.
- DandelionR-vs-Dandelion cell overlap check: the paper reports 17,281 vs 17,234 cells with 17,208 shared, so the retained set should overlap heavily with the original Python output.

## Limitations

- destiny-based diffusion maps limit dandelionR trajectory inference to bifurcation or linear topologies, constraining what the projected pseudotime can represent.
- Fundamental discrepancies in diffusion map construction and KNN graph construction between R and Python packages mean the projection is not an exact reimplementation of Dandelion (Lin's CCC for CD4+ probability was 0.850, below the commonly accepted 0.9 threshold).
- Cells not assigned to any neighbourhood are dropped irreversibly during projection (39 cells in the reference run).

## Evidence

- [methods] result-projecting function: "As an R implementation of Dandelion, it aims to reproduce the preprocessing, feature-space-building and result-projecting functions of the original software."
- [results] output cell counts: "The dandelionR output includes 17,281 cells, while the original Dandelion output contains 17,234 cells, with 17,208 shared between the two."
- [readme] absorbing markov chain pseudotime: "a custom implementation of absorbing markov chain for pseudotime inference, inspired based on the palantir"
- [discussion] topology limitation: "destiny, is only suitable for a dataset with a bifurcation or linear topology, which limits the versatility of dandelionR’s trajectory inference function"
