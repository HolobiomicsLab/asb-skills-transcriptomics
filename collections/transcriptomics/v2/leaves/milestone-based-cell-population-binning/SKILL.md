---
name: milestone-based-cell-population-binning
description: Use when a single-cell RNA-seq dataset contains a continuous activation
  or differentiation process (e.g. naive CD4 T cells across 0 h/12 h/24 h stimulation
  with intermediate populations) and you need discrete per-node expression matrices
  for multisample pathway analysis.
when_to_use_negative:
- The analysis goal is a simple two-group comparison (e.g. resting vs activated cells),
  where compare_seurat or two-sample compare_pathways suffices without trajectory
  inference
- The dataset lacks a continuous biological process to order cells along, so milestone
  binning would impose an arbitrary trajectory
- You need graded per-cell trajectory values rather than discrete population-level
  samples for pathway testing
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: dyno
  role: Infers the pseudotime trajectory (ti_slingshot) via infer_trajectory, wraps
    expression/counts with wrap_expression, and bins cells into trajectory nodes with
    group_onto_nearest_milestones
- name: Seurat
  role: Holds the naive CD4 T cell scRNA-seq object whose metadata (milestone column)
    and expression matrices are subset by seurat_extract
  repo: https://github.com/jackbibby1/SCPA
- name: SCPA
  role: Consumes the per-milestone expression matrices as samples in compare_pathways
    to obtain multisample per-pathway qvals across pseudotime
  repo: https://github.com/jackbibby1/SCPA
- name: ComplexHeatmap
  role: Visualizes the resulting qvals heatmap across milestone populations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.celrep.2022.111697
    title: Systematic single-cell pathway analysis to characterize early T cell activation
  source_content_hash: sha256:8cc6e4a12e799ea57f2bd34e7c8724a35e5f715359d8a8305e3970e033bd927a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jackbibby1/SCPA
  tools:
  - ComplexHeatmap
  - SCPA
  - Seurat
  - dyno
  - tidyverse
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# milestone-based-cell-population-binning

## Summary

Bins single cells inferred along a pseudotime trajectory into discrete milestone populations using dyno, so that each trajectory node can be treated as a 'sample' for downstream multisample pathway comparison with SCPA's compare_pathways.

## When to use

Use when a single-cell RNA-seq dataset contains a continuous activation or differentiation process (e.g. naive CD4 T cells across 0 h/12 h/24 h stimulation with intermediate populations) and you need discrete per-node expression matrices for multisample pathway analysis.

## When NOT to use

- The analysis goal is a simple two-group comparison (e.g. resting vs activated cells), where compare_seurat or two-sample compare_pathways suffices without trajectory inference
- The dataset lacks a continuous biological process to order cells along, so milestone binning would impose an arbitrary trajectory
- You need graded per-cell trajectory values rather than discrete population-level samples for pathway testing

## Inputs

- Seurat object (.rds) containing naive/activated cell populations with metadata (e.g. naive_cd4.rds)
- Gene expression and counts matrices selected over the top 1000 most variable genes
- Milestone assignment metadata column ('milestone') produced by group_onto_nearest_milestones

## Outputs

- Per-milestone expression matrices (one list element per trajectory node from seurat_extract)
- Milestone assignments per cell in Seurat metadata
- Sample list suitable for compare_pathways, yielding per-pathway qvals across pseudotime for visualization in ComplexHeatmap or plot_rank

## How to apply

Load the Seurat object (e.g. naive_cd4.rds) and subset to the populations of interest (remove Treg cells, keep naive/activated non-Treg CD4 populations). Select the top 1000 most variable genes and wrap the expression and counts matrices with dyno's wrap_expression, then infer the trajectory with dyno's infer_trajectory using the ti_slingshot method. Assign each cell to a trajectory node with group_onto_nearest_milestones, which populates a 'milestone' metadata column. Extract per-milestone expression matrices by looping seurat_extract over the 'milestone' metadata column; each milestone list element becomes one sample input to SCPA's compare_pathways. The rationale is that SCPA's multisample framework assesses pathway activity as a change in multivariate distribution across multiple groups, so binning pseudotime into milestones converts a continuous trajectory into comparable samples, with the qval as the primary statistic.

## Related tools

- **dyno** (Infers the pseudotime trajectory (ti_slingshot) via infer_trajectory, wraps expression/counts with wrap_expression, and bins cells into trajectory nodes with group_onto_nearest_milestones)
- **Seurat** (Holds the naive CD4 T cell scRNA-seq object whose metadata (milestone column) and expression matrices are subset by seurat_extract) — https://github.com/jackbibby1/SCPA
- **SCPA** (Consumes the per-milestone expression matrices as samples in compare_pathways to obtain multisample per-pathway qvals across pseudotime) — https://github.com/jackbibby1/SCPA
- **ComplexHeatmap** (Visualizes the resulting qvals heatmap across milestone populations)

## Examples

```
dyn <- dyno::infer_trajectory(dyno::wrap_expression(expression, counts), method = "ti_slingshot"); seu$milestone <- dyno::group_onto_nearest_milestones(dyn, dyn$dimred) ; mil_samples <- lapply(unique(seu$milestone), function(m) SCPA::seurat_extract(seu, meta1 = "milestone", value_meta1 = m))
```

## Evaluation signals

- group_onto_nearest_milestones produces a non-null 'milestone' metadata column in the Seurat object covering all retained cells
- The number of samples passed to compare_pathways equals the number of distinct milestones, and each sample matrix contains the selected top 1000 most variable genes
- Per-milestone expression matrices extracted by seurat_extract are non-empty and match the expected cell counts per node
- Internal control: multisample pseudotime results should correlate with a classical two-sample comparison of 0 h resting cells against 12 h intermediate and 24 h activated naive CD4 T cells
- Pathway qvals across milestones show a coherent trend for known metabolic pathways (e.g. glycolysis ranking highly), which can be highlighted via plot_rank

## Limitations

- Binning pseudotime into milestones discretizes a continuous process, so per-node results depend on the milestone resolution and the slingshot trajectory inference
- SCPA output should be interpreted with qval as the primary statistic; fold change/enrichment are secondary and pathways such as arachidonic acid metabolism can show large qvals without enrichment
- Trajectory inference requires filtering to relevant populations (e.g. removing Treg cells) and selecting variable genes before wrap_expression; contaminated populations will distort milestone assignment

## Evidence

- [methods] infer trajectory using ti_slingshot and bin cells into milestones: "And finally running the `infer_trajectory` function using slingshot as the modeller"
- [readme] multisample testing across pseudotime phases motivates milestone binning: [evidence span withheld — anchor status: external_doc]
