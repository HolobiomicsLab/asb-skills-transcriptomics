---
name: synthetic-scrnaseq-benchmark-evaluation
description: Use when benchmarking malignant/non-malignant classification methods
  on synthetic scRNA-seq count matrices with known ground-truth labels, especially
  when quantifying robustness to dropout noise via F1 scores.
when_to_use_negative:
- The dataset consists of real (not synthetic) tumor scRNA-seq samples without ground-truth
  labels — no F1 scores can be computed and the classification should be evaluated
  on biological grounds instead
- The goal is to evaluate clonal substructure/subclone identification rather than
  the binary malignant/non-malignant classification task
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: SCEVAN
  role: Automatic malignant/non-malignant classification of cells using pipelineCNA/classifyTumorCells
    with ground_truth labels supplied, parallelized over 20 cores
  repo: github.com/AntonioDeFalco/SCEVAN
- name: CopyKAT v1.0.5
  role: 'Baseline comparator: automatic malignant/non-malignant classification against
    which SCEVAN''s F1 scores are compared'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:ff8b324637fdbec101907706a05603e24551c981652ea314ccb43dfc1ed24bac
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CopyKAT v1.0.5
  - SCEVAN
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# synthetic-scrnaseq-benchmark-evaluation

## Summary

Benchmark protocol for evaluating automatic malignant/non-malignant cell classification on synthetic scRNA-seq count matrices with known tumor/normal ground truth, comparing SCEVAN against CopyKAT v1.0.5 via mean F1 scores across dropout noise levels.

## When to use

Use when benchmarking malignant/non-malignant classification methods on synthetic scRNA-seq count matrices with known ground-truth labels, especially when quantifying robustness to dropout noise via F1 scores.

## When NOT to use

- The dataset consists of real (not synthetic) tumor scRNA-seq samples without ground-truth labels — no F1 scores can be computed and the classification should be evaluated on biological grounds instead
- The goal is to evaluate clonal substructure/subclone identification rather than the binary malignant/non-malignant classification task

## Inputs

- 500 synthetic scRNA-seq count matrices with known tumor/normal ground-truth labels (Supplementary Data 3)
- Dropout noise level grouping (x0 = none, -2, -1, 0)
- Raw count matrix format: genes on rows (Gene Symbol or Ensembl ID), cells on columns

## Outputs

- Per-matrix F1 scores for SCEVAN and CopyKAT at each dropout level
- Mean F1 scores per method per dropout level (benchmark_f1_scores.csv: rows = method, columns = dropout level)

## How to apply

Load the synthetic scRNA-seq count matrices (500 matrices, 322,687 cells total) with known tumor/normal ground-truth labels from Supplementary Data 3, organized by dropout noise level (x0 = none, -2, -1, 0). For each matrix, run SCEVAN's automatic classification via SCEVAN::pipelineCNA or classifyTumorCells with parallelization over 20 cores (par_cores = 20) and collect predicted labels; run CopyKAT v1.0.5 on the same matrices and collect its predictions. Compute per-matrix F1 scores against the known labels for each method and dropout level, then average over the matrices. The expected reference means are SCEVAN 0.948/0.943/0.909/0.824 and CopyKAT 0.798/0.792/0.763/0.726 for x0 = none/-2/-1/0. Assemble the recomputed means into a table (rows = method, columns = dropout level) written to benchmark_f1_scores.csv and verify agreement with the paper-reported values within a small tolerance.

## Related tools

- **SCEVAN** (Automatic malignant/non-malignant classification of cells using pipelineCNA/classifyTumorCells with ground_truth labels supplied, parallelized over 20 cores) — github.com/AntonioDeFalco/SCEVAN
- **CopyKAT v1.0.5** (Baseline comparator: automatic malignant/non-malignant classification against which SCEVAN's F1 scores are compared)

## Examples

```
results <- SCEVAN::pipelineCNA(count_mtx, sample = "synthetic_001", par_cores = 20, SUBCLONES = FALSE)
```

## Evaluation signals

- Recomputed mean F1 values match paper-reported values (SCEVAN 0.948/0.943/0.909/0.824; CopyKAT 0.798/0.792/0.763/0.726 for x0 = none/-2/-1/0) within a small tolerance
- benchmark_f1_scores.csv is present and numerically complete for both methods at all four dropout levels
- F1 scores decrease monotonically (or near-monotonically) as dropout noise increases for each method, consistent with the reported trend
- Predicted label vectors are non-empty and aligned to the ground-truth labels for every matrix analyzed
- SCEVAN's mean F1 exceeds CopyKAT's at every dropout level, as reported in the article

## Limitations

- An aneuploidy-based approach like SCEVAN may not be suited to tumors with minimal genomic alterations (e.g. leukemia, pediatric cancers, Ependymomas)
- SCEVAN was reported with an overall F1 of 0.90 on 106 real samples versus 0.63 for state-of-the-art tools; classification quality depends on identifying confident normal cells via gene signatures and may fail in pure malignant samples with no confident normal cells

## Evidence

- [abstract] F1 scores across dropout levels: [evidence span withheld — anchor status: external_doc]
- [intro] CopyKAT as comparator: "An approach to overcome these limitations is represented by the CopyKAT method11 that automatically classifies malignant and non-malignant cells."
- [methods] SCEVAN invocation: "results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)"
- [discussion] Ground truth resolution caveat: "The ground truth extracted from WES and WGS have of course different resolutions with respect to the single-cell data."
- [readme] pipelineCNA usage: [evidence span withheld — anchor status: external_doc]
