---
name: single-cell-copy-number-baseline-subtraction
description: Use when you have a raw scRNA-seq count matrix (genes on rows, cells
  on columns) and need to remove the normal-cell expression baseline so that cell-specific
  copy number signal can be segmented, without manually identifying reference cell
  clusters as inferCNV requires.
when_to_use_negative:
- The tumor type has minimal copy number alterations (e.g., leukemia, pediatric cancers,
  Ependymomas) — the aneuploidy-based baseline/CNV approach may not be suited in this
  case
- You already have a trusted reference profile or matched normal cells and only need
  clonal structure with a fixed reference — pass norm_cells with FIXED_NORMAL_CELLS=TRUE
  rather than re-estimating a baseline
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0121
tools:
- name: SCEVAN
  role: R package that identifies confident normal cells, estimates the copy number
    baseline (median of confident normal cells), and subtracts it to build the relative
    expression matrix (via getConfidentNormalCells / pipelineCNA)
  repo: github.com/AntonioDeFalco/SCEVAN
- name: yaGST
  role: Mann–Whitney–Wilcoxon single-sample gene set test used to score each cell
    against normal-cell signatures for confident normal cell identification (P < 1e-10,
    NES > 1.0)
  repo: github.com/miccec/yaGST
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:9e816580a35ba5bd5fb6ab42c49cf71b3f620ee1083fc245c6906b2f5dff7543
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCEVAN
  - yaGST
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# single-cell-copy-number-baseline-subtraction

## Summary

Estimates a copy number baseline from confidently normal cells in a scRNA-seq count matrix (median expression of cells passing yaGST significance thresholds, P < 1e-10 and NES > 1.0) and subtracts it to produce the relative expression matrix that SCEVAN smooths and segments for clonal CNV detection.

## When to use

Use when you have a raw scRNA-seq count matrix (genes on rows, cells on columns) and need to remove the normal-cell expression baseline so that cell-specific copy number signal can be segmented, without manually identifying reference cell clusters as inferCNV requires.

## When NOT to use

- The tumor type has minimal copy number alterations (e.g., leukemia, pediatric cancers, Ependymomas) — the aneuploidy-based baseline/CNV approach may not be suited in this case
- You already have a trusted reference profile or matched normal cells and only need clonal structure with a fixed reference — pass norm_cells with FIXED_NORMAL_CELLS=TRUE rather than re-estimating a baseline

## Inputs

- Raw scRNA-seq count matrix (genes × cells; Gene Symbol or Ensembl ID allowed)
- Normal-cell gene signatures (SCEVANsignatures, Supplementary Data 1), optionally extended with AdditionalGeneSets
- Optional: user-provided vector of known normal cells (norm_cells, with FIXED_NORMAL_CELLS=TRUE)

## Outputs

- Relative expression matrix (count matrix minus the estimated copy number baseline), ordered by genomic coordinates
- Identified confident normal cell set (P < 1e-10, NES > 1.0)

## How to apply

First filter cells with fewer than 200 detected genes, genes expressed in less than 1% of cells, and REACTOME cell-cycle pathway genes, annotate genes with Ensembl-based genomic locations and sort by genomic coordinates. Run SCEVAN::getConfidentNormalCells, which tests each cell against normal cell-type gene signatures (Supplementary Data 1) with the yaGST Mann–Whitney–Wilcoxon single-sample gene set test; take the top classified cells with P < 1e-10 and NES > 1.0 (at most 30 cells) as confident normal cells. Compute the per-gene median expression of these confident normal cells as the copy number baseline and subtract it from the count matrix to obtain the relative expression matrix, on which the edge-preserving logcosh TV smoothing and greedy Mumford–Shah segmentation (beta_vega, default 0.5) are subsequently applied. If no confident normal cells are found, assume the sample is pure malignant and remove a synthetic gaussian baseline instead. If user-provided normal cells are available, pass them via norm_cells with FIXED_NORMAL_CELLS=TRUE to use a fixed reference.

## Related tools

- **SCEVAN** (R package that identifies confident normal cells, estimates the copy number baseline (median of confident normal cells), and subtracts it to build the relative expression matrix (via getConfidentNormalCells / pipelineCNA)) — github.com/AntonioDeFalco/SCEVAN
- **yaGST** (Mann–Whitney–Wilcoxon single-sample gene set test used to score each cell against normal-cell signatures for confident normal cell identification (P < 1e-10, NES > 1.0)) — github.com/miccec/yaGST

## Examples

```
results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)
```

## Evaluation signals

- Confident normal cells must satisfy P < 1e-10 and NES > 1.0 (maximum 30 cells); if none pass, the synthetic gaussian fallback must have been triggered instead of a data-derived baseline
- The relative expression matrix must have the same gene × cell dimensions as the (filtered, genomically sorted) input count matrix
- Downstream check: after smoothing and segmentation, the matrix should be piecewise constant along the genomic axis with retained breakpoints, confirming baseline subtraction did not destroy CN structure
- Run on the MGH106 sample (GSE131928), the result should reproduce SCEVAN's malignant/non-malignant segregation and clonal substructure reported for that sample

## Limitations

- If no confident normal cells are found, the sample is assumed pure and a synthetic gaussian baseline is removed, which may misestimate the baseline in mixed samples where signature tests fail
- CopyKAT-style classification can be affected by wrong identification of normal cells; the baseline quality depends on the confident normal cell set
- The approach assumes aneuploidy; it may not suit tumors with minimal genomic alterations (leukemia, pediatric cancers, Ependymomas)

## Evidence

- [discussion] baseline subtraction via median of normal cells: "the copy number baseline, estimated from the median expression of confident normal cells, is removed from the count matrix, thus obtaining the relative matrix"
- [discussion] confident normal cell thresholds: "assume as normal confident cells the top classified cells with P value less than 10−10 and Normalized Enrichment Score (NES) greater than 1.0."
- [discussion] pure-sample fallback: "If no confident normal cells are found, we assume that the sample is pure and contains only malignant cells."
- [methods] confident normal cell function signature: "getConfidentNormalCells(mtx, sample, par_cores, AdditionalGeneSets, SCEVANsignatures, organism, output_dir)"
- [discussion] aneuploidy assumption limitation: "Thus, our approach (and similar) may not be suited in this case."
