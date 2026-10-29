---
name: wilcoxon-differential-expression-marker-selection
description: Use when you need cell-type-specific marker gene sets from scRNA-seq
  cluster data — e.g., ten highly specific plus ten low-specificity markers per cell
  type via Wilcoxon rank-sum — as input to ScType gene_sets_prepare/sctype_score.
when_to_use_negative:
- No prior cell-type grouping or labels exist and markers must be selected purely
  from cluster-wide averages without DE testing (e.g., scCATCH-style workflows that
  cannot accept custom markers)
- The goal is variant/SNV calling rather than marker-based annotation (use VarTrix
  instead)
- Input is already a curated marker gene set — skip DE selection and pass markers
  directly to gene_sets_prepare
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ScType
  role: Consumes the Wilcoxon-derived positive/negative marker sets via gene_sets_prepare
    and sctype_score to assign cell types
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: R
  role: Implementation environment for Wilcoxon rank-sum testing, marker selection,
    and the ScType R-package
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: Splatter
  role: Generates simulated scRNA-seq datasets with varying dropout (~50%, ~65%, ~80%)
    whose ground-truth labels drive the Wilcoxon DE marker selection
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-28803-w
    title: Fully-automated and ultra-fast cell-type identification using specific
      marker combinations from single-cell transcriptomic data
  source_content_hash: sha256:69de0c9ff9013033e5a4ee496feaca5ccceab1037ad96bf5b6f9387b1008827f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IanevskiAleksandr/sc-type
  tools:
  - R
  - SCINA
  - ScType
  - Splatter
  - scSorter
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# wilcoxon-differential-expression-marker-selection

## Summary

Selects cell-type marker genes by Wilcoxon rank-sum differential expression between cell types, picking ten highly specific and ten low-specificity markers per cell type to build ScType positive and negative marker sets for annotation and dropout-robustness benchmarks.

## When to use

Use when you need cell-type-specific marker gene sets from scRNA-seq cluster data — e.g., ten highly specific plus ten low-specificity markers per cell type via Wilcoxon rank-sum — as input to ScType gene_sets_prepare/sctype_score.

## When NOT to use

- No prior cell-type grouping or labels exist and markers must be selected purely from cluster-wide averages without DE testing (e.g., scCATCH-style workflows that cannot accept custom markers)
- The goal is variant/SNV calling rather than marker-based annotation (use VarTrix instead)
- Input is already a curated marker gene set — skip DE selection and pass markers directly to gene_sets_prepare

## Inputs

- Normalized/scaled scRNA-seq expression matrix (e.g., Seurat object or scaled matrix, genes x cells)
- Known cell-type labels (from simulation ground truth or prior annotation)
- ScType marker database (ScTypeDB_full.xlsx or ScTypeDB_short.xlsx)

## Outputs

- Per-cell-type marker gene sets: ten highly specific and ten low-specificity markers per cell type
- ScType gs_list with gs_positive and gs_negative gene sets from gene_sets_prepare
- Cell-type-by-cell scoring matrix (es.max) from sctype_score and per-cell-type annotation accuracy (e.g., annotation_accuracy_vs_dropout.csv)

## How to apply

For each simulated or real scRNA-seq dataset (e.g., Splatter-simulated data with 10 cell types at scaled lognormal proportions), perform Wilcoxon rank-sum differential expression between cell types and select the ten most highly specific markers plus ten low-specificity markers per cell type. Feed the positive and negative marker lists into the ScType gene_sets_prepare function against ScTypeDB_full.xlsx to construct gs (positive) and gs2 (negative) gene sets. Run sctype_score with scaled scRNAseqData to assign cell types, and compute per-cell-type annotation accuracy against known labels. The rationale is that ScType's accuracy depends on marker specificity across both cell clusters and cell types, with gene expression multiplied by its cell-type-specificity (Sit) score.

## Related tools

- **ScType** (Consumes the Wilcoxon-derived positive/negative marker sets via gene_sets_prepare and sctype_score to assign cell types) — https://github.com/IanevskiAleksandr/sc-type
- **R** (Implementation environment for Wilcoxon rank-sum testing, marker selection, and the ScType R-package) — https://github.com/IanevskiAleksandr/sc-type
- **Splatter** (Generates simulated scRNA-seq datasets with varying dropout (~50%, ~65%, ~80%) whose ground-truth labels drive the Wilcoxon DE marker selection)

## Examples

```
gs_list <- gene_sets_prepare('ScTypeDB_full.xlsx', 'Immune system'); es.max <- sctype_score(scRNAseqData = scRNAseqData, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)
```

## Evaluation signals

- Exactly ten highly specific and ten low-specificity marker genes are selected per cell type by the Wilcoxon test
- gene_sets_prepare returns non-empty gs_positive and gs_negative sets when matched against ScTypeDB_full.xlsx
- Annotation accuracy against known simulated cell-type labels stays above 90% even at ~80% dropout
- ScType accuracy exceeds comparator methods scSorter and SCINA on the same marker inputs
- Aggregate accuracy summary written to annotation_accuracy_vs_dropout.csv is consistent per method and dropout level

## Limitations

- Drop-out events in scRNA-seq data are not corrected in the current ScType version, which can affect robustness of negative markers
- z-score transformation of gene expression may be suboptimal for single-cell transcriptomic data
- Selection depends on a curated marker database; CellMarker-derived entries did not list source publications, limiting confirmability of marker provenance

## Evidence

- [discussion] ScType marker specificity scoring: [evidence span withheld — anchor status: not_found]
- [discussion] ScType outperformed prior-knowledge comparators: "Our unsupervised approach outperformed the prior knowledge-based methods scSorter30, SCINA33, and scCATCH34"
- [methods] gene_sets_prepare loading: "source("https://raw.githubusercontent.com/IanevskiAleksandr/sc-type/master/R/gene_sets_prepare.R")"
- [methods] sctype_score invocation: "es.max <- sctype_score(scRNAseqData = scRNAseqData, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)"
- [intro] ScType marker specificity requirement: "ensuring the specificity of marker genes"
