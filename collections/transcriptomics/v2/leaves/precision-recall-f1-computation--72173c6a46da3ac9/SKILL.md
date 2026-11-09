---
name: precision-recall-f1-computation
description: Use when comparing per-cell predictions of malignant-cell identification
  tools (marker-based or CNV-based) against a published ground-truth label vector,
  and accuracy, precision, recall, and F1 with wall-clock runtime must be reported.
when_to_use_negative:
- No ground-truth (author-confirmed) malignant cell labels exist for the dataset —
  precision/recall/F1 cannot be validated
- Predictions are at cluster or sample level rather than per-cell, so per-cell F1
  is not meaningful
- The task is general cell-type annotation of healthy tissues rather than aberrant/malignant
  cell identification benchmarking
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cell Marker Accordion
  role: Source of predicted aberrant/malignant cells via the accordion_disease function,
    evaluated against ground truth
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
- name: SCEVAN
  role: CNV-based malignant cell caller run with default parameters; predictions scored
    with precision/recall/F1 baseline
- name: CopyKAT
  role: CNV-based malignant cell caller run with default parameters; predictions scored
    with precision/recall/F1 baseline
- name: Seurat
  role: Standard scRNA-seq pre-processing pipeline used to prepare datasets before
    annotation and scoring
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:58755f42f8c303d2a97abcb682ca0723af4affd28abdc51d68efa79ae4444a9e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TebaldiLab/cellmarkeraccordion
  tools:
  - Cell Marker Accordion
  - CopyKAT
  - R
  - SCEVAN
  - Seurat
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# precision-recall-f1-computation

## Summary

Computation of percentage of correctly identified cells, precision, recall, and F1 score to benchmark predicted malignant/aberrant cell calls (e.g., from Cell Marker Accordion, SCEVAN, or CopyKAT) against author-provided ground-truth annotations in scRNA-seq cancer datasets.

## When to use

Use when comparing per-cell predictions of malignant-cell identification tools (marker-based or CNV-based) against a published ground-truth label vector, and accuracy, precision, recall, and F1 with wall-clock runtime must be reported.

## When NOT to use

- No ground-truth (author-confirmed) malignant cell labels exist for the dataset — precision/recall/F1 cannot be validated
- Predictions are at cluster or sample level rather than per-cell, so per-cell F1 is not meaningful
- The task is general cell-type annotation of healthy tissues rather than aberrant/malignant cell identification benchmarking

## Inputs

- Ground-truth malignant-cell labels (published cluster assignments, e.g., GBM tumor clusters from SOX2/OLIG1/GFAP/S100B marker list; lung adenocarcinoma published malignant annotations)
- Per-cell binary predictions from accordion_disease (Cell Marker Accordion)
- Per-cell predictions from SCEVAN (default parameters)
- Per-cell predictions from CopyKAT (default parameters)
- Wall-clock runtime measurements per tool on identical inputs and hardware

## Outputs

- Percentage of correctly identified aberrant cells per tool per cancer type
- Precision, recall, and F1 score per tool per cancer type
- Tool wall-clock runtimes
- cancer_benchmark_results.csv compiling per-cell predictions, metrics, and runtimes

## How to apply

Define a binary ground-truth vector per dataset — for the glioblastoma scRNA-seq benchmark, tumor cells are clusters 1, 4, 5, 7, 10, 11, 12, 20, 21 identified via the author-provided marker list (SOX2, OLIG1, GFAP, S100B); for lung adenocarcinoma, use the published malignant cell-type annotations. For each tool (accordion_disease output, SCEVAN, CopyKAT run with default parameters), treat predicted malignant cells as positives and compute the percentage of correctly identified aberrant cells plus precision, recall, and F1 against ground truth. Record wall-clock runtimes on identical inputs and hardware (Accordion <2 min, SCEVAN >6 h, CopyKAT >7 days in the published benchmark). Compile per-cell predictions, metrics, and runtimes into a single results table (cancer_benchmark_results.csv). Validate by checking the reproduced metric values match the published benchmark (Accordion F1 ~0.97 GBM, 0.84 lung adenocarcinoma vs 0.52 for both SCEVAN and CopyKAT).

## Related tools

- **Cell Marker Accordion** (Source of predicted aberrant/malignant cells via the accordion_disease function, evaluated against ground truth) — https://github.com/TebaldiLab/cellmarkeraccordion
- **SCEVAN** (CNV-based malignant cell caller run with default parameters; predictions scored with precision/recall/F1 baseline)
- **CopyKAT** (CNV-based malignant cell caller run with default parameters; predictions scored with precision/recall/F1 baseline)
- **Seurat** (Standard scRNA-seq pre-processing pipeline used to prepare datasets before annotation and scoring)

## Examples

```
from sklearn.metrics import precision_recall_fscore_support; p, r, f1, _ = precision_recall_fscore_support(y_true, y_pred, average='binary')
```

## Evaluation signals

- Reproduced GBM F1 for Cell Marker Accordion is approximately 0.97 with 100% of malignant cells identified
- Reproduced lung adenocarcinoma F1 is 0.84 for Accordion versus 0.52 for both SCEVAN and CopyKAT
- Accordion wall-clock runtime is under 2 minutes versus more than 6 hours (SCEVAN) and more than 7 days (CopyKAT) on identical inputs and hardware
- All per-cell predictions, metrics, and runtimes are present in cancer_benchmark_results.csv
- Percentages of correctly identified aberrant cells are computed against the exact published ground-truth cluster sets (GBM clusters 1, 4, 5, 7, 10, 11, 12, 20, 21)

## Limitations

- The published cancer benchmark covers only two liquid and two solid tumors; metric expectations may not transfer to other cancer types
- Limited availability of high-quality ground-truth datasets constrains benchmark expansion
- Direct comparison with annotation tools that do not rely on marker genes was not conducted

## Evidence

- [other] GBM ground truth definition: [evidence span withheld — anchor status: not_found]
- [other] Benchmark metrics: [evidence span withheld — anchor status: not_found]
- [discussion] CNV tool framing: "Some approaches, such as SCEVAN and CopyKAT, attempt to distinguish malignant cancer cells in single-cell RNA-seq through CNV calling"
- [methods] Accordion automatic annotation: "Users can annotate clusters or cells exploiting the built-in Cell Marker Accordion database with the accordion() function"
- [discussion] Benchmark scope limitation: "our current benchmark, focused on two liquid and two solid tumors, demonstrates improved performance within these specific contexts. We plan to expand this benchmark in the future"
