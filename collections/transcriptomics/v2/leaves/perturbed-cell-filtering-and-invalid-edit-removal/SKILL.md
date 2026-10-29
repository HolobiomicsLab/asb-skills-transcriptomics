---
name: perturbed-cell-filtering-and-invalid-edit-removal
description: Use when preprocessing pooled CRISPR screening + scRNA-seq data (e.g.
  crop_stimulated Perturb-seq input) before topic modeling, to remove perturbed cells
  whose DE profile resembles controls and perturbations with too few remaining cells.
when_to_use_negative:
- When running an imputation/filtering ablation that requires skipping Cell_filtering
  to compare overall perturbation effect rankings against the full pipeline
- When input is raw fastq or 10X matrix files not yet processed by Input_preprocess/Input_preprocess_10X
  into expression_profile + perturb_information
- When only cell quality control is needed (genes detected, UMI counts, mitochondrial
  percentage) — use Cell_qc instead of Cell_filtering
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MUSIC
  role: Provides the Cell_filtering function that removes invalid-edit cells and filters
    perturbations by cell count thresholds
  repo: github.com/bm2-lab/MUSIC
- name: SAVER
  role: Performs the preceding Data_imputation step whose output feeds Cell_filtering
  repo: github.com/mohuangx/SAVER
- name: R
  role: Runtime environment for MUSIC (requires R>=3.4.1) in which Cell_filtering
    is executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-10216-x
    title: Model-based understanding of single-cell CRISPR screening
  source_content_hash: sha256:46a8d43e8a82ea31104938e8d6a17dd53bf4950fcec92d212ac52e6f2db5d3cc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MUSIC
  - R
  - SAVER
  - devtools
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# perturbed-cell-filtering-and-invalid-edit-removal

## Summary

MUSIC's Cell_filtering step removes perturbed cells with invalid edits and drops perturbations that lose too many cells, so downstream topic modeling is built only from cells whose perturbation is credible. It relies on comparing each perturbed cell's differential expression profile against control cells and same-perturbation peers.

## When to use

Use when preprocessing pooled CRISPR screening + scRNA-seq data (e.g. crop_stimulated Perturb-seq input) before topic modeling, to remove perturbed cells whose DE profile resembles controls and perturbations with too few remaining cells.

## When NOT to use

- When running an imputation/filtering ablation that requires skipping Cell_filtering to compare overall perturbation effect rankings against the full pipeline
- When input is raw fastq or 10X matrix files not yet processed by Input_preprocess/Input_preprocess_10X into expression_profile + perturb_information
- When only cell quality control is needed (genes detected, UMI counts, mitochondrial percentage) — use Cell_qc instead of Cell_filtering

## Inputs

- Imputed expression profile matrix (genes x cells) from Data_imputation
- Named character vector perturb_information mapping cell barcodes to sgRNA/target labels with control labelled 'CTRL'
- Parameters: cell_num_threshold=20, cpu_num for parallel computation

## Outputs

- Filtered expression profile and perturb_information (crop_seq_filtered) containing only cells with valid edits
- Invalid-rate diagnostics plot (Invalid_rate.png) when plot=TRUE
- Perturbation set pruned of all-zero, >=90%-filtered, and <30-cell perturbations

## How to apply

After Cell_qc and optional SAVER-based Data_imputation, run MUSIC's Cell_filtering with cell_num_threshold=20 on the imputed expression profile and perturb_information. For each perturbation, MUSIC compares the differentially expressed gene profile of each perturbed cell: if a perturbed cell is more similar to control cells than to other cells carrying the same perturbation, it is filtered as an invalid edit. Perturbations whose gene expression values are all zero across cells are dropped directly, perturbations where filtered cells reach a high proportion (default 90%) are removed, and perturbations with fewer than 30 perturbed cells are excluded from further analysis. Filtering is skippable in ablation settings but is part of the default MUSIC preprocessing that feeds Get_high_varGenes, Get_topics, and Rank_overall.

## Related tools

- **MUSIC** (Provides the Cell_filtering function that removes invalid-edit cells and filters perturbations by cell count thresholds) — github.com/bm2-lab/MUSIC
- **SAVER** (Performs the preceding Data_imputation step whose output feeds Cell_filtering) — github.com/mohuangx/SAVER
- **R** (Runtime environment for MUSIC (requires R>=3.4.1) in which Cell_filtering is executed)

## Examples

```
crop_seq_filtered<-Cell_filtering(crop_seq_imputation$expression_profile,crop_seq_imputation$perturb_information,cell_num_threshold=20,plot=T,cpu_num=30)
```

## Evaluation signals

- Invalid_rate plot is produced when plot=TRUE, showing the fraction of cells filtered per perturbation
- No perturbation retained with fewer than cell_num_threshold=20 cells after filtering
- Perturbations with all-zero gene expression across cells, >=90% filtered cells, or <30 perturbed cells are absent from the filtered output
- Filtered object (crop_seq_filtered) carries matched expression_profile and perturb_information with consistent cell barcodes
- In ablation comparisons, the full pipeline (imputation + filtering) should produce a reference ranking with ~0.6 average Pearson similarity to ablation rankings, with the combination ablation differing most

## Limitations

- Cell filtering may take a long time without parallel computation; use cpu_num for speed
- The authors acknowledge that current filtering steps for data noise may need future refinement to become more effective
- Filtering requires the control sample label to be exactly 'CTRL' in perturb_information

## Evidence

- [readme] Invalid-edit cell filtering: "crop_seq_filtered<-Cell_filtering(crop_seq_imputation$expression_profile,crop_seq_imputation$perturb_information,cell_num_threshold=20,plot=T,cpu_num=30)"
- [discussion] Similarity-based invalid edit detection: "if the differentially expressed gene profile of a perturbed cell is more similar to the control cells than that of other same perturbed cells, this cell will be filtered"
- [discussion] Perturbation-level filtering thresholds: "if the influenced cells filtered are amount to a high proportion (default 90%) among all, such perturbation is filtered"
- [discussion] Minimum perturbed cell count: "the perturbations with perturbed cells lower than 30 (default) are not considered in MUSIC"
- [readme] Control label requirement: "The label of the control sample needs to be "CTRL"."
