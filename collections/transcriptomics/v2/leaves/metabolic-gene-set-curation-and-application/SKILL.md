---
name: metabolic-gene-set-curation-and-application
description: Use when analyzing metabolic pathway activity in single-cell RNA-seq
  data — e.g., across T cell activation time points or pseudotime trajectory nodes
  — where combined metabolic gene sets and distribution-based qvals are needed rather
  than simple enrichment.
when_to_use_negative:
- When the question is gene-level differential expression rather than pathway-level
  distribution change across populations
- When the input is already a scored pathway activity matrix (e.g., from AUCell/GSVA)
  rather than single-cell expression data with population metadata
- When judging results solely by fold change/enrichment — SCPA pathways can show large
  qvals without enrichment, so enrichment-only workflows will misinterpret results
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3407
tools:
- name: SCPA
  role: 'Core package: seurat_extract to subset expression matrices by metadata, compare_pathways
    for multisample distribution testing against metabolic gene sets, and plot_rank/plot_heatmap
    for visualization'
  repo: https://github.com/jackbibby1/SCPA
- name: Seurat
  role: Load and manipulate the Seurat object (e.g., naive_cd4.rds), subset populations
    (removing Treg cells), and provide the object passed to seurat_extract/compare_seurat
  repo: https://github.com/satijalab/seurat
- name: dyno
  role: Infer the pseudotime trajectory with ti_slingshot and bin cells into trajectory
    milestones with group_onto_nearest_milestones to define multisample populations
- name: msigdbr
  role: Retrieve MSigDB pathway gene sets (e.g., Hallmark) that are formatted with
    format_pathways for use as compare_pathways input
- name: ComplexHeatmap
  role: Plot the qval matrix of pathway activity across pseudotime milestones
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.celrep.2022.111697
    title: Systematic single-cell pathway analysis to characterize early T cell activation
  source_content_hash: sha256:c9f8cf73e64df13d676bd3604cd0692ff0dfd16e17776e8d31f0871125961649
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jackbibby1/SCPA
  tools:
  - ComplexHeatmap
  - SCPA
  - Seurat
  - dyno
  - msigdbr
  - tidyverse
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# metabolic-gene-set-curation-and-application

## Summary

Curate and apply metabolic pathway gene sets (Hallmark/KEGG/Reactome from MSigDB) with SCPA's compare_pathways to quantify metabolic reprogramming across single-cell populations or pseudotime milestones, using the qval as the primary statistic and fold change/enrichment only as secondary evidence.

## When to use

Use when analyzing metabolic pathway activity in single-cell RNA-seq data — e.g., across T cell activation time points or pseudotime trajectory nodes — where combined metabolic gene sets and distribution-based qvals are needed rather than simple enrichment.

## When NOT to use

- When the question is gene-level differential expression rather than pathway-level distribution change across populations
- When the input is already a scored pathway activity matrix (e.g., from AUCell/GSVA) rather than single-cell expression data with population metadata
- When judging results solely by fold change/enrichment — SCPA pathways can show large qvals without enrichment, so enrichment-only workflows will misinterpret results

## Inputs

- Seurat object (.rds, e.g., naive_cd4.rds) or SingleCellExperiment object, or manually subsetted expression matrices
- Metabolic pathway gene sets: combined_metabolic_pathways.csv (Hallmark/KEGG/Reactome) or MSigDB sets retrieved via msigdbr
- Metadata column defining populations (e.g., 'milestone' from group_onto_nearest_milestones, or condition/time point)

## Outputs

- Per-pathway statistics table from compare_pathways with qval and fold change/enrichment for each gene set
- Ranked pathway plots (plot_rank) highlighting top pathways such as glycolysis
- Heatmaps of qvals across populations or pseudotime milestones (plot_heatmap or ComplexHeatmap)

## How to apply

Load the Seurat object (e.g., naive_cd4.rds) and, for trajectory work, remove unwanted populations (e.g., Treg cells), select the top 1000 most variable genes, and infer pseudotime with dyno's infer_trajectory using ti_slingshot, binning cells into nodes with group_onto_nearest_milestones. Extract per-population expression matrices by looping SCPA's seurat_extract over the metadata column of interest (e.g., 'milestone'). Provide curated metabolic gene sets — combined_metabolic_pathways.csv or Hallmark sets via msigdbr with format_pathways — as the pathway input. Run compare_pathways on the list of expression samples against the gene sets to obtain per-pathway qvals and fold change/enrichment. Interpret results with the qval as the primary statistic of biological relevance and fold change/enrichment as secondary, since pathways like arachidonic acid metabolism can show large qvals without overall enrichment; visualize with plot_rank, plot_heatmap, or ComplexHeatmap.

## Related tools

- **SCPA** (Core package: seurat_extract to subset expression matrices by metadata, compare_pathways for multisample distribution testing against metabolic gene sets, and plot_rank/plot_heatmap for visualization) — https://github.com/jackbibby1/SCPA
- **Seurat** (Load and manipulate the Seurat object (e.g., naive_cd4.rds), subset populations (removing Treg cells), and provide the object passed to seurat_extract/compare_seurat) — https://github.com/satijalab/seurat
- **dyno** (Infer the pseudotime trajectory with ti_slingshot and bin cells into trajectory milestones with group_onto_nearest_milestones to define multisample populations)
- **msigdbr** (Retrieve MSigDB pathway gene sets (e.g., Hallmark) that are formatted with format_pathways for use as compare_pathways input)
- **ComplexHeatmap** (Plot the qval matrix of pathway activity across pseudotime milestones)

## Examples

```
library(SCPA); pathways <- get_paths("combined_metabolic_pathways.csv"); scpa_out <- compare_pathways(samples, pathways)
```

## Evaluation signals

- compare_pathways output contains a qval for every input pathway in every sample/population, with no missing rows for the metabolic gene sets supplied
- Top-ranked pathways (e.g., glycolysis in activated naive CD4 T cells) are consistent between the qval ranking and the plot_rank figure
- qvals from a multisample pseudotime analysis can be internally validated by correlating with a classical two-sample compare_pathways run (e.g., 0 h resting vs 12 h intermediate and 24 h activated naive CD4 T cells)
- Interpretation treats qval as the primary statistic; pathways with high qval but low fold change (e.g., arachidonic acid metabolism) are retained as biologically relevant
- Heatmap of qvals across milestones shows coherent ordering along the inferred slingshot trajectory

## Limitations

- SCPA defines pathway activity as a change in multivariate distribution, so high-qval pathways may lack fold change/enrichment; judging by enrichment alone will miss these
- Installation can require manually installing crossmatch (v1.3.1) and multicross (v2.1.0) dependencies if devtools::install_github errors
- Large pathway/gene set inputs can be memory-intensive in older versions; versions ≥1.3.0 improved memory efficiency and ≥1.5.0 added parallel processing

## Evidence

- [methods] qval primary statistic: "we suggest that the qval should be used as the primary statistic when judging biological relevance, and the pathway fold change/enrichment should only be a secondary informative value"
- [methods] pseudotime workflow: "And finally running the `infer_trajectory` function using slingshot as the modeller"
- [methods] seurat_extract usage: "we can use the `seurat_extract` function from the SCPA package, which takes a Seurat object and subsets the data based on the metadata"
- [readme] multisample capability: [evidence span withheld — anchor status: external_doc]
- [methods] qval without enrichment: "we highlight arachidonic acid as one particular pathway that shows this pattern"
