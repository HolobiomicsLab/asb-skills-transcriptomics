---
name: multisample-pathway-distribution-testing
description: Use when you have scRNA-seq expression matrices for three or more populations
  (time points, trajectory nodes, or conditions) and want to identify which pathway
  classes show the largest distribution change across all conditions simultaneously.
when_to_use_negative:
- When you only need pairwise enrichment/over-representation statistics for a single
  contrast rather than distribution change across multiple conditions
- When the input is already aggregated (e.g., bulk expression or a feature table)
  — SCPA requires single-cell-level distributions across populations
- When fold-change enrichment alone is the target statistic; SCPA explicitly treats
  fold change as secondary and may report high qval with low fold change
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SCPA
  role: Core package providing seurat_extract and compare_pathways for multisample
    pathway distribution testing
  repo: https://github.com/jackbibby1/SCPA
- name: Seurat
  role: Holds the single-cell dataset (e.g., szabo_t_cell.rds); SCPA auto-detects
    Seurat version and supports v5 layer system in seurat_extract and compare_seurat
- name: msigdbr
  role: Source of Hallmark pathway gene sets, formatted for SCPA with format_pathways
- name: dyno
  role: Infers pseudotime trajectory (ti_slingshot) and bins cells into milestones
    that become the multisample populations
- name: ComplexHeatmap
  role: Visualizes per-pathway qvals across pseudotime milestones as a heatmap
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1016/j.celrep.2022.111697
    title: Systematic single-cell pathway analysis to characterize early T cell activation
  source_content_hash: sha256:aca45749ffe295fb208502fdec2c54a8fc9ea5407b6559a89769897def0a1210
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jackbibby1/SCPA
  tools:
  - ComplexHeatmap
  - R
  - SCPA
  - Seurat
  - dyno
  - ggplot2
  - msigdbr
  - tidyverse
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# multisample-pathway-distribution-testing

## Summary

Apply SCPA's multisample compare_pathways to test changes in the multivariate distribution of pathway activity across >2 conditions (e.g., 0/12/24 h T cell activation time points or pseudotime milestones), ranking pathways by qval rather than fold change.

## When to use

Use when you have scRNA-seq expression matrices for three or more populations (time points, trajectory nodes, or conditions) and want to identify which pathway classes show the largest distribution change across all conditions simultaneously.

## When NOT to use

- When you only need pairwise enrichment/over-representation statistics for a single contrast rather than distribution change across multiple conditions
- When the input is already aggregated (e.g., bulk expression or a feature table) — SCPA requires single-cell-level distributions across populations
- When fold-change enrichment alone is the target statistic; SCPA explicitly treats fold change as secondary and may report high qval with low fold change

## Inputs

- Seurat object (e.g., szabo_t_cell.rds or naive_cd4.rds) or SingleCellExperiment object with metadata specifying population/condition and time point
- Pathway gene sets: msigdbr Hallmark output formatted with format_pathways, a gmt file, or a csv such as combined_metabolic_pathways.csv / h_k_r_go_pid_reg_wik.csv (pathway name in column 1, genes in subsequent columns)
- List of per-population normalized expression matrices (optionally downsampled, default 500 cells per population)

## Outputs

- Ranked data frame with Pathway, pval, adjPval, and qval columns per pathway across the multisample comparison
- CSV export (e.g., scpa_pathway_qvals.csv) and visualizations such as rank plots or heatmaps of qvals

## How to apply

Extract normalized expression matrices per population with SCPA's seurat_extract, subsetting a Seurat object (e.g., szabo_t_cell.rds) on metadata columns for cell identity and time point (0, 12, 24 h). Define gene sets with msigdbr Hallmark pathways formatted via format_pathways, or supply a gene set file such as combined_metabolic_pathways.csv or h_k_r_go_pid_reg_wik.csv (pathway name in column 1, genes in subsequent columns). Run compare_pathways(samples = list(pop1, pop2, pop3), pathways = pathways), optionally with parallel = TRUE and cores set, which downsamples to a default of 500 cells per population. Rank results by qval — SCPA's authors state the qval should be the primary statistic and fold change/enrichment only secondary, since pathways like arachidonic acid metabolism can show large qvals without fold change. Internally validate multisample results against a two-sample comparison (e.g., 0 h resting vs 12 h intermediate vs 24 h activated naive CD4 T cells), and write the ranked table (Pathway, pval, adjPval, qval) to CSV or visualize with plot_rank/plot_heatmap.

## Related tools

- **SCPA** (Core package providing seurat_extract and compare_pathways for multisample pathway distribution testing) — https://github.com/jackbibby1/SCPA
- **Seurat** (Holds the single-cell dataset (e.g., szabo_t_cell.rds); SCPA auto-detects Seurat version and supports v5 layer system in seurat_extract and compare_seurat)
- **msigdbr** (Source of Hallmark pathway gene sets, formatted for SCPA with format_pathways)
- **dyno** (Infers pseudotime trajectory (ti_slingshot) and bins cells into milestones that become the multisample populations)
- **ComplexHeatmap** (Visualizes per-pathway qvals across pseudotime milestones as a heatmap)

## Examples

```
scpa_out <- compare_pathways(samples = list(pop_0h, pop_12h, pop_24h), pathways = pathways, parallel = TRUE, cores = 4); write.csv(scpa_out, "scpa_pathway_qvals.csv")
```

## Evaluation signals

- Output table contains Pathway, pval, adjPval, and qval columns with one row per input pathway
- Largest qval pathways fall in interferon/cytokine-response and metabolic pathway classes for early T cell activation data, matching the paper's findings
- Multisample qval ranking is consistent with an independent two-sample comparison of 0 h resting vs 12 h/24 h activated cells (the paper's internal control)
- Each population input contains no more than the 500-cell downsampling default; results are stable under reruns with different seeds
- High-qval pathways with low fold change (e.g., arachidonic acid metabolism) are retained, confirming qval was used as the primary statistic

## Limitations

- compare_pathways downsamples to a default of 500 cells per population, which may affect sensitivity for rare populations
- Pathways can show large qvals without any fold change/enrichment, so interpreting results by fold change alone will miss biologically relevant pathways
- Parallel processing (parallel = TRUE) requires manually installing dependencies such as crossmatch 1.3.1 and multicross 2.1.0; installation errors may otherwise occur

## Evidence

- [readme] SCPA defines pathway activity as distribution change: [evidence span withheld — anchor status: external_doc]
- [readme] Multisample testing use case: [evidence span withheld — anchor status: external_doc]
