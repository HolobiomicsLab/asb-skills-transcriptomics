---
name: single-cell-reference-based-purity-scoring
description: Use when evaluating a segmentation method's cell-gene matrix for expression
  purity or cross-cell contamination, e.g. measuring B cells expressing T-cell markers
  (CD4, CD8A, CD8B, CD3D, CD3E) in Xenium breast cancer data.
when_to_use_negative:
- The input is raw imaging or transcript coordinates rather than an already-generated
  cell-gene expression matrix — run BIDCell's pipeline (make_cell_gene_mat) first
- No matched single-cell reference or marker definition files exist for the tissue,
  since the purity F1 scores depend on cell-type-specific top/bottom 10 percentile
  markers
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0625
tools:
- name: BIDCell
  role: Produces the segmented cell-gene expression matrix (expr_mat.csv) that is
    scored for purity and contamination
  repo: https://github.com/SydneyBioX/BIDCell
- name: 10x Genomics Xenium
  role: Platform generating the Xenium-BreastCancer1 spatial data evaluated by this
    purity analysis
- name: TISCH2
  role: Public repository source for the breast cancer single-cell reference used
    to define cell-type markers
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-44560-w
    title: 'BIDCell: Biologically-informed self-supervised learning for segmentation
      of subcellular spatial transcriptomics data'
  source_content_hash: sha256:c985f7748764340ce678265627b2c4595dedfeb5739071c226e3a436cc4673f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/BIDCell
  tools:
  - 10x Genomics Xenium
  - BIDCell
  - TISCH2
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# single-cell-reference-based-purity-scoring

## Summary

Quantifies segmentation purity and neighbour contamination by scoring BIDCell-segmented cells against a single-cell reference (TISCH-BRCA sc_breast.csv) using positive/negative marker F1 scores and distance-stratified expression of negative markers in B cells (Fig. 3a-e).

## When to use

Use when evaluating a segmentation method's cell-gene matrix for expression purity or cross-cell contamination, e.g. measuring B cells expressing T-cell markers (CD4, CD8A, CD8B, CD3D, CD3E) in Xenium breast cancer data.

## When NOT to use

- The input is raw imaging or transcript coordinates rather than an already-generated cell-gene expression matrix — run BIDCell's pipeline (make_cell_gene_mat) first
- No matched single-cell reference or marker definition files exist for the tissue, since the purity F1 scores depend on cell-type-specific top/bottom 10 percentile markers

## Inputs

- BIDCell cell-gene expression matrix (expr_mat.csv) from Xenium-BreastCancer1 segmentation
- Breast cancer single-cell reference (TISCH-BRCA sc_breast.csv)
- Positive marker file (sc_breast_markers_pos.csv)
- Negative marker file (sc_breast_markers_neg.csv)
- Paired Chromium scFFPE-seq average expression data for correlation

## Outputs

- Correlation metric between segmented and scFFPE-seq average expression
- Positive and negative purity F1 scores per evaluation category
- Distance-stratified percentages of B cells expressing T-cell markers
- fig3_evaluation_metrics.csv

## How to apply

Load the BIDCell cell-gene expression matrix (expr_mat.csv) with pandas alongside the breast cancer single-cell reference (TISCH-BRCA sc_breast.csv) and marker files (sc_breast_markers_pos.csv, sc_breast_markers_neg.csv). First compute the correlation between average gene expression of BIDCell-segmented cells and average expression from the paired Chromium scFFPE-seq data (Fig. 3a-d). Compute positive and negative purity F1 scores using markers defined as genes in the highest and lowest 10 expression percentile per cell type, excluding positive markers common to at least a third of cell types. Identify B cells in the segmentation, stratify them by distance to the nearest T cell, and compute the percentage of B cells expressing CD4, CD8A, CD8B, CD3D, and CD3E per distance bin (Fig. 3e). Write correlation, purity F1 scores, and distance-stratified percentages to fig3_evaluation_metrics.csv.

## Related tools

- **BIDCell** (Produces the segmented cell-gene expression matrix (expr_mat.csv) that is scored for purity and contamination) — https://github.com/SydneyBioX/BIDCell
- **10x Genomics Xenium** (Platform generating the Xenium-BreastCancer1 spatial data evaluated by this purity analysis)
- **TISCH2** (Public repository source for the breast cancer single-cell reference used to define cell-type markers)

## Examples

```
import pandas as pd; expr = pd.read_csv('expr_mat.csv'); markers_pos = pd.read_csv('sc_breast_markers_pos.csv'); markers_neg = pd.read_csv('sc_breast_markers_neg.csv')
```

## Evaluation signals

- Correlation and purity F1 values are consistent across evaluation categories as shown in Fig. 3a-d
- Distance-dependent T-marker expression in B cells is reproduced, as shown in Fig. 3e
- Positive/negative markers match the definition: highest and lowest 10 percentile expression per cell type, with positive markers common to at least a third of cell types excluded
- All computed metrics (correlation, purity F1, distance-stratified percentages) are present in fig3_evaluation_metrics.csv with no missing bins

## Limitations

- Only <1,000 genes are needed for segmentation, so marker panels are limited relative to full transcriptomes
- Using a larger number of positive markers tends to increase the size of predicted cells, affecting purity interpretation
- Two-dimensional segmentation may achieve limited representation of cells' three-dimensional structure, affecting measured contamination

## Evidence

- [readme] Positive and negative markers were those with expressions in the highest and lowest 10 percentile for each cell type of a tissue sample.: "The positive and negative markers were those with expressions in the highest and lowest 10 percentile for each cell type of a tissue sample."
- [readme] We found that removing positive markers that were common to at least a third of cell types in each dataset was appropriate across various datasets.: "We found that removing positive markers that were common to at least a third of cell types in each dataset was appropriate across various datasets."
- [methods] Please see the provided breast cancer single-cell reference and positive/negative marker files (`sc_breast.csv`, `sc_breast_markers_pos.csv`, and `sc_breast_markers_neg.csv`): "Please see the provided breast cancer single-cell reference and positive/negative marker files (`sc_breast.csv`, `sc_breast_markers_pos.csv`, and `sc_breast_markers_neg.csv`)"
