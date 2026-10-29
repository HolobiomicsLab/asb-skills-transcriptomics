---
name: runtime-benchmark-comparison
description: Use when a new single-cell annotation or cell-calling tool must be validated
  against baseline tools on the same datasets, with per-cell predictions scored against
  ground-truth labels and runtimes recorded on identical hardware.
when_to_use_negative:
- No high-quality ground-truth annotations exist for the test dataset — the article
  notes the availability of high-quality, ground-truth datasets constrains benchmark
  expansion
- The candidate tool is not marker-based and comparison against marker-based tools
  is the goal — the article states a direct comparison between non-marker-based annotation
  approaches and the Cell Marker Accordion was not conducted
tools:
- name: Cell Marker Accordion
  role: Candidate tool under benchmark; flags aberrant/malignant cells via accordion_disease
    using the built-in disease marker database
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
- name: SCEVAN
  role: CNV-based baseline for malignant cell identification, run with default parameters
    on identical inputs
- name: CopyKAT
  role: CNV-based baseline for malignant cell identification, run with default parameters
    on identical inputs
- name: Seurat
  role: Standard scRNA-seq preprocessing pipeline applied to the benchmark datasets
    before annotation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:40937943d8aa6936182be0eb8ffe5995528e30592ba09591d912331ddae56a50
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# runtime-benchmark-comparison

## Summary

A benchmarking pattern for comparing a proposed annotation tool against established baselines on identical scRNA-seq inputs, recording accuracy metrics (precision, recall, F1, percentage of recovered cells) and wall-clock runtimes, as done for the Cell Marker Accordion versus SCEVAN and CopyKAT in cancer datasets.

## When to use

Use when a new single-cell annotation or cell-calling tool must be validated against baseline tools on the same datasets, with per-cell predictions scored against ground-truth labels and runtimes recorded on identical hardware.

## When NOT to use

- No high-quality ground-truth annotations exist for the test dataset — the article notes the availability of high-quality, ground-truth datasets constrains benchmark expansion
- The candidate tool is not marker-based and comparison against marker-based tools is the goal — the article states a direct comparison between non-marker-based annotation approaches and the Cell Marker Accordion was not conducted

## Inputs

- Published glioblastoma scRNA-seq dataset with author-provided tumor marker list (SOX2, OLIG1, GFAP, S100B)
- Lung adenocarcinoma scRNA-seq dataset with published malignant cell-type annotations
- Marker-based annotation tool (Cell Marker Accordion R package, accordion_disease function with built-in disease marker database)
- Baseline CNV-based tools (SCEVAN, CopyKAT) run with default parameters
- Identical hardware and runtime environment for wall-clock timing

## Outputs

- cancer_benchmark_results.csv with per-cell predictions, precision, recall, F1, and runtimes per tool per cancer type
- Accuracy metrics: % malignant cells recovered (Accordion 100% in GBM), F1 scores (Accordion ~0.97 GBM and 0.84 lung vs 0.52 for both SCEVAN and CopyKAT)
- Wall-clock runtime comparison (<2 min vs >6 h vs >7 days)

## How to apply

Assemble the benchmark datasets — here the published glioblastoma scRNA-seq dataset (ground-truth tumor cells defined as clusters 1, 4, 5, 7, 10, 11, 12, 20, 21 via tumor markers SOX2, OLIG1, GFAP, S100B) and a lung adenocarcinoma dataset with published malignant cell-type annotations. Run each tool with default parameters on identical inputs and hardware: the Cell Marker Accordion via accordion_disease with its built-in disease marker database, and CNV-based baselines SCEVAN and CopyKAT. Score per-cell predictions against ground truth by computing the percentage of correctly identified aberrant cells plus precision, recall, and F1 per tool per cancer type, and record wall-clock runtimes (Accordion <2 min vs SCEVAN >6 h vs CopyKAT >7 days). Compile per-cell predictions, metrics, and runtimes into a single results table (cancer_benchmark_results.csv). Validate by accepting the benchmark only when the candidate tool reproduces the reported F1 (~0.97 GBM; 0.84 lung) and outperforms both CNV-based baselines on accuracy and runtime.

## Related tools

- **Cell Marker Accordion** (Candidate tool under benchmark; flags aberrant/malignant cells via accordion_disease using the built-in disease marker database) — https://github.com/TebaldiLab/cellmarkeraccordion
- **SCEVAN** (CNV-based baseline for malignant cell identification, run with default parameters on identical inputs)
- **CopyKAT** (CNV-based baseline for malignant cell identification, run with default parameters on identical inputs)
- **Seurat** (Standard scRNA-seq preprocessing pipeline applied to the benchmark datasets before annotation)

## Examples

```
predictions <- accordion_disease(seurat_obj)  # Cell Marker Accordion R package: flag aberrant/malignant cells using the built-in disease marker database
```

## Evaluation signals

- Cell Marker Accordion achieves F1 ~0.97 and recovers 100% of ground-truth malignant cells in the glioblastoma benchmark
- Lung adenocarcinoma F1 of 0.84 for Accordion versus 0.52 for both SCEVAN and CopyKAT
- Accordion completes annotation in under 2 minutes versus >6 hours (SCEVAN) and >7 days (CopyKAT) on identical inputs and hardware
- Output CSV contains per-cell predictions plus precision, recall, and F1 for each of the three tools in each cancer type
- Benchmark reproduces the neoplastic endothelial subpopulation call in lung adenocarcinoma

## Limitations

- Benchmark coverage limited to two liquid and two solid tumors; article notes benchmark expansion is planned
- Results are specific to the tested glioblastoma and lung adenocarcinoma datasets and may not transfer to other cancer types
- Direct comparison with annotation tools that do not rely on marker genes was not conducted in the study

## Evidence

- [discussion] Baseline comparison motivation: "Some approaches, such as SCEVAN and CopyKAT, attempt to distinguish malignant cancer cells in single-cell RNA-seq through CNV calling"
- [discussion] Benchmark scope limitation: "our current benchmark, focused on two liquid and two solid tumors, demonstrates improved performance within these specific contexts."
- [methods] Annotation function used for candidate tool: "Users can annotate clusters or cells exploiting the built-in Cell Marker Accordion database with the accordion() function"
- [methods] Standard preprocessing of benchmark datasets: "Single-cell datasets were analyzed employing a standard pre-processing pipeline for single-cell RNA-seq data, using Seurat"
- [readme] Tool purpose from README: "The Cell Marker Accordion enhances annotation accuracy in single-cell and spatial omics datasets across various human and murine tissues."
