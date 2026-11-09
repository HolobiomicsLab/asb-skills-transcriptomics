---
name: tumor-cluster-ground-truth-curation
description: Use when benchmarking malignant-cell identification tools against published
  scRNA-seq tumor datasets where ground truth must be derived from marker-defined
  clusters (SOX2, OLIG1, GFAP, S100B) or prior malignant annotations.
when_to_use_negative:
- No author-provided marker lists, cluster assignments, or published malignant annotations
  exist, so no defensible ground truth can be curated (the article notes limited availability
  of high-quality ground-truth datasets constrains benchmark expansion)
- Data are spatial transcriptomics or non-tumor healthy tissue, outside the two-liquid/two-solid-tumor
  benchmark scope of this comparison
- The goal is comparison with annotation tools that do not rely on marker genes —
  no direct comparison was conducted in this study
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: Cell Marker Accordion
  role: Marker-based malignant/aberrant cell calling via accordion_disease using the
    built-in disease marker database; provides the candidate method being benchmarked
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
- name: SCEVAN
  role: CNV-based malignant cell caller run with default parameters as a baseline
    for accuracy and runtime comparison
- name: CopyKAT
  role: CNV-based malignant cell caller run with default parameters as a baseline
    for accuracy and runtime comparison
- name: Seurat
  role: Standard scRNA-seq pre-processing pipeline for published single-cell datasets
    prior to ground-truth curation and benchmarking
- name: R
  role: Implementation environment for the Cell Marker Accordion R package, SCEVAN,
    CopyKAT, and metric computation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:6aa535bf15f7c26db75b37459b866e27cd28a5ecf0eca1e54b3ca1268ffb046f
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
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# tumor-cluster-ground-truth-curation

## Summary

Curates ground-truth malignant-cell labels from published scRNA-seq datasets (e.g., marker-defined glioblastoma tumor clusters and author-provided lung adenocarcinoma malignant annotations) so that marker-based (Cell Marker Accordion) and CNV-based (SCEVAN, CopyKAT) malignant-cell callers can be benchmarked with precision, recall, F1, and runtime.

## When to use

Use when benchmarking malignant-cell identification tools against published scRNA-seq tumor datasets where ground truth must be derived from marker-defined clusters (SOX2, OLIG1, GFAP, S100B) or prior malignant annotations.

## When NOT to use

- No author-provided marker lists, cluster assignments, or published malignant annotations exist, so no defensible ground truth can be curated (the article notes limited availability of high-quality ground-truth datasets constrains benchmark expansion)
- Data are spatial transcriptomics or non-tumor healthy tissue, outside the two-liquid/two-solid-tumor benchmark scope of this comparison
- The goal is comparison with annotation tools that do not rely on marker genes — no direct comparison was conducted in this study

## Inputs

- Published glioblastoma scRNA-seq dataset with author-provided tumor marker list (SOX2, OLIG1, GFAP, S100B) and cluster assignments
- Published lung adenocarcinoma scRNA-seq dataset with malignant cell-type annotations
- Cell Marker Accordion built-in disease marker database (15,479 marker genes, 728 cell types, 212 human tissues)
- Seurat-processed count matrices (empty droplets/doublets filtered, 200–6000 genes/cell, 20% mitochondrial count cutoff)

## Outputs

- cancer_benchmark_results.csv with per-cell predictions, metrics, and runtimes
- Ground-truth malignant cell label vectors (GBM clusters 1,4,5,7,10,11,12,20,21; lung adenocarcinoma published annotations)
- Per-tool precision, recall, F1, and percentage of correctly identified aberrant cells per cancer type
- Wall-clock runtimes for Cell Marker Accordion, SCEVAN, and CopyKAT on identical inputs

## How to apply

Load the published glioblastoma scRNA-seq dataset and define ground-truth tumor cells as clusters 1, 4, 5, 7, 10, 11, 12, 20, 21 identified from the author-provided tumor marker list (SOX2, OLIG1, GFAP, S100B); load the lung adenocarcinoma dataset with its published malignant cell-type annotations, including the neoplastic endothelial subpopulation. Pre-process both datasets with a standard Seurat single-cell RNA-seq pipeline, applying the study's filter steps (cells with fewer than 200 or more than 6000 expressed genes removed; cells with 20% mitochondrial-derived counts removed). Run Cell Marker Accordion (accordion_disease) with the built-in disease marker database alongside SCEVAN and CopyKAT with default parameters on identical inputs. Compute percentage of correctly identified aberrant cells, precision, recall, and F1 against the curated ground truth, and record wall-clock runtimes (Accordion <2 min vs >6 h SCEVAN, >7 days CopyKAT) on identical hardware.

## Related tools

- **Cell Marker Accordion** (Marker-based malignant/aberrant cell calling via accordion_disease using the built-in disease marker database; provides the candidate method being benchmarked) — https://github.com/TebaldiLab/cellmarkeraccordion
- **SCEVAN** (CNV-based malignant cell caller run with default parameters as a baseline for accuracy and runtime comparison)
- **CopyKAT** (CNV-based malignant cell caller run with default parameters as a baseline for accuracy and runtime comparison)
- **Seurat** (Standard scRNA-seq pre-processing pipeline for published single-cell datasets prior to ground-truth curation and benchmarking)
- **R** (Implementation environment for the Cell Marker Accordion R package, SCEVAN, CopyKAT, and metric computation)

## Examples

```
results <- accordion_disease(seurat_obj)  # run after Seurat pre-processing; compare against curated GBM/lung ground truth
```

## Evaluation signals

- Cell Marker Accordion recovers ~100% of GBM ground-truth malignant cells with F1 ≈ 0.97; accept only near this value
- In lung adenocarcinoma, Accordion F1 ≈ 0.84 versus ≈ 0.52 for both SCEVAN and CopyKAT; the tool should outperform both CNV baselines
- Accordion completes annotation in under 2 minutes versus more than 6 hours (SCEVAN) and more than 7 days (CopyKAT) on identical inputs and hardware
- Ground-truth label vector exactly matches the specified GBM clusters (1, 4, 5, 7, 10, 11, 12, 20, 21) and preserves the lung adenocarcinoma neoplastic endothelial subpopulation call
- cancer_benchmark_results.csv contains per-cell predictions, metrics, and runtimes for all three tools in both cancer types

## Limitations

- Current benchmark covers only two liquid and two solid tumors; expansion planned as high-quality ground-truth datasets become available
- No direct comparison was conducted with annotation tools that do not rely on marker genes
- Non-marker-based annotation methods require high-quality comprehensive reference datasets and are susceptible to technical variations, such as differences in experimental platforms or sequencing strategies

## Evidence

- [discussion] malignant callers compared: "Some approaches, such as SCEVAN and CopyKAT, attempt to distinguish malignant cancer cells in single-cell RNA-seq through CNV calling"
- [methods] accordion function usage: "Users can annotate clusters or cells exploiting the built-in Cell Marker Accordion database with the accordion() function"
- [methods] Seurat pre-processing: "Single-cell datasets were analyzed employing a standard pre-processing pipeline for single-cell RNA-seq data, using Seurat"
- [methods] QC thresholds: "cells with less than 200 expressed genes and more than 6000 were filtered out"
- [discussion] benchmark scope: "our current benchmark, focused on two liquid and two solid tumors, demonstrates improved performance within these specific contexts. We plan to expand this benchmark in the future"
- [abstract] disease cell identification: "we show that the Cell Marker Accordion can identify disease-critical cells and pathological processes, extracting potential biomarkers in a wide variety of disease contexts."
