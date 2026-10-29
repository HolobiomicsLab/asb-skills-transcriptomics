---
name: pseudotime-trajectory-inference
description: Use when scRNA-seq captures a continuous biological transition (e.g.
  naive CD4+ T cell activation through intermediate populations at 0 h, 12 h, 24 h)
  and you need pathway dynamics along the trajectory rather than a two-group comparison.
when_to_use_negative:
- The question is a simple two-population comparison (e.g. 0 h resting vs 12 h intermediate
  cells) — a classical two-sample compare_pathways call is the appropriate design,
  as used as the internal control in the pseudotime analysis
- Cells form discrete, unrelated clusters with no continuous transition — ordering
  them on pseudotime milestones would be unjustified
- Input is bulk RNA-seq or an already-aggregated feature table — trajectory inference
  needs per-cell expression distributions
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: dyno
  role: Wraps counts/expression matrices (wrap_expression), infers the pseudotime
    trajectory with ti_slingshot via infer_trajectory, and bins cells onto nodes with
    group_onto_nearest_milestones
- name: SCPA
  role: Runs multisample compare_pathways over the milestone expression matrices to
    produce per-pathway qvals across the pseudotime trajectory
  repo: https://github.com/jackbibby1/SCPA
- name: Seurat
  role: Loads and subsets the Seurat object (e.g. removing Treg cells) and provides
    the object from which seurat_extract pulls milestone expression matrices
- name: ComplexHeatmap
  role: Plots the per-pathway qval matrix across trajectory milestones as a heatmap
- name: tidyverse
  role: Data wrangling and plotting support for looping seurat_extract over milestones
    and generating plot_rank figures
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.celrep.2022.111697
    title: Systematic single-cell pathway analysis to characterize early T cell activation
  source_content_hash: sha256:1fbdb785ef677a4a4327772bea6fcbc45cc6d2123ada439bf4d7ee045bf73cb4
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# pseudotime-trajectory-inference

## Summary

Infer a pseudotime trajectory from single-cell RNA-seq data using dyno (ti_slingshot), bin cells onto trajectory milestones, and feed the milestone populations into SCPA's multisample compare_pathways to quantify pathway activity across differentiation or activation transitions.

## When to use

Use when scRNA-seq captures a continuous biological transition (e.g. naive CD4+ T cell activation through intermediate populations at 0 h, 12 h, 24 h) and you need pathway dynamics along the trajectory rather than a two-group comparison.

## When NOT to use

- The question is a simple two-population comparison (e.g. 0 h resting vs 12 h intermediate cells) — a classical two-sample compare_pathways call is the appropriate design, as used as the internal control in the pseudotime analysis
- Cells form discrete, unrelated clusters with no continuous transition — ordering them on pseudotime milestones would be unjustified
- Input is bulk RNA-seq or an already-aggregated feature table — trajectory inference needs per-cell expression distributions

## Inputs

- Seurat object (.rds) containing populations along a continuous transition, e.g. naive_cd4.rds with naive/intermediate/activated CD4 T cells
- Pathway gene set file (CSV or GMT), e.g. combined_metabolic_pathways.csv of Hallmark/KEGG/Reactome metabolic pathways
- Cell metadata column identifying populations or conditions used to subset cells before trajectory inference

## Outputs

- dyno trajectory object with milestones, inferred via ti_slingshot
- Per-milestone expression matrices from seurat_extract
- compare_pathways results table of per-pathway qvals across pseudotime milestones
- ComplexHeatmap of qvals and plot_rank figure highlighting top-ranked pathways

## How to apply

Load the Seurat object (e.g. naive_cd4.rds) with Seurat, subset out unwanted populations such as Treg cells, and select the top 1000 most variable genes before wrapping the counts/expression matrices with dyno's wrap_expression. Infer the trajectory with infer_trajectory using the ti_slingshot method, then assign each cell to a trajectory node via group_onto_nearest_milestones. Extract per-milestone expression matrices by looping seurat_extract over the 'milestone' metadata column, so each milestone becomes one sample in a multisample design. Run compare_pathways on the list of milestone samples against a pathway file such as combined_metabolic_pathways.csv (Hallmark/KEGG/Reactome metabolic gene sets) to obtain per-pathway qvals across pseudotime. Judge results by qval — the article states qval is the primary statistic for biological relevance, with fold change/enrichment only secondary — and visualize with a ComplexHeatmap of qvals and a plot_rank figure highlighting a top pathway (e.g. glycolysis).

## Related tools

- **dyno** (Wraps counts/expression matrices (wrap_expression), infers the pseudotime trajectory with ti_slingshot via infer_trajectory, and bins cells onto nodes with group_onto_nearest_milestones)
- **SCPA** (Runs multisample compare_pathways over the milestone expression matrices to produce per-pathway qvals across the pseudotime trajectory) — https://github.com/jackbibby1/SCPA
- **Seurat** (Loads and subsets the Seurat object (e.g. removing Treg cells) and provides the object from which seurat_extract pulls milestone expression matrices)
- **ComplexHeatmap** (Plots the per-pathway qval matrix across trajectory milestones as a heatmap)
- **tidyverse** (Data wrangling and plotting support for looping seurat_extract over milestones and generating plot_rank figures)

## Examples

```
dyn_obj <- wrap_expression(counts, expression); trajectory <- infer_trajectory(dyn_obj, method = "ti_slingshot"); trajectory <- group_onto_nearest_milestones(trajectory)
```

## Evaluation signals

- Trajectory inference succeeds with ti_slingshot and every cell is assigned to a milestone by group_onto_nearest_milestones (no unassigned cells in the 'milestone' metadata column)
- seurat_extract returns one expression matrix per milestone, and each matrix contains the selected top-1000 variable genes
- compare_pathways returns a qval for every pathway in combined_metabolic_pathways.csv across all milestone samples
- Multisample qvals along the trajectory are directionally consistent with the classical two-sample control comparing 0 h resting cells against 12 h intermediate and 24 h activated naive CD4 T cells
- Ranking by qval places expected metabolic shifts (e.g. glycolysis) among top pathways, with qval treated as the primary statistic rather than fold change

## Limitations

- The pseudotime analysis requires internal validation against a two-sample comparison of real-time data; qval dynamics inferred from milestone binning should not be over-interpreted in isolation
- qval should be the primary statistic; fold change/enrichment can be low or absent even for highly changed pathways (e.g. arachidonic acid metabolism), so enrichment-only interpretation will miss real biology
- SCPA installation requires specific dependency versions (crossmatch 1.3.1, multicross 2.1.0), and users may need to manually install dependencies if errors occur

## Evidence

- [readme] A tutorial for multisample SCPA, comparing pathways across a scRNA-seq T cell pseudotime trajectory: [evidence span withheld — anchor status: not_found]
- [readme] Multivariate distribution testing allows for the identification of pathways that show enrichment in a given population AND also pathways that show transcriptional change independent of enrichment.: [evidence span withheld — anchor status: external_doc]
