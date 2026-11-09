---
name: tsne-dimensionality-reduction-of-scrna-seq
description: Use when you have normalized scRNA-seq count data (e.g. 10x Genomics
  PBMC, 4500 cells, 9 immune cell types) and need a 2D embedding to assess whether
  imputation or preprocessing improves subpopulation separation.
when_to_use_negative:
- Input is not normalized scRNA-seq count data or lacks cell-type labels needed to
  color and assess cluster separation
- The analysis goal is a numeric clustering metric or differential expression testing
  rather than qualitative 2D visualization of subpopulation separation
- The dataset requires quantitative, threshold-based validation — this paper reports
  only qualitative t-SNE agreement with no numeric threshold
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scImpute
  role: Produces the imputed count matrix (drop_thre=0.5, labeled mode) whose t-SNE
    embedding is compared against the raw-data embedding to show improved T-cell subpopulation
    separation
  repo: https://github.com/Vivianstats/scImpute
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-03405-7
    title: An accurate and robust imputation method scImpute for single-cell RNA-seq
      data
  source_content_hash: sha256:f9664e6f8572eec8d631fd00fd13bb352b1c885df6130a377a3a9375f1905882
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scImpute
  tools:
  - scImpute
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# tsne-dimensionality-reduction-of-scrna-seq

## Summary

Apply t-SNE dimension reduction to raw and scImpute-imputed scRNA-seq count matrices to visualize separation of immune cell subpopulations, e.g. resolving cytotoxic vs naive cytotoxic and naive vs memory T cells in the 10x Genomics PBMC dataset.

## When to use

Use when you have normalized scRNA-seq count data (e.g. 10x Genomics PBMC, 4500 cells, 9 immune cell types) and need a 2D embedding to assess whether imputation or preprocessing improves subpopulation separation.

## When NOT to use

- Input is not normalized scRNA-seq count data or lacks cell-type labels needed to color and assess cluster separation
- The analysis goal is a numeric clustering metric or differential expression testing rather than qualitative 2D visualization of subpopulation separation
- The dataset requires quantitative, threshold-based validation — this paper reports only qualitative t-SNE agreement with no numeric threshold

## Inputs

- Raw 10x Genomics PBMC count matrix (rows = genes, columns = cells, ~4500 cells)
- scImpute-imputed count matrix (scImpute_count.csv from out_dir)
- Character vector of cell-type labels for the 9 immune cell types
- Marker-gene expression vectors (e.g. S100A8, FCER1A)

## Outputs

- t-SNE embeddings (2D coordinates per cell) for raw and imputed data
- Side-by-side t-SNE plots colored by the 9 cell types (saved as PNG)
- Marker-gene overlay plots (S100A8, FCER1A) on the t-SNE embeddings (saved as PNG)

## How to apply

Normalize the raw and imputed count matrices (imputed matrix produced by scimpute(count_path, infile='csv', outfile='csv', out_dir, labeled=TRUE, drop_thre=0.5, labels=labels, ncores=10), which only imputes values with dropout probability larger than 0.5) and run t-SNE separately on each. Render side-by-side t-SNE plots colored by the 9 cell-type labels to compare separation before and after imputation. Overlay marker genes such as S100A8 and FCER1A on the embeddings to verify monocyte-cluster marker patterns. Judge success qualitatively: cytotoxic vs naive cytotoxic T cells should form two groups, and naive vs memory T cells should become distinguishable from other T cells, as reported in Fig. 6.

## Related tools

- **scImpute** (Produces the imputed count matrix (drop_thre=0.5, labeled mode) whose t-SNE embedding is compared against the raw-data embedding to show improved T-cell subpopulation separation) — https://github.com/Vivianstats/scImpute

## Examples

```
scimpute(count_path, infile='csv', outfile='csv', out_dir, labeled=TRUE, drop_thre=0.5, labels=labels, ncores=10)
```

## Evaluation signals

- Cytotoxic and naive cytotoxic T cells separate into two distinct groups in the imputed-data t-SNE
- Naive and memory T cells become distinguishable from other T cells in the imputed-data t-SNE
- S100A8 and FCER1A overlay plots reproduce the reported monocyte-cluster marker patterns in raw vs imputed data
- Raw and imputed t-SNE runs each start from their own normalized count matrix, and comparison figures are saved as PNG files

## Limitations

- Validation is qualitative only — no numeric threshold is reported in the paper for the t-SNE separation
- Imputation efficiency degrades when dropout rates in the raw data are severely high (droplet-based technologies)
- The choice of parameter K (number of cell populations) in scImpute is left to the user without a definitive selection procedure

## Evidence

- [results] After scImpute imputation, cytotoxic and naive cytotoxic T cells are separated into two groups, and naive and memory T cells become distinguishable from other T cells in the t-SNE result (Fig. 6).: [evidence span withheld — anchor status: not_found]
- [methods] Only the values that have dropout probability larger than drop_thre are imputed by scImpute.: "Only the values that have **dropout probability** larger than `drop_thre` are imputed by `scImpute`."
- [readme] This function returns the column indices of outlier cells, and creates a new file scimpute_count.csv in out_dir to store the imputed count matrix.: "creates a new file `scimpute_count.csv` in `out_dir` to store the imputed count matrix"
- [discussion] another future direction is to improve imputation efficiency further when dropout rates in the raw data are severely high: "another future direction is to improve imputation efficiency further when dropout rates in the raw data are severely high"
