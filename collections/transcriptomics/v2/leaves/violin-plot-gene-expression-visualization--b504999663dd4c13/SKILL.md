---
name: violin-plot-gene-expression-visualization
description: Use when you need to visually confirm that specific genes (e.g., DEFA1/DEFA3)
  are expressed exclusively in one subset of cells grouped by a metadata variable
  such as tissue, after a pathway or differential analysis flags them.
when_to_use_negative:
- The input is already aggregated pseudobulk or a feature table rather than single-cell
  level expression, so per-cell distribution violins are not meaningful.
- You need statistical testing of expression differences rather than visual confirmation
  — use SCPA compare_pathways qval-based analysis instead.
- The gene set of interest spans pathway-level activity rather than individual marker
  genes; violin plots of single genes cannot represent multivariate pathway distribution
  shifts.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: Seurat
  role: Provides DimPlot and VlnPlot to visualize the Seurat object split by tissue/stimulation
    and to plot gene expression grouped by tissue
  repo: https://github.com/jackbibby1/SCPA
- name: ggplot2
  role: Underlying graphics engine for violin plots and for styling summary figures
    (e.g., variance-rank plot with royalblue2 points and blank panel background)
- name: tidyverse
  role: Loaded alongside Seurat for data manipulation supporting the visualization
    workflow
- name: SCPA
  role: Upstream compare_pathways qval and variance analysis that nominates which
    genes/pathways to visualize with violins
  repo: https://github.com/jackbibby1/SCPA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.celrep.2022.111697
    title: Systematic single-cell pathway analysis to characterize early T cell activation
  source_content_hash: sha256:0380de91b66c99f2b354a84a89a7fa3cb35b613b589e38a8999c3237abecb163
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jackbibby1/SCPA
  tools:
  - R
  - SCPA
  - Seurat
  - dplyr
  - ggplot2
  - tidyverse
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# violin-plot-gene-expression-visualization

## Summary

Visualize single-cell gene expression distributions with Seurat/ggplot2 violin plots grouped by metadata (e.g., tissue, stimulation) to localize marker expression to specific populations — as used to show DEFA1/DEFA3 alpha-defensin expression restricted to bone-marrow-derived T cells in the Szabo T cell dataset.

## When to use

Use when you need to visually confirm that specific genes (e.g., DEFA1/DEFA3) are expressed exclusively in one subset of cells grouped by a metadata variable such as tissue, after a pathway or differential analysis flags them.

## When NOT to use

- The input is already aggregated pseudobulk or a feature table rather than single-cell level expression, so per-cell distribution violins are not meaningful.
- You need statistical testing of expression differences rather than visual confirmation — use SCPA compare_pathways qval-based analysis instead.
- The gene set of interest spans pathway-level activity rather than individual marker genes; violin plots of single genes cannot represent multivariate pathway distribution shifts.

## Inputs

- Seurat object (RDS file, e.g., szabo_t_cell.rds) with cell metadata columns such as 'tissue', 'stimulation', and 'fine' cell type labels
- Candidate gene names from upstream pathway/differential analysis (e.g., DEFA1, DEFA3)
- Ranked pathway results (qval tables joined across tissue comparisons) to justify gene selection

## Outputs

- Violin plots of gene expression grouped by metadata (e.g., DEFA1/DEFA3 grouped by tissue)
- Accompanying ggplot2 summary figures (e.g., tissue_variance.png showing variance-ranked pathways)
- Visual validation that marker expression is exclusive to a target group (bone marrow T cells)

## How to apply

Load the Seurat object (e.g., szabo_t_cell.rds) with Seurat and tidyverse, and first DimPlot it split by metadata such as tissue/stimulation to orient the reader. Identify candidate genes from upstream analysis — here, SCPA compare_pathways qval rankings across tissue comparisons (blood vs bm, ln, lung) plus per-pathway variance via apply(all_data, 1, var) surfaced antimicrobial peptide production genes DEFA1 and DEFA3. Then call VlnPlot on the object with the feature(s) of interest and group.by set to the metadata column of interest (tissue), letting ggplot2 render the distribution per group. Judge success by whether expression violins are visibly non-zero only in the expected group (bone marrow) and near-zero elsewhere; validation for DEFA1/DEFA3 was expression restricted to bone-marrow-derived T cells. Rank/summary plots such as the tissue_variance.png ggplot2 figure (royalblue2 points, blank panel background) should accompany the violins to link the genes to their variable pathways.

## Related tools

- **Seurat** (Provides DimPlot and VlnPlot to visualize the Seurat object split by tissue/stimulation and to plot gene expression grouped by tissue) — https://github.com/jackbibby1/SCPA
- **ggplot2** (Underlying graphics engine for violin plots and for styling summary figures (e.g., variance-rank plot with royalblue2 points and blank panel background))
- **tidyverse** (Loaded alongside Seurat for data manipulation supporting the visualization workflow)
- **SCPA** (Upstream compare_pathways qval and variance analysis that nominates which genes/pathways to visualize with violins) — https://github.com/jackbibby1/SCPA

## Examples

```
library(Seurat); seu <- readRDS("szabo_t_cell.rds"); VlnPlot(seu, features = c("DEFA1", "DEFA3"), group.by = "tissue")
```

## Evaluation signals

- DEFA1/DEFA3 violin plots show expression restricted to bone-marrow-derived T cells, with near-zero expression in blood, lymph node, and lung groups.
- The variance-ranked ggplot2 figure places antimicrobial peptide production and prostaglandin synthesis among the top variable pathways and is saved as tissue_variance.png.
- DimPlots split by tissue/stimulation render without errors, confirming the Seurat object's metadata columns ('fine', 'stimulation', tissue) are correctly populated.
- Only Pathway and qval columns from compare_pathways output feed the upstream ranking, so visualized genes trace back to a defensible statistical ordering.
- Plots use the specified styling (e.g., royalblue2 points, blank panel background) and reproduce in a fresh R session from the loaded RDS object.

## Limitations

- Violin plots are qualitative visual evidence; the article advises that qval should be the primary statistic when judging biological relevance, with fold change/enrichment secondary.
- The demonstrated validation applies to the Szabo T cell dataset's tissue metadata; other datasets require appropriate group.by variables and may not show the same exclusivity pattern.
- SCPA pathways can show large qvals without fold change/enrichment (e.g., arachidonic acid metabolism), so individual-gene violins may not fully represent pathway-level changes.

## Evidence

- [methods] library(Seurat)
library(dplyr)
library(ggplot2): "library(Seurat)
library(dplyr)
library(ggplot2)"
