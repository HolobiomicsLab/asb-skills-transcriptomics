---
name: library-size-normalization-cp10k
description: Use when raw scRNAseq UMI counts must be summed over cells of each annotated
  cell type per sample/circadian time point and scaled to CP10K before running JTK_Cycle
  on pseudobulk series.
when_to_use_negative:
- Input is already normalized expression (e.g., SCTransform-normalized or log-transformed
  matrices) rather than raw UMI counts — CP10K re-scaling of normalized values double-corrects
  library size
- Data is a single-cell-level prediction task — the article notes tauFisher 'cannot
  overcome the high sequencing dropout rate in scRNAseq and thus cannot predict circadian
  time at a single cell level', so pseudobulk aggregation is required first
- No cell-type annotations or circadian time points are available to define pseudobulk
  aggregation groups
tools:
- name: tauFisher
  role: Provides run_metacycle and find_periodic_genes functions used to run JTK_Cycle
    on the CP10K-normalized pseudobulk matrices and extract JTK_pvalue < 0.05 rhythmic
    genes
  repo: https://github.com/micnngo/tauFisher
- name: MetaCycle
  role: Runs meta2d/JTK_Cycle on the CP10K-normalized pseudobulk time series
- name: dplyr
  role: R package dependency used in the tauFisher pipeline for summing counts over
    cell-type groups and reshaping the pseudobulk matrices
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-48041-6
    title: tauFisher predicts circadian time from a single sample of bulk and single-cell
      pseudobulk transcriptomic data
  source_content_hash: sha256:e74008d5a69b11a188555db2694dde154eee8eb5252851519c28ab4c9af1e1a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/micnngo/tauFisher
  tools:
  - MetaCycle
  - dplyr
  - tauFisher
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# library-size-normalization-cp10k

## Summary

Aggregates raw UMI counts to per-cell-type pseudobulk matrices per circadian time point and library-size normalizes each pseudobulk sample to counts per 10,000 (CP10K), preparing scRNAseq-derived series for JTK_Cycle rhythm detection with MetaCycle via tauFisher.

## When to use

Use when raw scRNAseq UMI counts must be summed over cells of each annotated cell type per sample/circadian time point and scaled to CP10K before running JTK_Cycle on pseudobulk series.

## When NOT to use

- Input is already normalized expression (e.g., SCTransform-normalized or log-transformed matrices) rather than raw UMI counts — CP10K re-scaling of normalized values double-corrects library size
- Data is a single-cell-level prediction task — the article notes tauFisher 'cannot overcome the high sequencing dropout rate in scRNAseq and thus cannot predict circadian time at a single cell level', so pseudobulk aggregation is required first
- No cell-type annotations or circadian time points are available to define pseudobulk aggregation groups

## Inputs

- Raw UMI count matrix from scRNAseq (CellRanger MM10 output) with per-cell cell-type annotations (e.g., fibroblasts, immune cells)
- Sample metadata mapping cells to sample/circadian time point

## Outputs

- Per-cell-type pseudobulk CP10K-normalized expression matrices with time-ordered columns (NA columns for missing time points)
- Rhythmic gene lists per cell type and their intersection, written to rhythmic_genes_fibroblast_immune.csv

## How to apply

Sum raw UMI counts over all cells of each annotated cell type (e.g., dermal fibroblasts and dermal immune cells) within each sample/circadian time point to build per-cell-type pseudobulk count matrices. Normalize each pseudobulk sample by dividing its counts by the sample's total read count and scaling to counts per 10,000 (CP10K). Format each cell-type pseudobulk matrix for meta2d with columns ordered by circadian time, inserting NA columns for any missing time points as required by JTK_Cycle. Run JTK_Cycle through MetaCycle meta2d via tauFisher's run_metacycle and extract significant genes with find_periodic_genes using the JTK_pvalue < 0.05 cutoff; for the mouse dermal skin data this yields 1946 rhythmic genes in fibroblasts, 432 in immune cells, and 79 shared.

## Related tools

- **tauFisher** (Provides run_metacycle and find_periodic_genes functions used to run JTK_Cycle on the CP10K-normalized pseudobulk matrices and extract JTK_pvalue < 0.05 rhythmic genes) — https://github.com/micnngo/tauFisher
- **MetaCycle** (Runs meta2d/JTK_Cycle on the CP10K-normalized pseudobulk time series)
- **dplyr** (R package dependency used in the tauFisher pipeline for summing counts over cell-type groups and reshaping the pseudobulk matrices)

## Examples

```
pbuk <- rowsum(counts, group = paste(cell_type, timepoint, sep = "_")); pbuk_cp10k <- sweep(t(t(pbuk)/colSums(pbuk)), 1, `*`, 10000)
```

## Evaluation signals

- Each CP10K-normalized pseudobulk sample column sums to exactly 10,000
- Reproducing the workflow on the mouse dermal skin data yields 1946 rhythmic fibroblast genes, 432 rhythmic immune-cell genes (JTK_pvalue < 0.05), and 79 shared genes
- Pseudobulk matrices contain NA columns for any missing circadian time points, with columns ordered by time as required by JTK_Cycle
- The shared-gene intersection is dominated by core clock network and metabolism genes
- Output file rhythmic_genes_fibroblast_immune.csv contains the fibroblast list, immune-cell list, and shared-gene intersection

## Limitations

- The CP10K pipeline is only a preparatory step; the downstream tauFisher/JTK_Cycle analysis cannot operate at single-cell resolution due to high scRNAseq dropout
- Claims about cell-type rhythm differences (e.g., dampened rhythms in immune cells) rest on computational analysis without experimental validation
- taufisher's performance on human blood samples is subpar, so the pseudobulk-to-JTK_Cycle workflow may not transfer to all human tissues

## Evidence

- [methods] Normalize each pseudobulk sample by its total read count, scaling to counts per 10,000 (CP10K).: [evidence span withheld — anchor status: not_found]
- [discussion] Genes with JTK_pvalue < 0.05 were determined to be significantly rhythmic.: "Genes with JTK_pvalue < 0.05 were determined to be significantly rhythmic."
- [results] 1946 rhythmic genes in dermal fibroblasts and 432 in dermal immune cells, with only 79 genes rhythmic in both cell types: [evidence span withheld — anchor status: not_found]
- [readme] tauFisher requires R >=3.5.0 and has been tested on R 4.0.5.: [evidence span withheld — anchor status: external_doc]
