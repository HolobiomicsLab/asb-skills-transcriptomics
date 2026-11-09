---
name: single-cell-pathway-perturbation-analysis
description: Use when comparing pathway activity across multiple conditions, time
  points, or pseudotime phases in scRNA-seq data — especially when enrichment-based
  methods miss pathways with distributional shifts but little fold change.
when_to_use_negative:
- Input is already an aggregated feature/summary table rather than single-cell expression
  data with per-cell metadata
- The analysis goal is gene set over-representation (e.g. ORA on a gene list) rather
  than distributional comparison of pathway activity across cell populations
- Only two conditions are compared and a simple enrichment score would suffice — SCPA's
  multisample distribution testing is aimed at comparing pathway distributions across
  conditions
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SCPA
  role: Core R package providing seurat_extract, format_pathways, and compare_pathways
    for multisample pathway distribution testing
  repo: https://github.com/jackbibby1/SCPA
- name: Seurat
  role: Holds and subsets the scRNA-seq object; SCPA v1.6 auto-detects Seurat v5 layer
    system in seurat_extract and compare_seurat
- name: msigdbr
  role: Source of Hallmark pathway gene sets, formatted for SCPA via format_pathways
- name: tidyverse
  role: Data manipulation of extracted expression matrices and SCPA output tables
- name: ggplot2
  role: Visualization of SCPA results (rank plots, volcano plots, heatmaps)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.celrep.2022.111697
    title: Systematic single-cell pathway analysis to characterize early T cell activation
  source_content_hash: sha256:5b32ffb2635f5533784c54802e2d8e3f3470aaece028cd7ebc9af8ad749f349c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jackbibby1/SCPA
  tools:
  - R
  - SCPA
  - Seurat
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
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# single-cell-pathway-perturbation-analysis

## Summary

Detect pathway activity changes in single-cell RNA-seq by testing shifts in the multivariate distribution of pathway gene expression across conditions with SCPA, rather than enrichment scoring. Applied here to early T cell activation (0/12/24 h) using the Szabo CD4 T cell dataset and Hallmark/metabolic gene sets.

## When to use

Use when comparing pathway activity across multiple conditions, time points, or pseudotime phases in scRNA-seq data — especially when enrichment-based methods miss pathways with distributional shifts but little fold change.

## When NOT to use

- Input is already an aggregated feature/summary table rather than single-cell expression data with per-cell metadata
- The analysis goal is gene set over-representation (e.g. ORA on a gene list) rather than distributional comparison of pathway activity across cell populations
- Only two conditions are compared and a simple enrichment score would suffice — SCPA's multisample distribution testing is aimed at comparing pathway distributions across conditions

## Inputs

- Seurat object (e.g. szabo_t_cell.rds) or SingleCellExperiment object with metadata columns for cell identity and condition/time point
- Pathway gene sets: msigdbr Hallmark output formatted via format_pathways, or a csv like combined_metabolic_pathways.csv / h_k_r_go_pid_reg_wik.csv (pathway name in column 1, genes in subsequent columns)

## Outputs

- Ranked pathway table with Pathway, pval, adjPval, and qval columns (e.g. scpa_pathway_qvals.csv)
- Visualizations such as rank plots (plot_rank) and heatmaps (plot_heatmap) of qval results

## How to apply

Load the Seurat object (e.g. szabo_t_cell.rds) with SCPA, Seurat, tidyverse, and msigdbr, then extract normalized expression matrices per population and time point using seurat_extract, subsetting on metadata columns for cell identity and the 0, 12, and 24 h time points. Define gene sets either by pulling Hallmark pathways with msigdbr and formatting them with format_pathways, or by supplying a csv file such as combined_metabolic_pathways.csv (pathway name in column 1, genes in subsequent columns). Run compare_pathways(samples = list(pop1, pop2, pop3), pathways = pathways), optionally with parallel = TRUE and cores set for speed; the function downsamples to a default of 500 cells per population. Rank results by qval, which should be used as the primary statistic for biological relevance, with fold change/enrichment only as a secondary informative value — pathways like arachidonic acid metabolism can show large qvals without fold change. Write the ranked table (Pathway, pval, adjPval, qval) to csv and visualize with plot_rank or plot_heatmap.

## Related tools

- **SCPA** (Core R package providing seurat_extract, format_pathways, and compare_pathways for multisample pathway distribution testing) — https://github.com/jackbibby1/SCPA
- **Seurat** (Holds and subsets the scRNA-seq object; SCPA v1.6 auto-detects Seurat v5 layer system in seurat_extract and compare_seurat)
- **msigdbr** (Source of Hallmark pathway gene sets, formatted for SCPA via format_pathways)
- **tidyverse** (Data manipulation of extracted expression matrices and SCPA output tables)
- **ggplot2** (Visualization of SCPA results (rank plots, volcano plots, heatmaps))

## Examples

```
scpa_out <- compare_pathways(samples = list(pop_0h, pop_12h, pop_24h), pathways = pathways, parallel = TRUE, cores = 4)
```

## Evaluation signals

- Output table contains exactly the columns Pathway, pval, adjPval, qval for every tested pathway
- Ranking by qval (not fold change) places interferon-response and metabolic pathway classes at the top for the 0/12/24 h T cell activation comparison
- Arachidonic acid metabolism shows a large qval despite low fold change — a known pattern that validates using qval as primary statistic
- Default downsampling to 500 cells per population was applied (or explicitly changed) and did not error on any of the three time-point subsets
- Installing SCPA requires crossmatch 1.3.1 and multicross 2.1.0 dependencies; missing versions indicate a broken environment

## Limitations

- Pathways can show large qvals without fold change/enrichment, so fold change alone is misleading — qval must be the primary statistic
- compare_pathways downsamples to a default of 500 cells per population, which may affect results with very small populations
- Installation often requires manually installing dependencies (e.g. crossmatch and multicross versions) as documented in GitHub issues 54 and 56

## Evidence

- [readme] multisample testing: [evidence span withheld — anchor status: external_doc]
- [readme] installation deps: [evidence span withheld — anchor status: external_doc]
