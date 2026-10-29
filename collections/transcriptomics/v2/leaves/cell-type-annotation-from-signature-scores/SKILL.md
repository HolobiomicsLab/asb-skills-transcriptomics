---
name: cell-type-annotation-from-signature-scores
description: Use when a normalized scRNA-seq object (e.g., SingCellaR Human_HSPC)
  needs cell-type labels from lineage signature gene sets rather than marker-gene
  inspection or reference mapping.
when_to_use_negative:
- When only 10x Cell Ranger raw outputs are available — the object must already contain
  normalized UMIs after normalize_UMIs
- When the biological system lacks relevant curated signatures and customized gene
  sets would have to be compiled from scratch
- When reference-based label transfer (e.g., via Seurat interop) is required — SingCellaR
  lacks automatic object transformation to interact with packages such as Seurat
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SingCellaR
  role: R package hosting the AUCell workflow functions (get_gmtGeneSets, Build_AUCell_Rankings,
    Run_AUCell, get_cells_annotation, plot_cells_annotation) on the SingleCellExperiment
    object
  repo: https://github.com/supatt-lab/SingCellaR
- name: AUCell
  role: Enrichment-based ranking/scoring engine underlying Build_AUCell_Rankings and
    Run_AUCell
- name: R
  role: Runtime environment in which the SingCellaR AUCell pipeline is executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:28ad0962fc22ae8f47c2cdceee726c5ab395319c2385d3b33b943b94e57b23c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/supatt-lab/SingCellaR
  tools:
  - AUCell
  - R
  - SingCellaR
  edam_operation: http://edamontology.org/operation_1777
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# cell-type-annotation-from-signature-scores

## Summary

Assign per-cell lineage labels by computing AUCell enrichment scores for curated signature gene sets (GMT) within SingCellaR, then applying gene-set-specific AUC cutoffs and visualizing the annotations on UMAP.

## When to use

Use when a normalized scRNA-seq object (e.g., SingCellaR Human_HSPC) needs cell-type labels from lineage signature gene sets rather than marker-gene inspection or reference mapping.

## When NOT to use

- When only 10x Cell Ranger raw outputs are available — the object must already contain normalized UMIs after normalize_UMIs
- When the biological system lacks relevant curated signatures and customized gene sets would have to be compiled from scratch
- When reference-based label transfer (e.g., via Seurat interop) is required — SingCellaR lacks automatic object transformation to interact with packages such as Seurat

## Inputs

- Integrated SingCellaR object with normalized UMI matrix (Human_HSPC)
- GMT file of lineage signature gene sets (75 curated hematopoiesis sets)
- AUCell rankings file (cached output of Build_AUCell_Rankings)

## Outputs

- Per-cell AUCell gene-set enrichment (AUC) score table covering all 75 gene sets
- Per-cell cell-type annotations retrievable via get_cells_annotation(object)
- UMAP plot colored by assigned lineage annotations (Figure 7A-equivalent, e.g., aucell_celltype_umap.png)

## How to apply

Load the integrated object containing the normalized UMI matrix (via get_normalized_umi) into SingCellaR in R, then read the lineage signature sets from a GMT file with get_gmtGeneSets(gmt.file) and confirm them with listAvailableGeneSets(gmt.file); the protocol provides 75 pre-curated hematopoiesis gene sets. Build AUCell rankings with Build_AUCell_Rankings(object, AUCell_buildRankings.file, nCores, plotStats, splitByBlocks), caching to the rankings file (use Build_AUCell_Rankings_Fast with block.cell.size for large-scale data to avoid running time and memory issues). Compute per-cell gene-set enrichment with Run_AUCell(object, AUCell_buildRankings.file, geneSets.gmt.file, n.random.genes), then apply gene-set-specific AUC score cutoffs to assign cell-type labels retrievable via get_cells_annotation(object). Visualize assigned labels on the UMAP embedding with plot_cells_annotation(object, type), as in the protocol's Figure 7A where the HSC/MPP signature is uniformly enriched across cluster 1 cells.

## Related tools

- **SingCellaR** (R package hosting the AUCell workflow functions (get_gmtGeneSets, Build_AUCell_Rankings, Run_AUCell, get_cells_annotation, plot_cells_annotation) on the SingleCellExperiment object) — https://github.com/supatt-lab/SingCellaR
- **AUCell** (Enrichment-based ranking/scoring engine underlying Build_AUCell_Rankings and Run_AUCell)
- **R** (Runtime environment in which the SingCellaR AUCell pipeline is executed)

## Examples

```
Run_AUCell(object, AUCell_buildRankings.file, geneSets.gmt.file, n.random.genes); plot_cells_annotation(object, type)
```

## Evaluation signals

- Rankings file and AUCell score table exist and contain all 75 curated gene sets
- UMAP plot renders cells colored by assigned lineage annotations, matching Figure 7A
- HSC/MPP gene signature is uniformly enriched across the majority of cells in cluster 1
- get_cells_annotation(object) returns a non-empty annotation vector for all cells
- Permutation control: n.random.genes supplied to Run_AUCell yields expected null distributions for cutoff setting

## Limitations

- AUCell_buildRankings has running time and memory issues on large-scale datasets; use Build_AUCell_Rankings_Fast with block.cell.size
- Only 75 pre-curated hematopoiesis gene sets are provided; users must compile and curate customized signature gene sets for other systems
- UMAP/FDG plot rotations can differ from the protocol due to software version and seed number differences

## Evidence

- [full_text] Read the lineage signature gene sets from the GMT file using get_gmtGeneSets(gmt.file): [evidence span withheld — anchor status: not_found]
- [discussion] Build_AUCell_Rankings function for a large-scale dataset: "running time and memory issues when performing the AUCell_buildRankings function for a large-scale dataset."
- [discussion] 75 gene sets curated from previous studies relevant to hematopoiesis: "the user would have to compile and curate customized gene sets for the relevant system of interest. In this protocol, we provide 75 gene sets curated from previous studies relevant to hematopoiesis."
- [results] HSC/MPP gene signature uniformly enriched across the majority of cells in cluster 1: [evidence span withheld — anchor status: not_found]
- [readme] example datasets described in the tutorial can be downloaded: "The example datasets described in the tutorial can be downloaded from this link https://drive.google.com/drive/folders/1ko6XGu1OEoVdWDd2FdMlucx7IBK7Dpq0"
