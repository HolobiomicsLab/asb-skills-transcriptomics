---
name: multi-reference-ctgene-panel-construction
description: Use when building or reproducing cell type specificity marker panels
  from deCS reference expression datasets, or when t-score/z-score-based CTgene lists
  are needed for cell type enrichment or annotation.
when_to_use_negative:
- The task is annotating a query scRNA-seq dataset against an already-built reference
  panel rather than reconstructing the panels themselves
- The query input is a marker gene list to be tested with deCS.fisher against existing
  CTgene panels rather than deriving new panels
- Novel/uncharacterized cell types are expected — deCS is a mapping-based toolkit
  and fails to characterize novel cell types
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: deCS
  role: Provides the reference expression panels, t-score/z-score scoring convention,
    and ref_panel_markers_ratio filter used to define CTgene sets per cell type
  repo: https://github.com/bsml320/deCS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2022.04.001
    title: '<i>deCS</i> : A Tool for Systematic Cell Type Annotations of Single-Cell
      RNA Sequencing Data Among Human Tissues'
  source_content_hash: sha256:25b7f14dd9ca2a0749214647a3f23178f5664066e307929cb5b6d9edef83903c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/deCS
  tools:
  - deCS
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# multi-reference-ctgene-panel-construction

## Summary

Reconstructs deCS cell type-specific gene (CTGene) reference panels by scoring every gene's cell type specificity — t-statistics for bulk RNA-seq references (BlueprintEncode, DICE, MonacoImmune) and z-scores for scRNA-seq references (HCL, HCAF) — and retaining the top 5% genes per cell type.

## When to use

Use when building or reproducing cell type specificity marker panels from deCS reference expression datasets, or when t-score/z-score-based CTgene lists are needed for cell type enrichment or annotation.

## When NOT to use

- The task is annotating a query scRNA-seq dataset against an already-built reference panel rather than reconstructing the panels themselves
- The query input is a marker gene list to be tested with deCS.fisher against existing CTgene panels rather than deriving new panels
- Novel/uncharacterized cell types are expected — deCS is a mapping-based toolkit and fails to characterize novel cell types

## Inputs

- Bulk RNA-seq reference expression datasets: BlueprintEncode, DICE, MonacoImmune (main and fine resolutions)
- scRNA-seq reference datasets: Human cell landscape (HCL) and Human cell atlas of fetal (HCAF)
- ref_panel_markers_ratio cutoff (default top 5% of ranked genes)

## Outputs

- Cell type specificity t-score matrices for bulk panels (e.g., MonacoImmune_main_t_score)
- Cell type specificity z-score matrices for scRNA-seq panels (HCL_z_score, HCAF_z_score)
- Per-cell-type CTGene marker lists assembled across all reference panels (e.g., ctgene_panels.tsv)

## How to apply

Load the bulk reference panels (BlueprintEncode_main/fine, DICE_main/fine, MonacoImmune_main/fine) and single-cell references (Human_cell_landscape, Human_cell_atlas_of_fetal) shipped with the deCS R package. For each bulk reference, fit per-gene regression models contrasting each cell type against the remainder and extract the t-statistics as the cell type specificity t-score matrix (e.g., MonacoImmune_main_t_score). For each scRNA-seq reference, average expression per cell cluster and compute row-wise z-scores across clusters via t(scale(t(...))) to obtain HCL_z_score and HCAF_z_score. Apply the ref_panel_markers_ratio cutoff — by default the top 5% of genes with the highest t-score/z-score — to define the CTgenes for every cell type in every panel, then assemble per-cell-type CTGene lists (e.g., written to ctgene_panels.tsv).

## Related tools

- **deCS** (Provides the reference expression panels, t-score/z-score scoring convention, and ref_panel_markers_ratio filter used to define CTgene sets per cell type) — https://github.com/bsml320/deCS

## Examples

```
library(deCS); data(MonacoImmune_main); data(Human_cell_landscape); ctgenes <- apply(MonacoImmune_main, 2, function(x) names(head(sort(x, decreasing=TRUE), length(x)*0.05)))
```

## Evaluation signals

- Each cell type in each panel contains exactly the configured top fraction of ranked genes (5% of assayed genes) ordered by descending specificity score
- Bulk-derived panels (BlueprintEncode, DICE, MonacoImmune) follow the t-score convention while HCL/HCAF panels follow the z-score convention
- Loaded panel objects match the expected built-in data names (e.g., MonacoImmune_main_t_score, HCL_z_score) and gene x cell type dimensions
- z-score matrices are row-wise standardized across cell types (mean 0, sd 1 per gene), consistent with t(scale(t(...)))

## Limitations

- deCS does not take tissue information (except for HCAF) into account, which may bias annotation results
- As a mapping-based annotation toolkit, deCS fails to characterize novel cell types
- Cancer cell annotations may be challenging due to expression profile differences between healthy and malignant cells
- The top-fraction cutoff (e.g., 5%) is user-definable, so panel composition depends on the chosen ref_panel_markers_ratio

## Evidence

- [methods] bulk vs single-cell scoring convention: "We allow the user to define the cutoff values, e.g., the top 5% genes with highest t-score/z-score as CTgenes"
- [methods] reference panels: "deCS collected several cell type reference panels, including BlueprintEncode, the Database of Immune Cell Expression (DICE), MonacoImmune, human cell landscape, human cell atlas of fetal"
- [readme] built-in data loading: "# MonacoImmune_main_t_score
data(MonacoImmune_main)
# Human_cell_landscape (HCL_z_score)
data(Human_cell_landscape)"
- [readme] z-score computation in workflow: [evidence span withheld — anchor status: external_doc]
- [abstract] tool purpose: "Here, we present decoding Cell type Specificity (deCS), an automatic cell type annotation method augmented by a comprehensive collection of human cell type expression profiles and marker genes."
