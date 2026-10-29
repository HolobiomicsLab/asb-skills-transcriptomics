---
name: aucell-gene-set-enrichment-scoring
description: Use when you have a normalized UMI matrix in SingCellaR and pre-curated
  lineage signature gene sets (GMT) and need enrichment-based per-cell cell-type annotation
  rather than marker-gene-only labeling.
when_to_use_negative:
- When no curated signature gene sets exist for your system — the article notes 'the
  user would have to compile and curate customized gene sets for the relevant system
  of interest', so enrichment scoring is not applicable until such sets are built
- When the input is raw unnormalized counts — the workflow requires the normalized
  UMI matrix produced by normalize_UMIs(object, scale.factor, use.scaled.factor)
- When you only need integrative batch correction (runScanorama/kBET) rather than
  cell-type annotation
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SingCellaR
  role: Host R package that stores the integrated Human_HSPC object, wraps AUCell
    ranking/scoring functions (Build_AUCell_Rankings, Run_AUCell), and provides cell-type
    annotation retrieval and UMAP plotting
  repo: https://github.com/supatt-lab/SingCellaR
- name: AUCell
  role: Enrichment-scoring engine used within SingCellaR to rank genes per cell and
    compute gene-set AUC scores against the lineage-signature GMT
- name: R
  role: Execution environment for SingCellaR and the AUCell-based annotation workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:13a63c5b4467a5af0f277439923e9182f37a76e20d06a5be550da14cbda679f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/supatt-lab/SingCellaR
  tools:
  - AUCell
  - R
  - SingCellaR
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# aucell-gene-set-enrichment-scoring

## Summary

Computes per-cell AUCell enrichment scores for curated lineage signature gene sets (GMT format) in the SingCellaR-integrated Human_HSPC scRNA-seq object, then applies gene-set-specific AUC cutoffs to assign cell-type annotations visualized on UMAP (Figure 7A-equivalent).

## When to use

Use when you have a normalized UMI matrix in SingCellaR and pre-curated lineage signature gene sets (GMT) and need enrichment-based per-cell cell-type annotation rather than marker-gene-only labeling.

## When NOT to use

- When no curated signature gene sets exist for your system — the article notes 'the user would have to compile and curate customized gene sets for the relevant system of interest', so enrichment scoring is not applicable until such sets are built
- When the input is raw unnormalized counts — the workflow requires the normalized UMI matrix produced by normalize_UMIs(object, scale.factor, use.scaled.factor)
- When you only need integrative batch correction (runScanorama/kBET) rather than cell-type annotation

## Inputs

- SingCellaR object containing the normalized UMI matrix (e.g. Human_HSPC integrated dataset)
- GMT file of lineage signature gene sets (75 pre-curated hematopoiesis gene sets in the protocol)
- AUCell rankings file path (AUCell_buildRankings.file) and nCores setting

## Outputs

- Per-cell AUCell enrichment (AUC) score table for all 75 gene sets
- Per-cell cell-type annotations retrievable via get_cells_annotation(object)
- UMAP plot colored by assigned lineage annotations (aucell_celltype_umap.png, Figure 7A-equivalent)

## How to apply

Load the integrated Human_HSPC object with its normalized UMI matrix (get_normalized_umi) into SingCellaR in R. Read the lineage signatures with get_gmtGeneSets(gmt.file) and verify the expected sets via listAvailableGeneSets(gmt.file) — the protocol supplies 75 pre-curated hematopoiesis gene sets. Build AUCell rankings with Build_AUCell_Rankings(object, AUCell_buildRankings.file, nCores, plotStats, splitByBlocks), caching to the rankings file; for large datasets use Build_AUCell_Rankings_Fast with block.cell.size to avoid running-time and memory issues. Score each cell with Run_AUCell(object, AUCell_buildRankings.file, geneSets.gmt.file, n.random.genes), then apply gene-set-specific AUC cutoffs to assign labels retrievable via get_cells_annotation(object), and render them on UMAP with plot_cells_annotation(object, type), saving the result as aucell_celltype_umap.png.

## Related tools

- **SingCellaR** (Host R package that stores the integrated Human_HSPC object, wraps AUCell ranking/scoring functions (Build_AUCell_Rankings, Run_AUCell), and provides cell-type annotation retrieval and UMAP plotting) — https://github.com/supatt-lab/SingCellaR
- **AUCell** (Enrichment-scoring engine used within SingCellaR to rank genes per cell and compute gene-set AUC scores against the lineage-signature GMT)
- **R** (Execution environment for SingCellaR and the AUCell-based annotation workflow)

## Examples

```
Run_AUCell(object, AUCell_buildRankings.file, geneSets.gmt.file, n.random.genes)
```

## Evaluation signals

- The AUCell rankings file and AUCell score table exist and contain all 75 curated gene sets
- listAvailableGeneSets(gmt.file) confirms the expected gene sets before scoring
- The HSC/MPP gene signature is uniformly enriched across the majority of cells in cluster 1 on the UMAP, confirming enrichment-based scoring worked
- The saved UMAP plot renders cells colored by assigned lineage annotations matching Figure 7A
- get_cells_annotation(object) returns non-empty, gene-set-specific AUC-cutoff-derived labels for all cells

## Limitations

- AUCell_buildRankings has running time and memory issues on large-scale datasets; mitigate with Build_AUCell_Rankings_Fast and block.cell.size
- Only 75 pre-curated hematopoiesis gene sets are provided; other systems require compiling and curating customized signature gene sets
- SingCellaR requires signature gene sets to perform the cell type annotation analysis, so enrichment annotation cannot proceed without a GMT input
- UMAP plot rotations may differ from the protocol due to software version and seed number differences

## Evidence

- [discussion] SingCellaR requires signature gene sets to perform the cell type annotation analysis.: "SingCellaR requires signature gene sets to perform the cell type annotation analysis."
- [discussion] AUCell_buildRankings memory issues: "running time and memory issues when performing the AUCell_buildRankings function for a large-scale dataset."
- [discussion] 75 curated hematopoiesis gene sets: "we provide 75 gene sets curated from previous studies relevant to hematopoiesis."
- [methods] normalized UMI prerequisite: "`normalize_UMIs(object, scale.factor, use.scaled.factor)`"
- [readme] SingCellaR README description: "an integrative analysis tool for analysing single-cell RNA sequencing data"
