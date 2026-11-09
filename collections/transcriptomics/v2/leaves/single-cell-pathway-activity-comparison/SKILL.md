---
name: single-cell-pathway-activity-comparison
description: Use when you have scRNA-seq expression matrices from two populations
  (e.g. healthy vs disease, blood vs tissue) and need to rank pathway perturbations
  by distributional change rather than enrichment.
when_to_use_negative:
- The question is differential gene expression or gene-set over-representation only,
  with no interest in distributional change of pathway gene expression
- Input is bulk RNA-seq or already-aggregated feature tables rather than single-cell-level
  expression with population metadata
- Only one population/sample is available, since SCPA compares pathway distributions
  across at least two conditions
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: SCPA
  role: 'Core package: seurat_extract to subset expression by metadata, compare_pathways/compare_seurat
    for multivariate distribution testing of pathways'
  repo: https://github.com/jackbibby1/SCPA
- name: Seurat
  role: Loads and splits scRNA-seq objects by condition (SplitObject); v5 layer compatibility
    auto-detected by SCPA
- name: msigdbr
  role: Sources HALLMARK, KEGG, Reactome, PID, Wiki, BioCarta gene sets formatted
    via format_pathways()
- name: ggplot2
  role: Ranks pathways by qval variance across tissue comparisons and renders supporting
    plots
- name: ComplexHeatmap
  role: Renders heatmaps of qvals per cell type for viral-pathway sensitivity benchmarking
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1016/j.celrep.2022.111697
    title: Systematic single-cell pathway analysis to characterize early T cell activation
  source_content_hash: sha256:323f4f7272572c99c15464bd2a5e1af4b7585103c7644714b0f55d1fb901949e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jackbibby1/SCPA
  tools:
  - ComplexHeatmap
  - R
  - SCPA
  - Seurat
  - dplyr
  - ggplot2
  - msigdbr
  - tidyverse
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# single-cell-pathway-activity-comparison

## Summary

Compare pathway activity between two or more single-cell populations using SCPA, which defines pathway activity as a change in the multivariate distribution of a pathway's genes (qval) rather than gene-set enrichment. This enables detection of perturbed pathways such as interferon/complement responses in COVID-19 or tissue-specific alpha-defensin expression in bone-marrow T cells.

## When to use

Use when you have scRNA-seq expression matrices from two populations (e.g. healthy vs disease, blood vs tissue) and need to rank pathway perturbations by distributional change rather than enrichment.

## When NOT to use

- The question is differential gene expression or gene-set over-representation only, with no interest in distributional change of pathway gene expression
- Input is bulk RNA-seq or already-aggregated feature tables rather than single-cell-level expression with population metadata
- Only one population/sample is available, since SCPA compares pathway distributions across at least two conditions

## Inputs

- Seurat object (.rds) or SingleCellExperiment object with metadata columns for cell type and condition (e.g. cell.type.fine, Status, tissue, stimulation)
- Pathway gene set file (e.g. h_k_r_go_pid_reg_wik.csv, combined_metabolic_pathways.csv, or gmt files)
- msigdbr Homo sapiens output for building custom gene sets

## Outputs

- SCPA result table with Pathway, fold change/enrichment, and qval columns per population comparison
- Condition-by-pathway qval matrix (e.g. viral_pathway_sensitivity.csv) filtered to qval > 2
- Ranked variance or qval plots (ggplot2, ComplexHeatmap, plot_rank, plot_heatmap)

## How to apply

Load the Seurat object (e.g. covid_blood_atlas.rds or szabo_t_cell.rds) with SCPA, Seurat, and tidyverse, then split it by condition using SplitObject and extract per-cell-type expression matrices with seurat_extract on a metadata column such as cell.type.fine. Define pathway gene sets either from a combined MSigDB file like h_k_r_go_pid_reg_wik.csv (canonical, GO, regulatory pathways) or by filtering msigdbr Homo sapiens output with format_pathways() (~3,000 KEGG/Reactome/BioCarta/Wiki/PID plus HALLMARK sets). For each population pair, run compare_pathways(list(pop1, pop2), pathways), keep the Pathway and qval columns, and join results into a condition-by-pathway qval table. Use qval as the primary statistic for biological relevance (filter to qval > 2 for highly perturbed pathways), treating fold change/enrichment as a secondary, purely informative value; pathways can show large qvals without fold change, as SCPA highlights for arachidonic acid metabolism. Visualize with plot_rank, plot_heatmap, or custom ggplot2/ComplexHeatmap plots ranking conditions by the number of high-qval pathways.

## Related tools

- **SCPA** (Core package: seurat_extract to subset expression by metadata, compare_pathways/compare_seurat for multivariate distribution testing of pathways) — https://github.com/jackbibby1/SCPA
- **Seurat** (Loads and splits scRNA-seq objects by condition (SplitObject); v5 layer compatibility auto-detected by SCPA)
- **msigdbr** (Sources HALLMARK, KEGG, Reactome, PID, Wiki, BioCarta gene sets formatted via format_pathways())
- **ggplot2** (Ranks pathways by qval variance across tissue comparisons and renders supporting plots)
- **ComplexHeatmap** (Renders heatmaps of qvals per cell type for viral-pathway sensitivity benchmarking)

## Examples

```
library(SCPA); covid <- SplitObject(readRDS("covid_blood_atlas.rds"), split.by = "Status"); scpa_out <- compare_pathways(list(seurat_extract(covid[["healthy"]], meta1="cell.type.fine", value_meta1="CD14+ Monocytes"), seurat_extract(covid[["covid"]], meta1="cell.type.fine", value_meta1="CD14+ Monocytes")), get_paths("h_k_r_go_pid_reg_wik.csv"))
```

## Evaluation signals

- Output table contains Pathway and qval fields with ranked values per comparison
- Highly perturbed conditions rank at top: CD14+ monocytes show large interferon/complement qvals in COVID-19 peripheral blood
- Tissue analysis: antimicrobial peptide production (DEFA1/DEFA3) and prostaglandin synthesis appear among the most variable pathways, with DEFA1/DEFA3 expression restricted to bone-marrow T cells in VlnPlots
- Pathways with qval > 2 in any comparison are retained; large-qval/low-fold-change pathways (e.g. arachidonic acid metabolism) are kept, not discarded
- Installation completes with pinned dependencies crossmatch 1.3.1 and multicross 2.1.0 without errors

## Limitations

- qval and fold change are complementary: relying only on fold change/enrichment misses pathways with distributional change but no enrichment
- Installation errors may require manually installing dependencies (crossmatch, multicross); see GitHub issues #54 and #56
- SCPA automatically detects Seurat object version; older workflows may break with Seurat v5 layer system unless SCPA >= v1.6 is used

## Evidence

- [readme] README: distribution over enrichment: [evidence span withheld — anchor status: external_doc]
