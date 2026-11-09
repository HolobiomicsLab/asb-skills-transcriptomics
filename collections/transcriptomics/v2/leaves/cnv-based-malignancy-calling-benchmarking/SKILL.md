---
name: cnv-based-malignancy-calling-benchmarking
description: Use when a scRNA-seq dataset requires distinguishing malignant from non-malignant
  cells and you must decide between marker-based aberrant annotation and CNV-based
  callers (SCEVAN, CopyKAT), or quantify their accuracy and runtime trade-offs.
when_to_use_negative:
- The task is general healthy-tissue cell-type annotation rather than malignant vs
  non-malignant cell discrimination — the CNV-based callers and the disease benchmark
  are specific to aberrant/malignant populations
- Only a small tumor cohort or no ground-truth malignant annotations exist — the article
  notes the benchmark is limited to two liquid and two solid tumors and that limited
  availability of high-quality ground-truth datasets constrains expansion
- A direct comparison with non-marker-based annotation tools is required — the authors
  state such a comparison was not conducted in this study
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: Cell Marker Accordion
  role: Marker-based aberrant/malignant cell identification via accordion_disease
    using the built-in disease marker database
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
- name: SCEVAN
  role: CNV-calling baseline for malignant cell identification with default parameters
- name: CopyKAT
  role: CNV-calling baseline for malignant cell identification with default parameters
- name: Seurat
  role: Standard scRNA-seq pre-processing pipeline for the analyzed published single-cell
    datasets
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:c0310b674345a33fcaf3487792d60fe4a209c7989af5f27c111055e627064934
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cnv-based-malignancy-calling-benchmarking

## Summary

Benchmark of marker-based aberrant cell identification (Cell Marker Accordion accordion_disease) against CNV-calling malignant cell tools SCEVAN and CopyKAT on glioblastoma and lung adenocarcinoma scRNA-seq, using F1 score, percentage of recovered malignant cells, and wall-clock runtime.

## When to use

Use when a scRNA-seq dataset requires distinguishing malignant from non-malignant cells and you must decide between marker-based aberrant annotation and CNV-based callers (SCEVAN, CopyKAT), or quantify their accuracy and runtime trade-offs.

## When NOT to use

- The task is general healthy-tissue cell-type annotation rather than malignant vs non-malignant cell discrimination — the CNV-based callers and the disease benchmark are specific to aberrant/malignant populations
- Only a small tumor cohort or no ground-truth malignant annotations exist — the article notes the benchmark is limited to two liquid and two solid tumors and that limited availability of high-quality ground-truth datasets constrains expansion
- A direct comparison with non-marker-based annotation tools is required — the authors state such a comparison was not conducted in this study

## Inputs

- Published glioblastoma scRNA-seq dataset with author tumor clusters (1, 4, 5, 7, 10, 11, 12, 20, 21) and marker list (SOX2, OLIG1, GFAP, S100B)
- Published lung adenocarcinoma scRNA-seq dataset with malignant cell-type annotations
- Cell Marker Accordion built-in disease marker database
- SCEVAN and CopyKAT R packages (default parameters)
- Seurat scRNA-seq objects / pre-processed count matrices

## Outputs

- cancer_benchmark_results.csv with per-cell predictions, precision, recall, F1, percentage of correctly identified aberrant cells, and wall-clock runtimes per tool and cancer type
- Comparison of Accordion F1 (~0.97 GBM; 0.84 lung) vs SCEVAN/CopyKAT (0.52/0.52) including the neoplastic endothelial subpopulation call

## How to apply

Load the published glioblastoma scRNA-seq dataset and define ground-truth tumor cells as clusters 1, 4, 5, 7, 10, 11, 12, 20, 21 derived from the author-provided tumor marker list (SOX2, OLIG1, GFAP, S100B); load the lung adenocarcinoma dataset with its published malignant cell-type annotations. Run the Cell Marker Accordion (accordion_disease) on each dataset to flag aberrant/malignant cells using the built-in disease marker database, then run SCEVAN and CopyKAT on identical inputs with default parameters for CNV-based malignant cell calling. Compute the percentage of correctly identified aberrant cells plus precision, recall, and F1 for each tool against the ground truth per cancer type, and record wall-clock runtimes on identical hardware. Compile per-cell predictions, metrics, and runtimes into a results table (cancer_benchmark_results.csv), expecting Accordion F1 ~0.97 (GBM) and 0.84 (lung adenocarcinoma) versus 0.52 for both SCEVAN and CopyKAT, and runtimes <2 min versus >6 h (SCEVAN) and >7 days (CopyKAT).

## Related tools

- **Cell Marker Accordion** (Marker-based aberrant/malignant cell identification via accordion_disease using the built-in disease marker database) — https://github.com/TebaldiLab/cellmarkeraccordion
- **SCEVAN** (CNV-calling baseline for malignant cell identification with default parameters)
- **CopyKAT** (CNV-calling baseline for malignant cell identification with default parameters)
- **Seurat** (Standard scRNA-seq pre-processing pipeline for the analyzed published single-cell datasets)

## Examples

```
Rscript -e 'library(cellmarkeraccordion); res <- accordion_disease(seurat_obj); write.csv(res, "cancer_benchmark_results.csv")'
```

## Evaluation signals

- Cell Marker Accordion recovers 100% of ground-truth malignant cells in glioblastoma with F1 ~0.97
- In lung adenocarcinoma, Accordion F1 reaches 0.84 versus 0.52 for both SCEVAN and CopyKAT
- Accordion annotation completes in under 2 minutes versus more than 6 hours (SCEVAN) and more than 7 days (CopyKAT) on identical inputs and hardware
- Precision, recall, and F1 are computed against the predefined GBM tumor clusters (1, 4, 5, 7, 10, 11, 12, 20, 21) and published lung adenocarcinoma malignant annotations
- Output includes the neoplastic endothelial subpopulation call consistent with the published comparison

## Limitations

- The cancer benchmark is limited to two liquid and two solid tumors; benchmark expansion is planned as high-quality ground-truth datasets become available
- CNV-based callers are computationally expensive: CopyKAT required more than 7 days on the benchmark inputs
- The glioblastoma ground truth is derived from author-provided tumor clusters based on a marker list (SOX2, OLIG1, GFAP, S100B), so metric values depend on that ground-truth definition

## Evidence

- [discussion] Some approaches, such as SCEVAN and CopyKAT, attempt to distinguish malignant cancer cells in single-cell RNA-seq through CNV calling: "Some approaches, such as SCEVAN and CopyKAT, attempt to distinguish malignant cancer cells in single-cell RNA-seq through CNV calling"
- [methods] accordion function usage: "Users can annotate clusters or cells exploiting the built-in Cell Marker Accordion database with the accordion() function"
- [methods] standard scRNA-seq pre-processing with Seurat: "Single-cell datasets were analyzed employing a standard pre-processing pipeline for single-cell RNA-seq data, using Seurat"
- [discussion] benchmark scope limitation: "our current benchmark, focused on two liquid and two solid tumors, demonstrates improved performance within these specific contexts. We plan to expand this benchmark in the future"
- [abstract] improved annotation accuracy: "We validate our approach on multiple single-cell and spatial datasets from different human and murine tissues, improving annotation accuracy in all cases."
