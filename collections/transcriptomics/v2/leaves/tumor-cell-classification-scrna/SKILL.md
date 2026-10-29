---
name: tumor-cell-classification-scrna
description: Use when you have raw scRNA-seq count matrices (Smart-seq2 or 10X Chromium)
  from tumor biopsies and need a fully unsupervised malignant/non-malignant cell classification
  without manually identified reference cell clusters.
when_to_use_negative:
- Tumors with minimal genomic alterations where the aneuploidy assumption fails (e.g.
  leukemia, pediatric cancers, Ependymomas)
- You already have manual reference clusters and prefer inferCNV-style guided analysis,
  or the goal is clonal phylogeny across multiple samples rather than single-sample
  classification (use multiSampleComparisonClonalCN instead)
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3361
tools:
- name: SCEVAN
  role: 'Primary tool: pipelineCNA performs automatic malignant/non-malignant classification
    via confident-normal-cell detection and variational segmentation'
  repo: github.com/AntonioDeFalco/SCEVAN
- name: CopyKAT v1.0.5
  role: Baseline comparator for automatic malignant/non-malignant classification (overall
    F1 0.63 vs SCEVAN 0.90)
- name: yaGST
  role: Mann–Whitney–Wilcoxon single-sample gene set test used to detect confident
    normal cells
  repo: github.com/miccec/yaGST
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:a37153122fc88e633dbd1cb02a0ed4a33c96706b7042a6fcccc4dc5a6e26b488
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CNVkit v0.9.9
  - CopyKAT v1.0.5
  - DNAcopy
  - SCEVAN
  - inferCNV v1.4.0
  - yaGST
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# tumor-cell-classification-scrna

## Summary

Automatically classify malignant versus non-malignant cells in tumor scRNA-seq count matrices using SCEVAN, a variational aneuploidy-analysis approach that detects confident normal cells and segments copy number profiles, benchmarked against CopyKAT.

## When to use

Use when you have raw scRNA-seq count matrices (Smart-seq2 or 10X Chromium) from tumor biopsies and need a fully unsupervised malignant/non-malignant cell classification without manually identified reference cell clusters.

## When NOT to use

- Tumors with minimal genomic alterations where the aneuploidy assumption fails (e.g. leukemia, pediatric cancers, Ependymomas)
- You already have manual reference clusters and prefer inferCNV-style guided analysis, or the goal is clonal phylogeny across multiple samples rather than single-sample classification (use multiSampleComparisonClonalCN instead)

## Inputs

- Raw scRNA-seq count matrix (genes x cells; Gene Symbol or Ensembl ID), e.g. MGH106 GBM sample from GSE131928
- Optional known-normal cell name vector (norm_cells) with FIXED_NORMAL_CELLS = TRUE
- Optional additional normal-cell-type gene signatures (AdditionalGeneSets)
- Platform label per sample (Smart-seq2 vs 10X Chromium) for benchmarking

## Outputs

- Per-cell malignant/non-malignant classification metadata (aneuploidy/diploid) suitable for Seurat meta.data
- Optional subclone assignments, shared/specific CNA segments (.seg) and phylogenetic tree (plotTree = TRUE)
- Aggregate per-sample F1 summary (SCEVAN 0.90 vs CopyKAT 0.63 on 106 samples / 93,322 cells)

## How to apply

Run SCEVAN::pipelineCNA on each raw count matrix (genes on rows, cells on columns; Gene Symbol or Ensembl ID) with par_cores = 20 and SUBCLONES = FALSE if only pure malignant/non-malignant classification is needed. Internally, the pipeline filters cells with <200 detected genes and genes expressed in <1% of cells, annotates genes with genomic locations and sorts by coordinates, filters REACTOME cell-cycle genes, and identifies confident normal cells via yaGST Mann–Whitney–Wilcoxon signature tests (P < 1e-10, NES > 1.0, up to 30 cells per signature). If no confident normal cells are found, the sample is assumed pure malignant; otherwise the median expression of confident normal cells is subtracted as the copy number baseline before TV (logcosh) smoothing and greedy Mumford–Shah joint segmentation (beta_vega default 0.5). Results can be added directly to a Seurat object via AddMetaData; compare per-sample F1 scores against ground truth from bulk WES/WGS copy number profiling when validating.

## Related tools

- **SCEVAN** (Primary tool: pipelineCNA performs automatic malignant/non-malignant classification via confident-normal-cell detection and variational segmentation) — github.com/AntonioDeFalco/SCEVAN
- **CopyKAT v1.0.5** (Baseline comparator for automatic malignant/non-malignant classification (overall F1 0.63 vs SCEVAN 0.90))
- **yaGST** (Mann–Whitney–Wilcoxon single-sample gene set test used to detect confident normal cells) — github.com/miccec/yaGST

## Examples

```
results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = FALSE)
```

## Evaluation signals

- Aggregate per-sample F1 on 106 real samples (GBM, HNSCC, colorectal) should match SCEVAN 0.90 within tolerance
- SCEVAN should outperform CopyKAT (F1 0.63) in ~63% of samples
- Confident-normal-cell detection should return signatures passing P < 1e-10 and NES > 1.0; absence triggers pure-tumor assumption
- Output metadata should attach to a Seurat object without row-name mismatches (cell names preserved)
- Segmentation stage should be substantially faster than inferCNV/CopyKAT (reported 11x and 19x on Multiple Myeloma 10X data)

## Limitations

- Aneuploidy-based inference is unsuitable for tumors with minimal genomic alterations such as leukemia, pediatric cancers, and Ependymomas
- Ground truth from WES/WGS has different resolution than single-cell data, requiring resampling before comparison
- On droplet-based data with very sparse coverage, inferCNV-like approaches (the comparison family) perform sub-optimally; CopyKAT is particularly slow on large cell counts (e.g. Multiple Myeloma 10X data)

## Evidence

- [methods] Automatic classification via pipelineCNA: "results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)"
- [abstract] Benchmark F1 finding: [evidence span withheld — anchor status: external_doc]
- [discussion] Filter thresholds: "the cells with less than 200 detected genes and the genes expressed in less than 1% of cells are removed."
- [discussion] Confident normal cell detection thresholds: "assume as normal confident cells the top classified cells with P value less than 10−10 and Normalized Enrichment Score (NES) greater than 1.0."
- [readme] Usage guidance from README: [evidence span withheld — anchor status: not_found]
- [discussion] Pure-tumor fallback: "If no confident normal cells are found, we assume that the sample is pure and contains only malignant cells."
