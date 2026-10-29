---
name: cell-type-specificity-tscore-estimation
description: 'Use when building or verifying deCS CTGene reference panels: you need
  per-cell-type ranked gene lists derived from bulk (t-score) or single-cell (z-score)
  specificity scores before annotation or enrichment.'
when_to_use_negative:
- You already have a pre-computed deCS reference panel and only need to run deCS.correlation()
  or deCS.fisher() annotation on a query
- The query is a raw scRNA-seq matrix needing clustering/normalization first (NormalizeData,
  FindVariableFeatures, ScaleData, RunPCA, FindNeighbors) rather than reference panel
  construction
- You need to discover novel cell types — deCS is a mapping-based toolkit and fails
  to characterize novel cell types
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: deCS
  role: R package providing the built-in reference panels (t-score/z-score matrices)
    and the ref_panel_markers_ratio-based CTGene definition used for cell type specificity
    scoring
  repo: https://github.com/bsml320/deCS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2022.04.001
    title: '<i>deCS</i> : A Tool for Systematic Cell Type Annotations of Single-Cell
      RNA Sequencing Data Among Human Tissues'
  source_content_hash: sha256:04944fb72885adb4e4387b78c25cdde1502b5f2592deabb48459993074a27730
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/deCS
  tools:
  - deCS
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# cell-type-specificity-tscore-estimation

## Summary

Reconstruct deCS cell type specificity (CTGene) reference panels by computing per-gene t-statistics for bulk RNA-seq references (BlueprintEncode, DICE, MonacoImmune) and row-wise z-scores for scRNA-seq references (HCL, HCAF), then retaining the top 5% most specific genes per cell type.

## When to use

Use when building or verifying deCS CTGene reference panels: you need per-cell-type ranked gene lists derived from bulk (t-score) or single-cell (z-score) specificity scores before annotation or enrichment.

## When NOT to use

- You already have a pre-computed deCS reference panel and only need to run deCS.correlation() or deCS.fisher() annotation on a query
- The query is a raw scRNA-seq matrix needing clustering/normalization first (NormalizeData, FindVariableFeatures, ScaleData, RunPCA, FindNeighbors) rather than reference panel construction
- You need to discover novel cell types — deCS is a mapping-based toolkit and fails to characterize novel cell types

## Inputs

- Bulk RNA-seq reference expression datasets (BlueprintEncode, DICE, MonacoImmune at main and fine resolutions)
- scRNA-seq reference datasets (Human_cell_landscape, Human_cell_atlas_of_fetal)
- ref_panel_markers_ratio cutoff (default: top 5% genes)

## Outputs

- Per-cell-type specificity score matrices (t-score for bulk panels, z-score for HCL/HCAF)
- CTGene marker gene lists per cell type per panel (ctgene_panels.tsv)

## How to apply

Load the built-in deCS reference panels: BlueprintEncode, DICE, and MonacoImmune at main and fine resolutions as pre-computed t-score matrices (e.g., MonacoImmune_main_t_score), and Human_cell_landscape and Human_cell_atlas_of_fetal as z-score matrices. For bulk references, fit per-gene regression models contrasting each cell type against the remainder and extract t-statistics; for scRNA-seq references, average expression per cell cluster and compute row-wise z-scores with t(scale(t(...))). Apply the ref_panel_markers_ratio cutoff, keeping the top 5% of genes with the highest t-score/z-score as CTgenes for every cell type. Assemble the per-cell-type CTGene lists from all score matrices into ctgene_panels.tsv for downstream deCS.correlation() or deCS.fisher() annotation.

## Related tools

- **deCS** (R package providing the built-in reference panels (t-score/z-score matrices) and the ref_panel_markers_ratio-based CTGene definition used for cell type specificity scoring) — https://github.com/bsml320/deCS

## Examples

```
data(MonacoImmune_main); data(Human_cell_landscape); ctgenes <- apply(MonacoImmune_main, 2, function(x) names(head(sort(x, decreasing=TRUE), n=0.05*length(x))))
```

## Evaluation signals

- Each cell type in each panel contains exactly the configured top fraction (5%) of ranked genes
- CTGene lists are ordered by descending specificity score
- Bulk panels (BlueprintEncode, DICE, MonacoImmune) follow the t-score convention while HCL/HCAF follow the z-score convention
- For scRNA-seq references, z-scores are row-wise across clusters via t(scale(t(...))) of log2(AverageExpression + 1) values
- ctgene_panels.tsv is written with complete per-cell-type gene lists for all score matrices

## Limitations

- deCS does not take tissue information (except for HCAF) into account, which may bias annotation results
- As a mapping-based annotation toolkit, deCS fails to characterize novel cell types
- deCS may be challenging for cancer cell annotations due to expression profile differences between healthy and malignant cells
- Users must define cutoff values themselves (e.g., the top 5% genes with highest t-score/z-score as CTgenes)

## Evidence

- [methods] ref_panel_markers_ratio: "We allow the user to define the cutoff values, e.g., the top 5% genes with highest t-score/z-score as CTgenes"
- [readme] built-in reference panels: "deCS collected several cell type reference panels, including BlueprintEncode, the Database of Immune Cell Expression (DICE), MonacoImmune, human cell landscape, human cell atlas of fetal et al."
- [readme] loading convention: "# MonacoImmune_main_t_score
data(MonacoImmune_main)
# MonacoImmune_fine_t_score
data(MonacoImmune_fine)
# Human_cell_landscape (HCL_z_score)
data(Human_cell_landscape)"
- [discussion] tissue information limitation: "First, deCS does not take tissue information (except for HCAF) into account which may bias the annotation results"
- [readme] z-score calculation: [evidence span withheld — anchor status: external_doc]
