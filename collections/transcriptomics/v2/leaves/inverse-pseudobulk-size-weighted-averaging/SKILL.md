---
name: inverse-pseudobulk-size-weighted-averaging
description: Use when a miloR neighbourhood object holds per-cell data and a pseudobulk
  milo object (pb.milo) carries pseudotime/branch probabilities in colData, and you
  need cell-level trajectory values.
when_to_use_negative:
- The pseudobulk object already contains cell-level (not pseudobulk) trajectory values
  — no projection/weighting step is needed
- The trajectory topology is cyclic (e.g. B cell receptor editing), which absorbing-Markov-chain-based
  pseudotime cannot represent
- The dataset is too small for miloR neighbourhood construction (e.g. Tonglin et al.'s
  972 healthy B cells)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: dandelionR
  role: Provides projectPseudotimeToCell for projecting pseudobulk pseudotime and
    branch probabilities back to individual cells with inverse-size weighting
  repo: https://github.com/tuonglab/dandelionR
- name: miloR
  role: Supplies the neighbourhood and pseudobulk (vdjPseudobulk) objects whose assignments
    drive the per-cell projection
- name: R
  role: Implementation language for the projection workflow and SingleCellExperiment
    output
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2025.06.047
    title: '<i>dandelionR</i> : Single-cell immune repertoire trajectory analysis
      in R'
  source_content_hash: sha256:674ec56829cb583418177838003dcd8bfc4993862f58b0b5db0eb7b369b23370
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tuonglab/dandelionR
  tools:
  - Dandelion
  - R
  - dandelionR
  - miloR
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# inverse-pseudobulk-size-weighted-averaging

## Summary

Projects pseudobulk-level trajectory results (pseudotime and branching probabilities from the miloR pb.milo object) back to individual cells in dandelionR via projectPseudotimeToCell, weighting each pseudobulk's contribution by the inverse of its pseudobulk size and dropping cells not assigned to any neighbourhood.

## When to use

Use when a miloR neighbourhood object holds per-cell data and a pseudobulk milo object (pb.milo) carries pseudotime/branch probabilities in colData, and you need cell-level trajectory values.

## When NOT to use

- The pseudobulk object already contains cell-level (not pseudobulk) trajectory values — no projection/weighting step is needed
- The trajectory topology is cyclic (e.g. B cell receptor editing), which absorbing-Markov-chain-based pseudotime cannot represent
- The dataset is too small for miloR neighbourhood construction (e.g. Tonglin et al.'s 972 healthy B cells)

## Inputs

- miloR neighbourhood object with per-cell metadata and UMAP
- pseudobulk milo object (pb.milo) with pseudotime and branch probability columns ('pseudotime', 'CD8+T', 'CD4+T') in colData
- value_key character vector naming the colData columns to project

## Outputs

- SingleCellExperiment with projected value columns in colData and unassigned cells removed
- projected_cell_trajectory.rds (saved projected single-cell object, ~17,281 retained cells for Suo et al.)

## How to apply

Load the miloR neighbourhood object (with per-cell metadata and UMAP) together with the pseudobulk milo object (pb.milo) whose colData contains columns such as 'pseudotime', 'CD8+T' and 'CD4+T'. Call dandelionR::projectPseudotimeToCell(milo_object, pb.milo, value_key = c('pseudotime', 'CD8+T', 'CD4+T')) to transfer the pseudobulk values onto single cells. The function removes cells that do not belong to any pseudobulk (39 cells removed in the Suo et al. run) and, for cells assigned to multiple pseudobulks, computes each projected value as a weighted average with weights equal to the inverse of the pseudobulk size, so smaller neighbourhoods do not dominate. Inspect the resulting SingleCellExperiment colData to confirm the projected value columns are present and cells were dropped, then save the object (e.g. projected_cell_trajectory.rds).

## Related tools

- **dandelionR** (Provides projectPseudotimeToCell for projecting pseudobulk pseudotime and branch probabilities back to individual cells with inverse-size weighting) — https://github.com/tuonglab/dandelionR
- **miloR** (Supplies the neighbourhood and pseudobulk (vdjPseudobulk) objects whose assignments drive the per-cell projection)
- **R** (Implementation language for the projection workflow and SingleCellExperiment output)

## Examples

```
sce_projected <- dandelionR::projectPseudotimeToCell(milo_object, pb.milo, value_key = c('pseudotime', 'CD8+T', 'CD4+T'))
```

## Evaluation signals

- Projected value columns for every entry in value_key ('pseudotime', 'CD8+T', 'CD4+T') are present in the output colData
- Cells not belonging to any neighbourhood are removed (39 cells removed in the paper's run)
- Retained cell count is consistent with the reported dandelionR output scale (~17,281 cells from Suo et al.)
- The output file (projected_cell_trajectory.rds) exists and loads as a valid SingleCellExperiment
- Multi-neighbourhood cells carry weighted averages with inverse-of-pseudobulk-size weights rather than simple means

## Limitations

- Cells not assigned to any miloR neighbourhood are irreversibly dropped during projection
- Cells assigned to multiple pseudobulks receive averaged values, which can blur branch-specific probabilities
- The underlying destiny-based diffusion map limits trajectory inference to bifurcation or linear topologies

## Evidence

- [results] When projecting the pseudobulk-level pseudotime and probabilities to individual cells with dandelionR::projectPseudotimeToCell, a total of 39 cells were removed because they did not belong to any neighbourhoods.: [evidence span withheld — anchor status: not_found]
- [intro] dandelionR enables the construction of the VDJ feature space to perform trajectory analysis using diffusion maps and absorbing Markov chains, with seamless interaction with scRepertoire.: "dandelionR enables the construction of the VDJ feature space to perform trajectory analysis using diffusion maps and absorbing Markov chains"
- [results] The dandelionR output includes 17,281 cells, while the original Dandelion output contains 17,234 cells, with 17,208 shared between the two.: "The dandelionR output includes 17,281 cells, while the original Dandelion output contains 17,234 cells"
- [methods] Pseudobulking can be achieved through the miloR (v2.0.0) package.: "Pseudobulking can be achieved through the miloR (v2.0.0) package"
