---
name: malignant-cell-identification-scrna-seq
description: Use when scRNA-seq data from a tumor sample (e.g., glioblastoma, lung
  adenocarcinoma) must be screened for aberrant/malignant cells and marker-based annotation
  must be compared to CNV-inference baselines.
when_to_use_negative:
- The goal is normal/healthy cell-type annotation rather than aberrant or malignant
  cell identification (use the standard accordion() function instead)
- No ground-truth malignant cell annotations or tumor marker list is available to
  validate calls, precluding F1-based benchmarking
- Direct comparison with non-marker-based annotation tools is required — the article
  states no such comparison was conducted in this study
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cell Marker Accordion
  role: Marker-based identification of aberrant/malignant cells via accordion_disease
    using the built-in disease marker database
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
- name: SCEVAN
  role: CNV-based malignant cell calling baseline run with default parameters
- name: CopyKAT
  role: CNV-based malignant cell calling baseline run with default parameters
- name: Seurat
  role: Standard pre-processing pipeline for the published single-cell datasets prior
    to annotation
- name: R
  role: Environment for running the Cell Marker Accordion R package and benchmarking
    scripts
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:a16533caba39b0ac127c6c0d3e434dc82a755b50d9590d909778eae8bcc903f0
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# malignant-cell-identification-scrna-seq

## Summary

Benchmarked skill for identifying aberrant/malignant tumor cells in scRNA-seq by comparing marker-based annotation via the Cell Marker Accordion (accordion_disease) against CNV-based callers SCEVAN and CopyKAT, using F1 score and wall-clock runtime.

## When to use

Use when scRNA-seq data from a tumor sample (e.g., glioblastoma, lung adenocarcinoma) must be screened for aberrant/malignant cells and marker-based annotation must be compared to CNV-inference baselines.

## When NOT to use

- The goal is normal/healthy cell-type annotation rather than aberrant or malignant cell identification (use the standard accordion() function instead)
- No ground-truth malignant cell annotations or tumor marker list is available to validate calls, precluding F1-based benchmarking
- Direct comparison with non-marker-based annotation tools is required — the article states no such comparison was conducted in this study

## Inputs

- Published glioblastoma scRNA-seq dataset with author-provided tumor marker list (SOX2, OLIG1, GFAP, S100B) and ground-truth tumor clusters
- Lung adenocarcinoma scRNA-seq dataset with published malignant cell-type annotations
- Pre-processed Seurat objects (standard scRNA-seq pre-processing, cells filtered 200-6000 expressed genes, <20% mitochondrial counts)
- Built-in Cell Marker Accordion disease marker database (15479 marker genes, 728 cell types, 212 human tissues)

## Outputs

- cancer_benchmark_results.csv with per-cell predictions, precision/recall/F1 metrics, and runtimes per tool
- Per-cell malignant/aberrant calls from accordion_disease, SCEVAN, and CopyKAT
- Neoplastic subpopulation calls (e.g., neoplastic endothelial cells)

## How to apply

Define ground-truth malignant cells from author-provided annotations or tumor marker lists (e.g., SOX2, OLIG1, GFAP, S100B in the glioblastoma dataset defining clusters 1, 4, 5, 7, 10, 11, 12, 20, 21). Run the Cell Marker Accordion via the accordion_disease function to flag aberrant cells using the built-in disease marker database, then run SCEVAN and CopyKAT with default parameters on identical inputs for CNV-based malignant cell calling. Compute the percentage of correctly identified aberrant cells plus precision, recall, and F1 against ground truth per tool and cancer type, and record wall-clock runtimes on identical hardware. Validation thresholds come from the paper: Cell Marker Accordion F1 ~0.97 in glioblastoma (100% malignant cells recovered) and 0.84 in lung adenocarcinoma, versus 0.52 for both SCEVAN and CopyKAT, with runtimes under 2 minutes versus more than 6 hours (SCEVAN) and more than 7 days (CopyKAT).

## Related tools

- **Cell Marker Accordion** (Marker-based identification of aberrant/malignant cells via accordion_disease using the built-in disease marker database) — https://github.com/TebaldiLab/cellmarkeraccordion
- **SCEVAN** (CNV-based malignant cell calling baseline run with default parameters)
- **CopyKAT** (CNV-based malignant cell calling baseline run with default parameters)
- **Seurat** (Standard pre-processing pipeline for the published single-cell datasets prior to annotation)
- **R** (Environment for running the Cell Marker Accordion R package and benchmarking scripts)

## Examples

```
accordion_disease(seurat_obj, tissue="lung", cancer="LUAD")  # flag aberrant cells; benchmark vs SCEVAN/CopyKAT F1 against ground-truth labels
```

## Evaluation signals

- Cell Marker Accordion achieves F1 ~0.97 with 100% malignant cell recovery in glioblastoma against ground-truth clusters (1, 4, 5, 7, 10, 11, 12, 20, 21)
- In lung adenocarcinoma, Cell Marker Accordion reaches F1 0.84 versus 0.52 for both SCEVAN and CopyKAT
- Runtimes on identical inputs/hardware: Accordion <2 minutes, SCEVAN >6 hours, CopyKAT >7 days
- cancer_benchmark_results.csv contains per-cell predictions plus precision, recall, F1, and runtime columns for all three tools
- The neoplastic endothelial subpopulation is recovered among the aberrant cell calls

## Limitations

- Cancer benchmark limited to two liquid and two solid tumors; benchmark expansion planned
- Limited availability of high-quality ground-truth datasets constrains benchmark expansion
- No direct comparison conducted with annotation tools that do not rely on marker genes

## Evidence

- [discussion] CNV-baseline comparison claim: "Some approaches, such as SCEVAN and CopyKAT, attempt to distinguish malignant cancer cells in single-cell RNA-seq through CNV calling"
- [methods] accordion() annotation function: "Users can annotate clusters or cells exploiting the built-in Cell Marker Accordion database with the accordion() function"
- [methods] pre-processing and filtering thresholds: "To ensure the removal of empty droplets and potential doublets, cells with less than 200 expressed genes and more than 6000 were filtered out"
- [methods] database scale: "Cell Marker Accordion includes a comprehensive collection of 15479 marker genes associated with 728 cell types in 212 human tissues"
- [readme] README platform scope: "it can identify disease-critical cells and pathological processes, helping to extract potential biomarkers across diverse disease contexts."
