---
name: copynumber-based-malignancy-classification
description: Use when you have a raw scRNA-seq count matrix from a tumor biopsy and
  need automatic malignant/non-malignant cell segregation without manually identifying
  reference normal cell clusters.
when_to_use_negative:
- Tumors with minimal genomic alterations (e.g. leukemia, pediatric cancers, Ependymomas)
  where the aneuploidy-based assumption is unsuitable
- You already have independently identified confident normal reference cells and only
  need clonal structure — then FIXED_NORMAL_CELLS-style usage, not re-classification,
  is appropriate
- Data is not a raw count matrix (e.g. already normalized feature table), since the
  method requires raw counts for baseline subtraction and segmentation
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: SCEVAN
  role: 'Primary tool: automatic malignant/non-malignant classification and clonal
    CNA substructure inference via pipelineCNA / classifyTumorCells'
  repo: github.com/AntonioDeFalco/SCEVAN
- name: CopyKAT v1.0.5
  role: Comparison baseline for automatic malignant/non-malignant classification
- name: inferCNV v1.4.0
  role: Alternative CNV-based classifier requiring manual identification of reference
    clusters; used for comparison
- name: yaGST
  role: Mann–Whitney–Wilcoxon single-sample gene set test used to identify high-confidence
    normal cells
  repo: github.com/miccec/yaGST
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:94ffb934e5e9a477c5b87f6bb7e70da8ad85e8ef861546c796242402319b478f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CopyKAT v1.0.5
  - SCEVAN
  - inferCNV v1.4.0
  - yaGST
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# copynumber-based-malignancy-classification

## Summary

Automatically classifies malignant vs non-malignant cells from raw scRNA-seq count matrices by inferring copy number alterations, using SCEVAN's variational Mumford–Shah segmentation with confident normal cells as baseline. Supports fully unsupervised end-to-end analysis, benchmarked against CopyKAT on 500 synthetic matrices (322,687 cells).

## When to use

Use when you have a raw scRNA-seq count matrix from a tumor biopsy and need automatic malignant/non-malignant cell segregation without manually identifying reference normal cell clusters.

## When NOT to use

- Tumors with minimal genomic alterations (e.g. leukemia, pediatric cancers, Ependymomas) where the aneuploidy-based assumption is unsuitable
- You already have independently identified confident normal reference cells and only need clonal structure — then FIXED_NORMAL_CELLS-style usage, not re-classification, is appropriate
- Data is not a raw count matrix (e.g. already normalized feature table), since the method requires raw counts for baseline subtraction and segmentation

## Inputs

- Raw scRNA-seq count matrix (genes x cells; Gene Symbol or Ensembl ID), e.g. from a droplet-based tumor dataset such as GSE131928 MGH106
- Optional ground-truth tumor/normal labels for supervised benchmarking (e.g. synthetic matrices from Supplementary Data 3)
- Optional vector of known normal cells (norm_cells) to use as confident normal reference

## Outputs

- Per-cell malignant/non-malignant classification (and subclone assignment if SUBCLONES = TRUE)
- Per-cell copy number profile (.seg / CNA matrix, e.g. MGH106_CNAmtx.RData)
- Benchmark table of mean F1 scores per method per dropout level (e.g. benchmark_f1_scores.csv)

## How to apply

Load the raw count matrix (genes on rows, cells on columns; Gene Symbols or Ensembl IDs accepted). SCEVAN first filters low-quality cells (<200 detected genes) and genes expressed in <1% of cells, removes REACTOME cell-cycle genes, and annotates/sorts genes by genomic coordinates. It identifies high-confidence normal cells via yaGST gene-signature tests (P < 1e-10, NES > 1.0, top 30 cells), subtracts the median-expression baseline of these cells, applies logcosh total-variation smoothing, then performs greedy Mumford–Shah joint segmentation to infer aneuploidy per cell. Run the whole pipeline with pipelineCNA (or classifyTumorCells for classification only, optionally supplying ground_truth labels when benchmarking), parallelized over par_cores (default 20). If benchmarking against CopyKAT v1.0.5, compute per-matrix F1 scores against known tumor/normal labels and average across dropout-noise levels (x0 = none, -2, -1, 0).

## Related tools

- **SCEVAN** (Primary tool: automatic malignant/non-malignant classification and clonal CNA substructure inference via pipelineCNA / classifyTumorCells) — github.com/AntonioDeFalco/SCEVAN
- **CopyKAT v1.0.5** (Comparison baseline for automatic malignant/non-malignant classification)
- **inferCNV v1.4.0** (Alternative CNV-based classifier requiring manual identification of reference clusters; used for comparison)
- **yaGST** (Mann–Whitney–Wilcoxon single-sample gene set test used to identify high-confidence normal cells) — github.com/miccec/yaGST

## Examples

```
results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)
```

## Evaluation signals

- On 500 synthetic matrices with ground-truth labels, mean F1 vs known labels should approximate SCEVAN's reported 0.948/0.943/0.909/0.824 and CopyKAT's 0.798/0.792/0.763/0.726 for dropout levels x0 = none/-2/-1/0 within tolerance
- Benchmark table contains numeric F1 values for both methods at all four dropout levels (no missing cells)
- If no confident normal cells are found (P < 1e-10, NES > 1.0), the method should treat the sample as pure tumor with a synthetic Gaussian baseline rather than fail
- Output classification covers all cells in the input matrix with a malignant/non-malignant label, and results integrate into a Seurat object via AddMetaData
- Whole-run completion of pipelineCNA at default par_cores = 20 without manual intervention (fully unsupervised)

## Limitations

- Aneuploidy-based approaches may not suit tumors with minimal genomic alterations such as leukemia, pediatric cancers, and Ependymomas
- CopyKAT classification can be affected by wrong identification of normal cells and was not designed for fully automatic end-to-end clone identification
- Cell-cycle artifacts must be explicitly filtered (REACTOME cell-cycle genes) or they create artificial segments
- F1 score on real data was 0.90 overall (README), not perfect; performance degrades at higher dropout noise (0.824 vs 0.948 for SCEVAN at x0 = 0 vs none)

## Evidence

- [abstract] SCEVAN can automatically and accurately discriminate between malignant and non-malignant cells, resulting in a practical framework to analyze tumors and their microenvironment: "SCEVAN can automatically and accurately discriminate between malignant and non-malignant cells, resulting in a practical framework"
- [discussion] Confident normal cell identification criteria: "assume as normal confident cells the top classified cells with P value less than 10−10 and Normalized Enrichment Score (NES) greater than 1.0."
- [discussion] Filtering thresholds: "the cells with less than 200 detected genes and the genes expressed in less than 1% of cells are removed."
- [results] Reported benchmark F1 scores vs CopyKAT: [evidence span withheld — anchor status: not_found]
- [methods] pipelineCNA usage example: "results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)"
- [readme] README description of package purpose and F1 comparison: [evidence span withheld — anchor status: external_doc]
