---
name: cross-cell-type-rhythmic-gene-overlap
description: Use when comparing circadian rhythmicity between annotated cell types
  in single-cell RNAseq time-course data, e.g., counting rhythmic genes per cell type
  and their shared overlap.
when_to_use_negative:
- You need circadian phase/timestamp prediction for individual samples rather than
  rhythmic-gene detection per cell type — use tauFisher's train/test functions instead
- Input is raw single-cell expression without aggregation — JTK_Cycle cannot handle
  single-cell dropout; pseudobulk aggregation per cell type and time point is required
- The data lack circadian time-point structure (no ordered time series per cell type),
  since meta2d requires columns ordered by circadian time
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: tauFisher
  role: Provides run_metacycle and find_periodic_genes functions to run JTK_Cycle
    on pseudobulk matrices and extract rhythmic genes
  repo: https://github.com/micnngo/tauFisher
- name: MetaCycle
  role: Executes meta2d/JTK_Cycle rhythmicity testing on each cell-type pseudobulk
    time series
- name: dplyr
  role: R data manipulation for building pseudobulk matrices, normalizing, and intersecting
    gene lists
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-48041-6
    title: tauFisher predicts circadian time from a single sample of bulk and single-cell
      pseudobulk transcriptomic data
  source_content_hash: sha256:875f073bf6d8ecc25e96fe031867368bec11fa00fdfe92e9151152c17ba0fb9a
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cross-cell-type-rhythmic-gene-overlap

## Summary

Detect cell-type-specific circadian rhythmic genes by running JTK_Cycle (via MetaCycle meta2d through tauFisher) on normalized per-cell-type pseudobulk scRNAseq time series, then intersecting the rhythmic gene lists across cell types.

## When to use

Use when comparing circadian rhythmicity between annotated cell types in single-cell RNAseq time-course data, e.g., counting rhythmic genes per cell type and their shared overlap.

## When NOT to use

- You need circadian phase/timestamp prediction for individual samples rather than rhythmic-gene detection per cell type — use tauFisher's train/test functions instead
- Input is raw single-cell expression without aggregation — JTK_Cycle cannot handle single-cell dropout; pseudobulk aggregation per cell type and time point is required
- The data lack circadian time-point structure (no ordered time series per cell type), since meta2d requires columns ordered by circadian time

## Inputs

- Raw UMI count matrix from scRNAseq with cell-type annotations and circadian time-point metadata per sample
- Per-cell-type pseudobulk count matrices (cell types x circadian time points), e.g., fibroblast and immune cell series

## Outputs

- List of rhythmic genes per cell type (JTK_pvalue < 0.05), e.g., rhythmic_genes_fibroblast_immune.csv containing fibroblast, immune-cell, and shared-gene lists
- meta2d/JTK_Cycle output files per cell-type pseudobulk series

## How to apply

Aggregate raw UMI counts over all cells of each annotated cell type (e.g., dermal fibroblasts and immune cells) within each sample/circadian time point to build per-cell-type pseudobulk count matrices. Normalize each pseudobulk sample by total read count, scaling to counts per 10,000 (CP10K). Format each cell-type matrix for meta2d with columns ordered by circadian time, inserting NA columns for missing time points as required by JTK_Cycle. Run JTK_Cycle through MetaCycle meta2d via tauFisher's run_metacycle on each cell-type series, extract genes with JTK_pvalue < 0.05 using find_periodic_genes, and intersect the resulting lists. For the dermal skin dataset this reproduces 1946 rhythmic genes in fibroblasts, 432 in immune cells, and 79 shared genes (mostly core clock and metabolism).

## Related tools

- **tauFisher** (Provides run_metacycle and find_periodic_genes functions to run JTK_Cycle on pseudobulk matrices and extract rhythmic genes) — https://github.com/micnngo/tauFisher
- **MetaCycle** (Executes meta2d/JTK_Cycle rhythmicity testing on each cell-type pseudobulk time series)
- **dplyr** (R data manipulation for building pseudobulk matrices, normalizing, and intersecting gene lists)

## Examples

```
genes <- find_periodic_genes(input_file=meta2d_file, p_value_cutoff=0.05); rhythmic_fb <- genes$JTK; shared <- intersect(rhythmic_fb, rhythmic_immune)
```

## Evaluation signals

- Expected gene counts reproduced: 1946 rhythmic genes in dermal fibroblasts, 432 in dermal immune cells
- Shared-gene intersection size equals 79, with shared genes enriched for the core clock network and metabolism
- All genes in each rhythmic list satisfy JTK_pvalue < 0.05 from meta2d output
- Pseudobulk matrices contain NA columns inserted for missing time points and columns ordered by circadian time, as required by JTK_Cycle
- Pseudobulk normalization verified as CP10K (scaled to counts per 10,000 per sample)

## Limitations

- tauFisher cannot overcome the high sequencing dropout rate in scRNAseq and thus cannot predict circadian time at a single-cell level — rhythmicity calls must be made on pseudobulk, not single cells
- Greater circadian phase heterogeneity in dermal immune cells may dampen collective core clock gene rhythm and reduce detected rhythmic genes, so cross-cell-type counts are confounded by phase heterogeneity
- Claim of phase heterogeneity differences between dermal fibroblasts and immune cells relies only on computational analysis without experimental validation

## Evidence

- [results] JTK_Cycle on normalized pseudobulk data identified 1946 rhythmic genes in dermal fibroblasts and 432 in dermal immune cells, with only 79 genes rhythmic in both cell types: [evidence span withheld — anchor status: not_found]
- [discussion] Genes with JTK_pvalue < 0.05 were determined to be significantly rhythmic.: "Genes with JTK_pvalue < 0.05 were determined to be significantly rhythmic."
- [methods] genes = find_periodic_genes(input_file=meta2d_file,: "run_metacycle(df=bulk_adj, 
              timepoints=time_adj, 
              out_file="../inst/extdata/GSE157077_full_adj.txt","
- [readme] tauFisher requires R >=3.5.0: [evidence span withheld — anchor status: external_doc]
- [discussion] tauFisher cannot predict circadian time at a single cell level due to scRNAseq dropout: "it cannot overcome the high sequencing dropout rate in scRNAseq and thus cannot predict circadian time at a single cell level"
