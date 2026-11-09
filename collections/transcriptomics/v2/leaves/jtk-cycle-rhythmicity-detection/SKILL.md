---
name: jtk-cycle-rhythmicity-detection
description: Use when you have normalized pseudobulk expression matrices across circadian
  time points per cell type and need to identify genes with significant 24-hour periodicity
  (JTK_pvalue < 0.05) and compare rhythmic gene sets between cell types.
when_to_use_negative:
- Input is single-cell level expression with high sequencing dropout — tauFisher's
  pipeline cannot predict circadian time at the single-cell level; use pseudobulk
  aggregation first
- The goal is to predict circadian time of an unlabeled sample rather than detect
  rhythmic genes — use tauFisher's train_tauFisher/test_tauFisher instead
- Time points lack ordering or the series is not sampled across the circadian cycle
  — JTK_Cycle requires time-ordered columns with NA placeholders for missing time
  points
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: tauFisher
  role: Provides run_metacycle and find_periodic_genes wrappers to run meta2d/JTK_Cycle
    and extract periodic genes
  repo: https://github.com/micnngo/tauFisher
- name: MetaCycle
  role: Runs the meta2d analysis that executes JTK_Cycle on the pseudobulk time series
- name: dplyr
  role: Used for intersecting rhythmic gene lists across cell types and writing results
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-48041-6
    title: tauFisher predicts circadian time from a single sample of bulk and single-cell
      pseudobulk transcriptomic data
  source_content_hash: sha256:3dbc82cf8b7e9cf830dcf164c6ffcbbf27d1db500b0c7a9bc8f7a9fd68bc3ae3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/micnngo/tauFisher
  tools:
  - MetaCycle
  - dplyr
  - tauFisher
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# jtk-cycle-rhythmicity-detection

## Summary

Detect rhythmic genes in cell-type pseudobulk circadian time-series by running JTK_Cycle through MetaCycle's meta2d, then intersecting rhythmic gene lists across cell types. Grounded in the tauFisher paper's dermal fibroblast vs immune cell analysis (1946 vs 432 rhythmic genes, 79 shared).

## When to use

Use when you have normalized pseudobulk expression matrices across circadian time points per cell type and need to identify genes with significant 24-hour periodicity (JTK_pvalue < 0.05) and compare rhythmic gene sets between cell types.

## When NOT to use

- Input is single-cell level expression with high sequencing dropout — tauFisher's pipeline cannot predict circadian time at the single-cell level; use pseudobulk aggregation first
- The goal is to predict circadian time of an unlabeled sample rather than detect rhythmic genes — use tauFisher's train_tauFisher/test_tauFisher instead
- Time points lack ordering or the series is not sampled across the circadian cycle — JTK_Cycle requires time-ordered columns with NA placeholders for missing time points

## Inputs

- Raw UMI count matrix from scRNAseq with cell-type annotations and circadian time-point metadata per sample
- Per-cell-type pseudobulk count matrix (genes x circadian time points), formatted for meta2d with time-ordered columns and NA columns for missing time points

## Outputs

- List of rhythmic genes per cell type (JTK_pvalue < 0.05), e.g., 1946 in dermal fibroblasts and 432 in dermal immune cells
- Intersection of rhythmic gene lists across cell types (e.g., 79 shared genes)
- CSV file of fibroblast, immune-cell, and shared rhythmic gene lists (rhythmic_genes_fibroblast_immune.csv)

## How to apply

Sum raw UMI counts over all cells of each annotated cell type (e.g., fibroblasts and immune cells) within each sample/circadian time point to build per-cell-type pseudobulk count matrices. Normalize each pseudobulk sample by its total read count, scaling to counts per 10,000 (CP10K). Format each cell-type matrix for meta2d with columns ordered by circadian time, inserting NA columns for missing time points as required by JTK_Cycle. Run JTK_Cycle via tauFisher's run_metacycle (which calls MetaCycle meta2d) on each cell-type series, then use find_periodic_genes to extract genes with JTK_pvalue < 0.05. Finally, intersect the two rhythmic gene lists with dplyr to identify shared rhythmic genes, which in the paper were enriched for the core clock network and metabolism.

## Related tools

- **tauFisher** (Provides run_metacycle and find_periodic_genes wrappers to run meta2d/JTK_Cycle and extract periodic genes) — https://github.com/micnngo/tauFisher
- **MetaCycle** (Runs the meta2d analysis that executes JTK_Cycle on the pseudobulk time series)
- **dplyr** (Used for intersecting rhythmic gene lists across cell types and writing results)

## Examples

```
genes_fb <- find_periodic_genes(input_file=run_metacycle(df=fb_pseudobulk_cp10k, timepoints=time_adj), pvalue_col='JTK_pvalue', threshold=0.05)
```

## Evaluation signals

- Reproduce the published counts: 1946 rhythmic genes in dermal fibroblasts and 432 in dermal immune cells from normalized pseudobulk scRNAseq
- The shared-gene intersection equals 79 genes, predominantly annotated to the core clock network and metabolism
- All extracted genes pass the JTK_pvalue < 0.05 threshold from the meta2d output of find_periodic_genes
- meta2d input matrices contain NA columns for missing circadian time points and columns strictly ordered by circadian time
- Output CSV (rhythmic_genes_fibroblast_immune.csv) contains three consistent lists: fibroblast, immune, and shared gene sets with |fibroblast ∩ immune| equal to shared list size

## Limitations

- tauFisher cannot overcome the high sequencing dropout rate in scRNAseq and thus cannot predict circadian time at a single cell level — pseudobulk aggregation is required
- tauFisher's performance on human blood samples is subpar compared to mouse benchmark datasets
- Claims on differences in phase heterogeneity between dermal fibroblasts and immune cells rely only on computational analysis without experimental validation

## Evidence

- [discussion] Genes with JTK_pvalue < 0.05 were determined to be significantly rhythmic.: "Genes with JTK_pvalue < 0.05 were determined to be significantly rhythmic."
- [results] Pseudobulk workflow: [evidence span withheld — anchor status: not_found]
- [methods] run_metacycle usage: "run_metacycle(df=bulk_adj, timepoints=time_adj, out_file="../inst/extdata/GSE157077_full_adj.txt""
- [methods] find_periodic_genes usage: "genes = find_periodic_genes(input_file=meta2d_file,"
- [discussion] Single-cell limitation: "it cannot overcome the high sequencing dropout rate in scRNAseq and thus cannot predict circadian time at a single cell level"
- [readme] README dependency list: [evidence span withheld — anchor status: not_found]
