---
name: batch-parallel-cell-classification
description: Use when you have raw scRNA-seq count matrices (genes x cells) of tumor
  biopsies and need fully unsupervised, automatic malignant/non-malignant cell classification
  without manually picking reference cell clusters, ideally across many samples in
  parallel.
when_to_use_negative:
- Tumors with minimal genomic alterations (e.g. leukemias, pediatric cancers, ependymomas)
  — the aneuploidy-based assumption makes SCEVAN unsuited there
- 'You only want clonal structure and already know the normal cells: supply norm_cells
  with FIXED_NORMAL_CELLS=TRUE instead of relying on automatic classification'
- The input is already a processed feature table or labeled cell annotation rather
  than a raw count matrix
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCEVAN
  role: 'Primary tool: automatic malignant/non-malignant classification and clonal
    CN substructure via pipelineCNA / classifyTumorCells, multi-core (par_cores)'
  repo: github.com/AntonioDeFalco/SCEVAN
- name: CopyKAT v1.0.5
  role: Baseline comparator for automatic malignant/non-malignant classification in
    the F1 benchmark
- name: yaGST
  role: Single-sample Mann–Whitney–Wilcoxon gene set test used internally to identify
    high-confidence normal cells (P < 1e-10, NES > 1.0)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:9659b991f26adbec2389ad0b8686ca31079268c091919cf89dbe830243c52576
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CopyKAT v1.0.5
  - SCEVAN
  - yaGST
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# batch-parallel-cell-classification

## Summary

Automatically segregate malignant from non-malignant cells in raw scRNA-seq count matrices using SCEVAN's unsupervised variational aneuploidy pipeline (SCEVAN::pipelineCNA / classifyTumorCells), parallelized over multiple cores, optionally benchmarked against CopyKAT via per-matrix F1 scores against known tumor/normal labels.

## When to use

Use when you have raw scRNA-seq count matrices (genes x cells) of tumor biopsies and need fully unsupervised, automatic malignant/non-malignant cell classification without manually picking reference cell clusters, ideally across many samples in parallel.

## When NOT to use

- Tumors with minimal genomic alterations (e.g. leukemias, pediatric cancers, ependymomas) — the aneuploidy-based assumption makes SCEVAN unsuited there
- You only want clonal structure and already know the normal cells: supply norm_cells with FIXED_NORMAL_CELLS=TRUE instead of relying on automatic classification
- The input is already a processed feature table or labeled cell annotation rather than a raw count matrix

## Inputs

- Raw scRNA-seq count matrix with genes on rows (Gene Symbol or Ensembl ID) and cells on columns
- Optional ground-truth malignant/non-malignant labels (e.g. synthetic matrices from Supplementary Data 3, organized by dropout noise level x0 = none, -2, -1, 0)
- Optional vector of known normal cells (norm_cells) with FIXED_NORMAL_CELLS flag

## Outputs

- Per-cell malignant/non-malignant classification assignments
- Subclone assignments and phylogenetic tree when SUBCLONES=TRUE / plotTree=TRUE
- Per-matrix F1 scores and aggregated benchmark table (e.g. benchmark_f1_scores.csv with rows=method, columns=dropout level)

## How to apply

Run SCEVAN's single-call pipeline (SCEVAN::pipelineCNA or classifyTumorCells) on the raw count matrix with genes on rows and cells on columns, setting par_cores (default 20) for multi-core batch execution and SUBCLONES=TRUE/FALSE depending on whether clonal structure or only tumor/normal classification is needed. Internally the pipeline removes cells with <200 detected genes, removes genes expressed in <1% of cells, annotates genes with Ensembl genomic locations and sorts by coordinates, filters REACTOME cell cycle genes to reduce artificial segments, and identifies confident normal cells via gene signatures tested with yaGST (P < 1e-10, NES > 1.0). The copy number baseline (median of confident normal cells) is subtracted, the relative matrix is TV-smoothed with a logcosh prior, and joint segmentation is performed by greedy Mumford–Shah piecewise-constant minimization (beta_vega, default 0.5, controls segmentation coarseness). For benchmarking, compare predicted labels against ground-truth tumor/normal annotations per matrix and average per-matrix F1 scores across dropout levels; paper-reported means for 500 synthetic matrices were SCEVAN 0.948/0.943/0.909/0.824 vs CopyKAT v1.0.5 0.798/0.792/0.763/0.726 at x0 = none/-2/-1/0. If no confident normal cells are found, the sample is assumed pure malignant and a synthetic gaussian baseline is removed instead.

## Related tools

- **SCEVAN** (Primary tool: automatic malignant/non-malignant classification and clonal CN substructure via pipelineCNA / classifyTumorCells, multi-core (par_cores)) — github.com/AntonioDeFalco/SCEVAN
- **CopyKAT v1.0.5** (Baseline comparator for automatic malignant/non-malignant classification in the F1 benchmark)
- **yaGST** (Single-sample Mann–Whitney–Wilcoxon gene set test used internally to identify high-confidence normal cells (P < 1e-10, NES > 1.0))

## Examples

```
results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)
```

## Evaluation signals

- Recomputed mean F1 scores on the 500 synthetic matrices match paper values (SCEVAN 0.948/0.943/0.909/0.824; CopyKAT 0.798/0.792/0.763/0.726 at x0 = none/-2/-1/0) within a small tolerance
- Benchmark table is present and numerically complete for both methods at all four dropout levels, with rows=method and columns=dropout level
- On real data (106 samples, ~93,322 cells) expected aggregate F1 is ~0.90 for SCEVAN versus ~0.63 for state-of-the-art comparators
- QC thresholds honored before classification: cells with <200 detected genes and genes expressed in <1% of cells removed; REACTOME cell cycle genes filtered

## Limitations

- Aneuploidy-based approach is unsuitable for tumors with minimal genomic alterations (leukemia, pediatric cancers, ependymomas)
- SCEVAN's own package version is not stated, unlike compared tools (CopyKAT v1.0.5, inferCNV v1.4.0)
- If no confident normal cells are found the sample is assumed pure malignant, which can misclassify tumor microenvironment cells
- Ground truth from WES/WGS has different resolution than single-cell data and requires resampling before comparison

## Evidence

- [abstract] SCEVAN can automatically and accurately discriminate between malignant and non-malignant cells: "SCEVAN can automatically and accurately discriminate between malignant and non-malignant cells, resulting in a practical framework to analyze tumors and their microenvironment"
- [methods] pipelineCNA single-call classification with 20 cores: "results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)"
- [discussion] QC filters before classification: "the cells with less than 200 detected genes and the genes expressed in less than 1% of cells are removed."
- [readme] F1 benchmark on real samples: [evidence span withheld — anchor status: external_doc]
- [discussion] Confident normal cell criteria: "assume as normal confident cells the top classified cells with P value less than 10−10 and Normalized Enrichment Score (NES) greater than 1.0."
