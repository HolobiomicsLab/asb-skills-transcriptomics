---
name: ctgene-top-percent-ranking
description: Use when building or customizing deCS cell type specificity reference
  panels from bulk or single-cell expression matrices, where per-cell-type marker
  gene sets must be defined by a ranked specificity cutoff.
when_to_use_negative:
- The query is novel cell types absent from all reference panels — deCS is a mapping-based
  annotation toolkit and 'fails to characterize novel cell types'
- Cancer cell annotation where malignant expression profiles diverge from the healthy-cell
  references
- Tissue-specific annotation is required and the reference is not HCAF — deCS does
  not take tissue information (except for HCAF) into account
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: deCS
  role: Provides the reference panels, score conventions, and ref_panel_markers_ratio
    ranking logic used to define CTGene sets
  repo: https://github.com/bsml320/deCS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2022.04.001
    title: '<i>deCS</i> : A Tool for Systematic Cell Type Annotations of Single-Cell
      RNA Sequencing Data Among Human Tissues'
  source_content_hash: sha256:a71cf6b414c78b1fcd5cf0326a150875ab3fe3efcafee9e4921fab02724539a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/deCS
  tools:
  - deCS
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# ctgene-top-percent-ranking

## Summary

Reconstruct deCS cell type-specific gene (CTGene) reference panels by ranking genes within each cell type using t-statistics for bulk RNA-seq references (BlueprintEncode, DICE, MonacoImmune) and row-wise z-scores for scRNA-seq references (HCL, HCAF), then retaining the top 5% scoring genes per cell type.

## When to use

Use when building or customizing deCS cell type specificity reference panels from bulk or single-cell expression matrices, where per-cell-type marker gene sets must be defined by a ranked specificity cutoff.

## When NOT to use

- The query is novel cell types absent from all reference panels — deCS is a mapping-based annotation toolkit and 'fails to characterize novel cell types'
- Cancer cell annotation where malignant expression profiles diverge from the healthy-cell references
- Tissue-specific annotation is required and the reference is not HCAF — deCS does not take tissue information (except for HCAF) into account

## Inputs

- Bulk RNA-seq reference expression matrices (BlueprintEncode, DICE, MonacoImmune at main and fine resolutions)
- scRNA-seq reference expression matrices (Human cell landscape HCL, Human cell atlas of fetal HCAF)
- ref_panel_markers_ratio cutoff value (e.g., 0.05 for top 5%)

## Outputs

- Per-cell-type CTGene lists for each reference panel (e.g., ctgene_panels.tsv)
- Cell type specificity score matrices (t-score matrices for bulk panels; HCL_z_score and HCAF_z_score matrices)

## How to apply

Load the built-in reference data (BlueprintEncode_main/fine, DICE_main/fine, MonacoImmune_main/fine t-score panels; Human_cell_landscape and Human_cell_atlas_of_fetal z-score panels). For bulk references, fit per-gene regression models contrasting each cell type against the remainder and extract t-statistics as the cell-type specificity score matrix. For scRNA-seq references, average expression per cell cluster and compute row-wise z-scores via t(scale(t(...))). Apply the ref_panel_markers_ratio cutoff — the default retains the top 5% of genes with the highest t-score/z-score as CTgenes for every cell type in every panel — then assemble per-cell-type CTGene lists (e.g., written to ctgene_panels.tsv).

## Related tools

- **deCS** (Provides the reference panels, score conventions, and ref_panel_markers_ratio ranking logic used to define CTGene sets) — https://github.com/bsml320/deCS

## Examples

```
data(MonacoImmune_main); data(Human_cell_landscape)  # load pre-ranked t-score / z-score CTGene panels in deCS
```

## Evaluation signals

- Each cell type in each panel contains exactly the configured top fraction of ranked genes (e.g., 5% of assayed genes)
- Genes within each cell type are ordered by descending specificity score
- Bulk-derived panels (BlueprintEncode, DICE, MonacoImmune) follow the t-score convention; HCL/HCAF panels follow the z-score convention
- The resulting CTGene sets reproduce the built-in deCS data objects (e.g., MonacoImmune_main_t_score, Human_cell_landscape) when the same ratio is applied

## Limitations

- deCS does not take tissue information (except for HCAF) into account, which may bias annotation results
- As a mapping-based annotation toolkit, deCS fails to characterize novel cell types
- deCS may be challenging for cancer cell annotations due to expression profile differences between healthy and malignant cells
- The ref_panel_markers_ratio cutoff is user-defined and choice of the fraction (e.g., top 5%) affects panel composition

## Evidence

- [methods] top 5% t-score/z-score CTgenes: "We allow the user to define the cutoff values, e.g., the top 5% genes with highest t-score/z-score as CTgenes"
- [methods] reference panel collection: "deCS collected several cell type reference panels, including BlueprintEncode, the Database of Immune Cell Expression (DICE), MonacoImmune, human cell landscape, human cell atlas of fetal"
- [readme] built-in t-score and z-score data objects: "# Human_cell_landscape (HCL_z_score)
data(Human_cell_landscape)
# Human_cell_atlas_of_fetal (HCAF_z_score)
data(Human_cell_atlas_of_fetal)"
- [readme] row-wise z-score scaling of cluster-averaged expression: [evidence span withheld — anchor status: external_doc]
- [other] regression-based specificity scoring: [evidence span withheld — anchor status: not_found]
