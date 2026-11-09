---
name: bulk-expression-reference-processing
description: Use when building or verifying deCS CTGene reference panels from bulk
  (BlueprintEncode, DICE, MonacoImmune) or single-cell (HCL, HCAF) expression references,
  or when the ref_panel_markers_ratio cutoff must be re-derived.
when_to_use_negative:
- The query is a gene expression profile to be annotated against existing panels —
  use deCS.correlation() or deCS.fisher() on the pre-computed panels instead of rebuilding
  them
- Marker gene lists from a curated database such as CellMatch are already available
  and sufficient for annotation
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: deCS
  role: R package providing the built-in reference panels (t-score and z-score data
    objects) from which CTGene sets are derived
  repo: https://github.com/bsml320/deCS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2022.04.001
    title: '<i>deCS</i> : A Tool for Systematic Cell Type Annotations of Single-Cell
      RNA Sequencing Data Among Human Tissues'
  source_content_hash: sha256:9216c84dc6cb40b8ff35c41e3e8a8d805cb11cd69df5a8827fd976764c6564b3
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
yes_rate: 1.0
n_claims: 11
---

# bulk-expression-reference-processing

## Summary

Reconstruct deCS cell type-specific gene (CTGene) reference panels by scoring genes in bulk RNA-seq references (BlueprintEncode, DICE, MonacoImmune) with per-cell-type t-statistics and in scRNA-seq references (HCL, HCAF) with row-wise z-scores, then retaining the top 5% scored genes per cell type.

## When to use

Use when building or verifying deCS CTGene reference panels from bulk (BlueprintEncode, DICE, MonacoImmune) or single-cell (HCL, HCAF) expression references, or when the ref_panel_markers_ratio cutoff must be re-derived.

## When NOT to use

- The query is a gene expression profile to be annotated against existing panels — use deCS.correlation() or deCS.fisher() on the pre-computed panels instead of rebuilding them
- Marker gene lists from a curated database such as CellMatch are already available and sufficient for annotation

## Inputs

- Bulk RNA-seq reference expression datasets: BlueprintEncode, DICE, MonacoImmune (main and fine resolutions)
- Single-cell reference expression datasets: Human_cell_landscape (HCL), Human_cell_atlas_of_fetal (HCAF)
- ref_panel_markers_ratio cutoff (default top 5%)

## Outputs

- Per-cell-type specificity score matrices (e.g., MonacoImmune_main_t_score, HCL_z_score, HCAF_z_score)
- Per-cell-type CTGene lists ranked by descending specificity score (e.g., ctgene_panels.tsv)

## How to apply

Load the built-in deCS reference panels (BlueprintEncode_main/fine, DICE_main/fine, MonacoImmune_main/fine t-score data; Human_cell_landscape and Human_cell_atlas_of_fetal z-score data). For each bulk reference, fit per-gene regression models contrasting each cell type against the remainder and extract t-statistics as the cell-type specificity score matrix. For each scRNA-seq reference, average expression per cell cluster and compute row-wise z-scores with t(scale(t(...))). Apply the ref_panel_markers_ratio cutoff — by default the top 5% of genes with the highest t-score/z-score — to define CTgenes for every cell type in every panel, then assemble per-cell-type CTGene lists (e.g., written to ctgene_panels.tsv).

## Related tools

- **deCS** (R package providing the built-in reference panels (t-score and z-score data objects) from which CTGene sets are derived) — https://github.com/bsml320/deCS

## Examples

```
library(deCS); data(MonacoImmune_main); data(Human_cell_landscape); ctgenes <- names(sort(MonacoImmune_main[,ct], decreasing=TRUE))[1:round(0.05*nrow(MonacoImmune_main))]
```

## Evaluation signals

- Each cell type in each panel contains exactly the configured top fraction of ranked genes (5% of assayed genes under the default ref_panel_markers_ratio)
- Genes within each CTGene list are ordered by descending specificity score
- Bulk-derived panels (BlueprintEncode, DICE, MonacoImmune) follow the t-score convention, while HCL/HCAF panels follow the z-score convention
- Loaded built-in data objects match expected names (e.g., BlueprintEncode_main_t_score, Human_cell_landscape) after data() calls in R

## Limitations

- deCS does not take tissue information (except for HCAF) into account, which may bias annotation results
- Cutoff values (e.g., the top 5% of genes by t-score/z-score) are user-definable, so panel contents depend on the chosen ref_panel_markers_ratio

## Evidence

- [methods] t-score/z-score ranking rule: "We allow the user to define the cutoff values, e.g., the top 5% genes with highest t-score/z-score as CTgenes"
- [readme] reference panel collection: "deCS collected several cell type reference panels, including BlueprintEncode, the Database of Immune Cell Expression (DICE), MonacoImmune, human cell landscape, human cell atlas of fetal"
- [readme] score convention in built-in data: "# BlueprintEncode_main_t_score
data(BlueprintEncode_main)"
- [readme] z-score computation on query clusters: [evidence span withheld — anchor status: external_doc]
