---
name: immune-cell-subpopulation-clustering-visualization
description: Use when raw droplet-based scRNA-seq count data (e.g. 10x Genomics PBMC)
  merges closely related cell types such as cytotoxic/naive cytotoxic or naive/memory
  T cells in clustering or t-SNE plots, and dropout imputation should resolve them.
when_to_use_negative:
- The goal is to smooth or transform all expression values (e.g. pseudotime-style
  denoising) — scImpute deliberately imputes only values with dropout probability
  above drop_thre and leaves the rest unchanged
- Cell-type labels are unavailable and the user cannot estimate the number of subpopulations
  — the labeled=TRUE workflow with supplied labels does not apply; the unlabeled path
  requires choosing Kcluster, which the article notes is left to the user without
  a definitive selection procedure
- The input is already normalized or otherwise processed — scImpute's only input should
  be the raw count matrix, applied before normalization, clustering, or dimension
  reduction
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scImpute
  role: Imputes dropout values in the raw PBMC count matrix (drop_thre=0.5, labeled=TRUE)
    before t-SNE clustering visualization
  repo: https://github.com/Vivianstats/scImpute
- name: t-SNE
  role: Dimension-reduction embedding of raw and imputed matrices used to visually
    assess separation of T-cell subpopulations
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-03405-7
    title: An accurate and robust imputation method scImpute for single-cell RNA-seq
      data
  source_content_hash: sha256:10809989d43abf7380f534b5d5a2a9b1d8bfa3b328a96c3a87700cfa43f8ee97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scImpute
  tools:
  - scImpute
  - t-SNE
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# immune-cell-subpopulation-clustering-visualization

## Summary

Applies scImpute dropout imputation to a 10x Genomics PBMC scRNA-seq count matrix (4500 cells, 9 immune cell types) and then visualizes raw-vs-imputed t-SNE embeddings colored by cell type to resolve T-cell subpopulations (cytotoxic vs naive cytotoxic, naive vs memory).

## When to use

Use when raw droplet-based scRNA-seq count data (e.g. 10x Genomics PBMC) merges closely related cell types such as cytotoxic/naive cytotoxic or naive/memory T cells in clustering or t-SNE plots, and dropout imputation should resolve them.

## When NOT to use

- The goal is to smooth or transform all expression values (e.g. pseudotime-style denoising) — scImpute deliberately imputes only values with dropout probability above drop_thre and leaves the rest unchanged
- Cell-type labels are unavailable and the user cannot estimate the number of subpopulations — the labeled=TRUE workflow with supplied labels does not apply; the unlabeled path requires choosing Kcluster, which the article notes is left to the user without a definitive selection procedure
- The input is already normalized or otherwise processed — scImpute's only input should be the raw count matrix, applied before normalization, clustering, or dimension reduction

## Inputs

- Raw 10x Genomics PBMC scRNA-seq count matrix as CSV (rows = genes, columns = cells; whole-genome matrix recommended)
- Character vector of cell-type labels for the 9 immune cell types (used when labeled=TRUE)
- Output directory path for the imputed matrix

## Outputs

- Imputed count matrix written to scImpute_count.csv in out_dir (same dimension as input)
- Column indices of detected outlier cells returned by scimpute()
- Side-by-side t-SNE plots of raw vs imputed data colored by the 9 cell types (saved as PNG)
- t-SNE overlays of S100A8 and FCER1A expression showing monocyte-cluster marker patterns (PNG)

## How to apply

Load the raw 10x Genomics PBMC count matrix (rows = genes, columns = cells) and, because labels are available, run scimpute() with labeled=TRUE, supplying the character vector of the 9 immune-cell-type labels so that Kcluster is inferred rather than user-set; set drop_thre=0.5 so only values with dropout probability larger than 0.5 are imputed, and pass ncores=10 for parallel computation. scImpute writes the imputed matrix to scImpute_count.csv in out_dir and returns the column indices of outlier cells, which are excluded from imputation. Normalize the raw and imputed matrices separately, run t-SNE on each, and render side-by-side plots colored by the 9 cell types. Rationale: scImpute imputes only likely dropouts (unlike MAGIC/SAVER, which alter all expression levels including those unaffected by dropouts, potentially introducing new biases), so biologically meaningful variation is preserved while masked T-cell structure is recovered. Finally, overlay monocyte marker genes S100A8 and FCER1A on the embeddings to confirm that monocyte-cluster marker patterns are consistent with the raw data.

## Related tools

- **scImpute** (Imputes dropout values in the raw PBMC count matrix (drop_thre=0.5, labeled=TRUE) before t-SNE clustering visualization) — https://github.com/Vivianstats/scImpute
- **t-SNE** (Dimension-reduction embedding of raw and imputed matrices used to visually assess separation of T-cell subpopulations)

## Examples

```
scimpute(count_path, infile='csv', outfile='csv', out_dir=out_dir, labeled=TRUE, drop_thre=0.5, labels=labels, ncores=10)
```

## Evaluation signals

- scimpute_count.csv exists in out_dir and has the same genes x cells dimension as the input raw count matrix
- Only entries whose dropout probability exceeded drop_thre=0.5 differ between raw and imputed matrices; all other values are unchanged
- t-SNE of the imputed data shows cytotoxic and naive cytotoxic T cells separated into two groups, and naive vs memory T cells distinguishable from other T cells (Fig. 6 pattern)
- Monocyte markers S100A8 and FCER1A show the reported monocyte-cluster expression patterns on the imputed t-SNE, consistent with raw-data patterns
- Outlier cells detected by scImpute are excluded from imputation (returned column indices match the cells altered least/noise-checked in output)

## Limitations

- Imputation efficiency degrades when dropout rates in the raw data are severely high, as in droplet-based technologies — noted as a future direction
- In the unlabeled setting, the number of cell populations K must be chosen by the user based on clustering of the raw data and desired resolution; no definitive selection procedure is given
- Computation efficiency depends on prior filtering of cells based on biological knowledge, which is not specified; gene filtering is acceptable but most genes should be present for robust dropout identification
- Validation of this PBMC t-SNE result is qualitative — no numeric threshold is reported in the paper, so correctness is judged by visual comparison

## Evidence

- [methods] Only the values that have dropout probability larger than drop_thre are imputed by scImpute.: "Only the values that have **dropout probability** larger than `drop_thre` are imputed by `scImpute`."
- [readme] scimpute output file description: "This function returns the column indices of outlier cells, and creates a new file `scimpute_count.csv` in `out_dir` to store the imputed count matrix."
- [abstract] scImpute selectivity rationale: "scImpute automatically identifies likely dropouts, and only perform imputation on these values without introducing new biases to the rest data."
- [intro] contrast with MAGIC/SAVER: "Both MAGIC and SAVER would alter all gene expression levels including those unaffected by dropouts"
- [readme] recommended input and applicability: "`scImpute` can be applied to raw read count matrix before the users perform downstream analyses such as"
