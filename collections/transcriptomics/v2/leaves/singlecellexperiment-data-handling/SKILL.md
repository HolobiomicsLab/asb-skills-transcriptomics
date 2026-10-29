---
name: singlecellexperiment-data-handling
description: Use when single-cell VDJ/scRNA-seq data are stored in SingleCellExperiment
  objects (e.g. demo_sce, sce_vdj, miloR neighbourhood objects) and colData-level
  columns such as pseudotime or CD4+/CD8+ branch probabilities must be added, inspected,
  or projected onto cells.
when_to_use_negative:
- Input is a Python AnnData object — this workflow operates on R SingleCellExperiment
  objects; use the original Python dandelion package instead
- The trajectory has a cyclic topology (e.g. B cell receptor editing), which cannot
  be represented by the absorbing Markov chain pseudotime stored in the SCE colData
- The dataset lacks sufficient cells for miloR neighbourhood construction (e.g. Tonglin
  et al.'s 972 healthy B cells), so no pseudobulk colData columns exist to project
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: dandelionR
  role: Provides projectPseudotimeToCell and setupVdjPseudobulk for moving pseudobulk
    trajectory results in and out of SingleCellExperiment colData
  repo: https://github.com/tuonglab/dandelionR
- name: miloR
  role: Builds neighbourhoods and pseudobulk objects (v2.0.0) whose colData carries
    pseudotime and branch probabilities projected back to cells
  repo: https://github.com/tuonglab/dandelionR
- name: scRepertoire
  role: combineTCR/combineExpression attach filtered TCR contigs onto the SingleCellExperiment
    before trajectory handling
  repo: https://github.com/ncborcherding/scRepertoire
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2025.06.047
    title: '<i>dandelionR</i> : Single-cell immune repertoire trajectory analysis
      in R'
  source_content_hash: sha256:bd2f92fd7640a63b88f95cfd717347e4be61b2b7c44b23ac65cda59758319623
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tuonglab/dandelionR
  tools:
  - Dandelion
  - R
  - dandelionR
  - miloR
  - scRepertoire
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# singlecellexperiment-data-handling

## Summary

Handles single-cell data as SingleCellExperiment (SCE) objects throughout the dandelionR immune-repertoire trajectory workflow — from combineExpression contig-to-SCE attachment through pseudobulk (miloR) colData manipulation and final projection of pseudotime/branch probabilities back to individual cells.

## When to use

Use when single-cell VDJ/scRNA-seq data are stored in SingleCellExperiment objects (e.g. demo_sce, sce_vdj, miloR neighbourhood objects) and colData-level columns such as pseudotime or CD4+/CD8+ branch probabilities must be added, inspected, or projected onto cells.

## When NOT to use

- Input is a Python AnnData object — this workflow operates on R SingleCellExperiment objects; use the original Python dandelion package instead
- The trajectory has a cyclic topology (e.g. B cell receptor editing), which cannot be represented by the absorbing Markov chain pseudotime stored in the SCE colData
- The dataset lacks sufficient cells for miloR neighbourhood construction (e.g. Tonglin et al.'s 972 healthy B cells), so no pseudobulk colData columns exist to project

## Inputs

- SingleCellExperiment object with scRNA-seq counts and reduced dims (e.g. 'X_scvi')
- AIRR/VDJ contig list processed by scRepertoire::combineTCR
- miloR neighbourhood object with per-cell metadata and UMAP
- Pseudobulk milo object (pb.milo) with 'pseudotime' and branch probability columns in colData
- value_key character vector naming colData columns to project (e.g. c('pseudotime', 'CD8+T', 'CD4+T'))

## Outputs

- SingleCellExperiment with projected pseudotime/branch probability columns in colData (cells not in any neighbourhood removed)
- RDS file, e.g. projected_cell_trajectory.rds, retaining ~17,281 cells from the Suo et al. dataset

## How to apply

Convert AIRR contigs and attach them to an existing SingleCellExperiment via scRepertoire::combineExpression after combineTCR filtering (removeNA = TRUE, filterMulti = TRUE). Build miloR neighbourhoods on the SCE (buildGraph k = 30, d = 20 on 'X_scvi'; makeNhoods prop = 0.3) so pseudobulk-level results live in the pb.milo colData alongside 'pseudotime' and branch columns (e.g. 'CD8+T', 'CD4+T'). Project pseudobulk values back to single cells with dandelionR::projectPseudotimeToCell(milo_object, pb.milo, value_key = ...): cells not belonging to any neighbourhood are removed (39 cells in the Suo et al. run), and cells belonging to multiple pseudobulks receive inverse-pseudobulk-size weighted averages. Inspect the resulting colData to confirm all value_key columns are present, then save the projected SCE as an RDS (projected_cell_trajectory.rds).

## Related tools

- **dandelionR** (Provides projectPseudotimeToCell and setupVdjPseudobulk for moving pseudobulk trajectory results in and out of SingleCellExperiment colData) — https://github.com/tuonglab/dandelionR
- **miloR** (Builds neighbourhoods and pseudobulk objects (v2.0.0) whose colData carries pseudotime and branch probabilities projected back to cells) — https://github.com/tuonglab/dandelionR
- **scRepertoire** (combineTCR/combineExpression attach filtered TCR contigs onto the SingleCellExperiment before trajectory handling) — https://github.com/ncborcherding/scRepertoire

## Examples

```
proj <- dandelionR::projectPseudotimeToCell(milo_object, pb.milo, value_key = c('pseudotime', 'CD8+T', 'CD4+T'))
```

## Evaluation signals

- The output SCE colData contains all requested value_key entries ('pseudotime', 'CD8+T', 'CD4+T')
- Retained cell count is consistent with the paper scale (~17,281 cells from the Suo et al. dataset; 17,208 shared with original Dandelion's 17,234)
- Cells not assigned to any neighbourhood are absent from the output (39 cells removed in the reference run)
- Cells belonging to multiple pseudobulks have each projected value equal to an inverse-pseudobulk-size weighted average
- The saved RDS file exists and loads as a SingleCellExperiment with the projected columns intact

## Limitations

- destiny-based diffusion map limits trajectory inference to bifurcation or linear topologies
- Cells not belonging to any neighbourhood are permanently dropped during projection, shrinking the output
- Fundamental discrepancies in diffusion map and KNN graph construction between R and Python packages mean outputs differ from the original Dandelion (17,281 vs 17,234 cells)

## Evidence

- [results] dandelionR output scale: "The dandelionR output includes 17,281 cells, while the original Dandelion output contains 17,234 cells, with 17,208 shared between the two."
- [methods] pseudobulking via miloR: "Pseudobulking can be achieved through the miloR (v2.0.0) package [7]."
- [readme] scRepertoire interfacing: [evidence span withheld — anchor status: not_found]
- [methods] dandelionR install: "devtools::install_github("tuonglab/dandelionR", dependencies = TRUE)"
- [methods] result projection scope: "As an R implementation of Dandelion, it aims to reproduce the preprocessing, feature-space-building and result-projecting functions of the original software."
