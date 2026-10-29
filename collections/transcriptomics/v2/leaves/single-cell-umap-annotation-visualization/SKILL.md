---
name: single-cell-umap-annotation-visualization
description: Use when an integrated scRNA-seq object (e.g. Human_HSPC normalized UMI
  matrix) needs cell-type annotation via AUCell enrichment of lineage signature GMT
  sets, with results rendered as a UMAP colored by assigned lineage labels.
when_to_use_negative:
- When no curated signature gene sets exist for the biological system of interest
  — the user would have to compile and curate customized gene sets, since only 75
  hematopoiesis-relevant sets are provided
- When annotation should come from reference-free marker analysis (identifyDifferentialGenes)
  rather than signature-set enrichment scoring
- When the object lacks a normalized UMI matrix or a UMAP embedding, since scoring
  and visualization depend on both
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SingCellaR
  role: R package hosting the AUCell scoring, annotation, and UMAP visualization functions
    (get_gmtGeneSets, Build_AUCell_Rankings, Run_AUCell, get_cells_annotation, plot_cells_annotation)
  repo: https://github.com/supatt-lab/SingCellaR
- name: AUCell
  role: Enrichment-based method underlying Build_AUCell_Rankings and Run_AUCell; rankings
    can hit running time and memory issues on large-scale datasets
- name: R
  role: Environment in which the SingCellaR/AUCell annotation workflow is executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:51cdd94e4e3f5db6f91e90cec06412ee5c6538483c1be1be540b4ccb75c32df4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/supatt-lab/SingCellaR
  tools:
  - AUCell
  - R
  - SingCellaR
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# single-cell-umap-annotation-visualization

## Summary

Assigns cell-type labels to individual cells by applying gene-set-specific AUCell score cutoffs to lineage-signature enrichment scores in SingCellaR, then visualizes the annotations on the UMAP embedding (Figure 7A equivalent). It grounds per-cell identity calls in curated signature gene sets rather than cluster markers alone.

## When to use

Use when an integrated scRNA-seq object (e.g. Human_HSPC normalized UMI matrix) needs cell-type annotation via AUCell enrichment of lineage signature GMT sets, with results rendered as a UMAP colored by assigned lineage labels.

## When NOT to use

- When no curated signature gene sets exist for the biological system of interest — the user would have to compile and curate customized gene sets, since only 75 hematopoiesis-relevant sets are provided
- When annotation should come from reference-free marker analysis (identifyDifferentialGenes) rather than signature-set enrichment scoring
- When the object lacks a normalized UMI matrix or a UMAP embedding, since scoring and visualization depend on both

## Inputs

- Integrated SingCellaR object containing the normalized UMI matrix (e.g. Human_HSPC, accessed via get_normalized_umi)
- GMT file of lineage signature gene sets (75 pre-curated hematopoiesis sets in the protocol)
- AUCell rankings file (built and cached via Build_AUCell_Rankings)

## Outputs

- Per-cell AUCell enrichment (AUC) score table for all gene sets
- Per-cell cell-type annotations retrievable via get_cells_annotation(object)
- UMAP plot of assigned cell types (Figure 7A equivalent, saved as aucell_celltype_umap.png)

## How to apply

Load the integrated object with its normalized UMI matrix (via get_normalized_umi) into SingCellaR in R, and read lineage gene sets from a GMT file with get_gmtGeneSets(gmt.file), confirming the sets with listAvailableGeneSets(gmt.file); the protocol provides 75 pre-curated hematopoiesis gene sets. Build AUCell rankings with Build_AUCell_Rankings(object, AUCell_buildRankings.file, nCores, plotStats, splitByBlocks), caching to the specified rankings file, and use Build_AUCell_Rankings_Fast with block.cell.size for large-scale data to avoid running time and memory issues. Compute per-cell enrichment scores with Run_AUCell(object, AUCell_buildRankings.file, geneSets.gmt.file, n.random.genes), then apply gene-set-specific AUC score cutoffs to assign cell-type labels retrievable via get_cells_annotation(object). Finally render the annotation on the UMAP with plot_cells_annotation(object, type) and export the plot as aucell_celltype_umap.png.

## Related tools

- **SingCellaR** (R package hosting the AUCell scoring, annotation, and UMAP visualization functions (get_gmtGeneSets, Build_AUCell_Rankings, Run_AUCell, get_cells_annotation, plot_cells_annotation)) — https://github.com/supatt-lab/SingCellaR
- **AUCell** (Enrichment-based method underlying Build_AUCell_Rankings and Run_AUCell; rankings can hit running time and memory issues on large-scale datasets)
- **R** (Environment in which the SingCellaR/AUCell annotation workflow is executed)

## Examples

```
Run_AUCell(object, AUCell_buildRankings.file, geneSets.gmt.file, n.random.genes)
```

## Evaluation signals

- Rankings file and AUCell score table exist and contain all 75 curated gene sets
- UMAP plot renders cells colored by the assigned lineage annotations, matching the Figure 7A layout
- The HSC/MPP gene signature is uniformly enriched across the majority of cells in cluster 1 of the Human_HSPC dataset
- get_cells_annotation(object) returns labels for every cell, each derived from a gene-set-specific AUC cutoff
- Cached AUCell rankings file can be reloaded and reused by Run_AUCell without rebuilding

## Limitations

- AUCell_buildRankings has running time and memory issues on large-scale datasets; use Build_AUCell_Rankings_Fast with block.cell.size
- SingCellaR requires signature gene sets to perform cell type annotation analysis; users must compile and curate customized gene sets for systems outside hematopoiesis
- Only 75 gene sets curated from previous hematopoiesis studies are provided with the protocol
- UMAP plot rotations may differ from the protocol figures due to software version and seed number differences

## Evidence

- [full_text] Read the lineage signature gene sets from the GMT file using get_gmtGeneSets(gmt.file) and confirm the expected sets with listAvailableGeneSets(gmt.file): [evidence span withheld — anchor status: not_found]
- [full_text] use Build_AUCell_Rankings_Fast with block.cell.size for large-scale data to avoid running time and memory issues: [evidence span withheld — anchor status: not_found]
- [discussion] running time and memory issues when performing the AUCell_buildRankings function for a large-scale dataset: "running time and memory issues when performing the AUCell_buildRankings function for a large-scale dataset"
- [discussion] SingCellaR requires signature gene sets to perform the cell type annotation analysis.: "SingCellaR requires signature gene sets to perform the cell type annotation analysis."
- [readme] The tutorial of how to use the software can be found at https://supatt-lab.github.io/SingCellaR.Doc/: "The tutorial of how to use the software can be found at https://supatt-lab.github.io/SingCellaR.Doc/"
