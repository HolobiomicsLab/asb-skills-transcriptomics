---
name: multipanel-figure-generation-from-module-scores
description: Use when you have run JOINTLY and scored cells with AddModuleScore_UCell
  and need to visualize factor activity per cell type or per covariate (e.g. factor
  10 insulin-signalling boxplots by status and sex), or build module/marker AUC comparison
  panels.
when_to_use_negative:
- No interpretable modules have been learned yet — JOINTLY must be run first (e.g.
  jointly(obj, batch.var = "batch_label")) before scores exist to plot
- Expression values are batch-smoothed/imputed from the JOINTLY H and F matrices —
  the authors advise these are for visualization with caution only, not for quantitative
  score comparisons
- The question is which integration method to choose — integration metrics such as
  ARI/NMI do not reveal the genes or features driving clustering, which is what this
  visualization skill targets
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: JOINTLY
  role: Learns the interpretable gene modules (factors) whose UCell scores are visualized
  repo: https://github.com/madsen-lab/rJOINTLY
- name: Seurat
  role: Runs UMAP on the JOINTLY reduction and renders DimPlot/FeaturePlot panels
    from module scores
  repo: https://github.com/satijalab/seurat
- name: UCell
  role: Scores every JOINTLY-derived gene module per cell with AddModuleScore_UCell
    as input to the figure panels
  repo: https://github.com/carmonalab/UCell
- name: presto
  role: wilcoxauc computes per-cell-type AUC for module genes versus marker genes
    used in the heatmap/comparison panel
  repo: https://github.com/immunogenomics/presto
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-44279-8
    title: 'JOINTLY: interpretable joint clustering of single-cell transcriptomes'
  source_content_hash: sha256:0a418bde212f13450846556dbfba2252fe0861faed424d1656742c5acd8d019a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madsen-lab/rJOINTLY
  tools:
  - JOINTLY
  - Seurat
  - UCell
  - aricode
  - presto
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Multipanel figure generation from module scores

## Summary

Generates publication figures (module-by-cell-type heatmaps, UMAP feature plots, and boxplots of JOINTLY factor module scores split by covariates such as insulin-signalling status and sex) from JOINTLY's interpretable gene modules and UCell scores, as done for Fig. 4A-H of the adipose analysis.

## When to use

Use when you have run JOINTLY and scored cells with AddModuleScore_UCell and need to visualize factor activity per cell type or per covariate (e.g. factor 10 insulin-signalling boxplots by status and sex), or build module/marker AUC comparison panels.

## When NOT to use

- No interpretable modules have been learned yet — JOINTLY must be run first (e.g. jointly(obj, batch.var = "batch_label")) before scores exist to plot
- Expression values are batch-smoothed/imputed from the JOINTLY H and F matrices — the authors advise these are for visualization with caution only, not for quantitative score comparisons
- The question is which integration method to choose — integration metrics such as ARI/NMI do not reveal the genes or features driving clustering, which is what this visualization skill targets

## Inputs

- Seurat object with a JOINTLY reduction (H matrix) and cell-type labels
- List of JOINTLY interpretable gene modules (liver$modules equivalent, one list per factor)
- Per-cell metadata grouping variables (e.g. insulin-signalling status, sex, donor/batch_label)
- Marker-gene table for the AUC comparison (filtered by logFC >= log2(1.1), padj <= 0.05, auc >= 0.6)

## Outputs

- Per-cell UCell module scores for each JOINTLY factor (columns '<factor>_UCell')
- Module-by-cell-type heatmap panel
- Boxplots of factor 10 (insulin-signalling) module scores split by insulin-signalling status and sex (Fig. 4A-H equivalent)
- Module-gene vs marker-gene AUC comparison table from wilcoxauc
- Combined result file (annotated Seurat object, module scores, AUC table, figure panels)

## How to apply

First compute per-cell activity of every JOINTLY-derived gene module with AddModuleScore_UCell(features = liver$modules), producing columns named '<factor>_UCell'. Project the joint co-embedding with RunUMAP on the JOINTLY reduction (dims 1:15) so module activity can be overlaid via FeaturePlot (e.g. 'factor_1_UCell'). For the cell-type panel, build the module-by-cell-type heatmap and compute, per cell type, the average AUC of module genes versus filtered marker genes (logFC >= log2(1.1), padj <= 0.05, auc >= 0.6) with wilcoxauc from presto. For the covariate panel, generate boxplots of the factor 10 (insulin-signalling) module scores split by insulin-signalling status and sex to reproduce Fig. 4A-H. Save the annotated Seurat object, module scores, AUC comparison table and figure panels together (e.g. a single RDS) for reproducibility, mirroring madsen-lab/JOINTLY_reproducibility scripts.

## Related tools

- **JOINTLY** (Learns the interpretable gene modules (factors) whose UCell scores are visualized) — https://github.com/madsen-lab/rJOINTLY
- **Seurat** (Runs UMAP on the JOINTLY reduction and renders DimPlot/FeaturePlot panels from module scores) — https://github.com/satijalab/seurat
- **UCell** (Scores every JOINTLY-derived gene module per cell with AddModuleScore_UCell as input to the figure panels) — https://github.com/carmonalab/UCell
- **presto** (wilcoxauc computes per-cell-type AUC for module genes versus marker genes used in the heatmap/comparison panel) — https://github.com/immunogenomics/presto

## Examples

```
liver$object <- AddModuleScore_UCell(liver$object, features = liver$modules); FeaturePlot(liver$object, features = "factor_1_UCell")
```

## Evaluation signals

- Module-score columns follow the UCell naming pattern '<factor>_UCell' (e.g. factor_1_UCell) after AddModuleScore_UCell
- The boxplot panel reproduces the Fig. 4A-H grouping: factor 10 (insulin-signalling) scores split by insulin-signalling status and sex
- Module genes that are also marker genes show a significantly higher average AUC than marker genes not in the most enriched module, per cell type
- The UMAP overlay (FeaturePlot) shows module score gradients consistent with the JOINTLY reduction (dims 1:15) rather than batch structure
- Outputs match the reproducibility artefacts of madsen-lab/JOINTLY_reproducibility: annotated Seurat object, module scores, AUC table and figure panels saved together

## Limitations

- JOINTLY is stochastic — different seeds give different solutions and sometimes it is worth running up to 5 times and choosing the best solution, so figure panels depend on the retained solution
- Batch-aware smoothened expression derived from the H and F matrices should be used with caution and only for visualization purposes
- Cluster/cell-type assignment depends on low-resolution clustering (FindClusters res = 0.05 on dims 1:15); misassigned labels propagate into heatmap and boxplot panels

## Evidence

- [other] Generate boxplots of the factor 10 (insulin-signalling) module scores split by insulin-signalling status and sex to reproduce Fig. 4A-H.: [evidence span withheld — anchor status: not_found]
- [methods] UCell module scoring step: "liver$object <- AddModuleScore_UCell(liver$object, features = liver$modules)"
- [readme] README module score visualization: "## Visualize module scores on the UMAP
FeaturePlot(liver$object, features = "factor_1_UCell")"
- [other] marker AUC comparison: [evidence span withheld — anchor status: not_found]
- [readme] seed sensitivity caveat: "Setting a seed and re-running produces a different result. Sometimes it is worth running JOINTLY up to 5 times and choosing the best solution."
