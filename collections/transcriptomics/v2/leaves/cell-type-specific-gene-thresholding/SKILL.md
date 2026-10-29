---
name: cell-type-specific-gene-thresholding
description: Use when running deCS.fisher on a marker gene list or GWAS trait-associated
  gene list and you must first convert a cell type specificity score panel into discrete
  per-cell-type gene sets via a score cutoff.
when_to_use_negative:
- The query input is a gene expression profile rather than a gene list — use deCS.correlation()
  with the raw specificity score matrix instead of thresholding to CTgenes
- Novel cell types not represented in the reference panels need annotation — deCS,
  as a mapping-based toolkit, fails to characterize novel cell types regardless of
  cutoff
- Cancer/malignant cells, where expression profile differences from healthy reference
  cells can bias thresholded marker sets
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
tools:
- name: deCS
  role: Provides the built-in specificity score reference panels (BlueprintEncode_main,
    DICE_main, MonacoImmune_main, Human_cell_landscape, Human_cell_atlas_of_fetal)
    whose t-scores/z-scores are thresholded into CTgenes, and the deCS.fisher function
    that consumes them for enrichment-based annotation
  repo: https://github.com/bsml320/deCS
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2022.04.001
    title: '<i>deCS</i> : A Tool for Systematic Cell Type Annotations of Single-Cell
      RNA Sequencing Data Among Human Tissues'
  source_content_hash: sha256:eb5c0ad7d6dfa386116cc1740236c90fa7b35953404b0c885932eaadfe2f64a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/deCS
  tools:
  - deCS
  - ggplot2
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# cell-type-specific-gene-thresholding

## Summary

Defines cell type-specific genes (CTgenes) for each cell type in a deCS reference panel (e.g., BlueprintEncode_main, MonacoImmune_main) by thresholding pre-calculated t-score/z-score specificity matrices at a user-set ratio such as the top 5% highest-scoring genes. These CTgene sets are the overlap universe for deCS.fisher enrichment tests and GWAS trait–cell type association analyses.

## When to use

Use when running deCS.fisher on a marker gene list or GWAS trait-associated gene list and you must first convert a cell type specificity score panel into discrete per-cell-type gene sets via a score cutoff.

## When NOT to use

- The query input is a gene expression profile rather than a gene list — use deCS.correlation() with the raw specificity score matrix instead of thresholding to CTgenes
- Novel cell types not represented in the reference panels need annotation — deCS, as a mapping-based toolkit, fails to characterize novel cell types regardless of cutoff
- Cancer/malignant cells, where expression profile differences from healthy reference cells can bias thresholded marker sets

## Inputs

- Pre-calculated cell type specificity score reference panel (t-score/z-score matrix, e.g., MonacoImmune_main, BlueprintEncode_main, DICE_main, Human_cell_landscape, Human_cell_atlas_of_fetal)
- ref_panel_markers_ratio cutoff (e.g., 0.05 for top 5% scoring genes per cell type)

## Outputs

- Per-cell-type CTgene sets (cell type-specific gene lists) used as the overlap reference in deCS.fisher
- Downstream: Fisher's exact test P values and intersection ratios (IR) per query cluster/cell type, and an annotation table (Query, Top candidates, IR, P value, Cell_labels)

## How to apply

Load a pre-calculated cell type specificity score reference panel from the installed deCS package (e.g., data(MonacoImmune_main) or data(BlueprintEncode_main) for t-scores, Human_cell_landscape/HCAF for z-scores). For each cell type (column), rank genes by specificity score and take the top fraction defined by ref_panel_markers_ratio, defaulting to 0.05 (top 5% of genes by t-score/z-score) to define that cell type's CTgenes. Users may adjust this cutoff; the ratio controls the stringency of the Fisher's exact test background because a smaller CTgene set makes overlap enrichment harder to achieve. These per-cell-type CTgene sets are then intersected with the query gene list (columns 'cluster' and 'gene') to build 2x2 contingency tables from which Fisher's exact test P values and intersection ratios (IR = Q∩R/Q) are computed, with assignments filtered by p_threshold (e.g., 1e-3) and cell_type_threshold (e.g., 0.05).

## Related tools

- **deCS** (Provides the built-in specificity score reference panels (BlueprintEncode_main, DICE_main, MonacoImmune_main, Human_cell_landscape, Human_cell_atlas_of_fetal) whose t-scores/z-scores are thresholded into CTgenes, and the deCS.fisher function that consumes them for enrichment-based annotation) — https://github.com/bsml320/deCS

## Examples

```
library(deCS); data(MonacoImmune_main); res <- deCS.fisher(pbmc_top10_markers_list, MonacoImmune_main_t_score, ref_panel_markers_ratio = 0.05, p_threshold = 1e-3, cell_type_threshold = 0.05)
```

## Evaluation signals

- ref_panel_markers_ratio is explicitly set (default 0.05) and CTgenes contain only the top 5% highest t-score/z-score genes per cell type
- Calling deCS.fisher(pbmc_top10_markers_list, MonacoImmune_main_t_score) reproduces the reference PBMC annotation (e.g., CD14+ Mono -> Monocytes, NK -> NK cells)
- Reproduced GWAS trait–cell type heatmap shows neurons enriched for neuropsychiatric/college/education traits and myeloid cells associated with Alzheimer's disease, matching Figure 7
- Assignments passing no cutoff are labeled 'Undetermined cells', confirming intersection_threshold/p_threshold/cell_type_threshold gating is active
- Adjusted P values (e.g., bonferroni via p.adjust.methods) fall below p_threshold (e.g., 1e-3) for called assignments

## Limitations

- deCS does not take tissue information (except for HCAF) into account, which may bias annotation results
- As a mapping-based annotation toolkit, deCS fails to characterize novel cell types no matter how CTgenes are thresholded
- deCS may be challenging for cancer cell annotations due to expression profile differences between healthy and malignant cells
- The default top-5% cutoff is user-adjustable; overly permissive or stringent ratios shift Fisher enrichment significance and can mislabel clusters

## Evidence

- [methods] CTgene cutoff via ref_panel_markers_ratio: "We allow the user to define the cutoff values, e.g., the top 5% genes with highest t-score/z-score as CTgenes"
- [methods] deCS.fisher signature: "deCS.fisher(markers_list, ref_panel, type, ref_panel_markers_ratio, top_n, p.adjust.methods, label, intersection_threshold, p_threshold, cell_type_threshold, plot_figure)"
- [methods] Fisher filter thresholds: "p_threshold = 1e-3, cell_type_threshold = 0.05"
- [readme] Built-in reference panels: "deCS collected several cell type reference panels, including BlueprintEncode, the Database of Immune Cell Expression (DICE), MonacoImmune, human cell landscape, human cell atlas of fetal et al."
- [methods] Undetermined label: "give `cor_threshold` to avoid mis-annotation (regard as "Undetermined cells")"
