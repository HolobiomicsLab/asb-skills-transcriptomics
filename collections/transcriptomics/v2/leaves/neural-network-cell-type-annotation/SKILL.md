---
name: neural-network-cell-type-annotation
description: Use when annotating single cells from cancer scRNA-seq data (Seurat object,
  counts matrix csv/rds/h5, or 10X folder) into tumor microenvironment cell types
  without marker-gene curation or programming.
when_to_use_negative:
- Tissue-specific cell types outside the TME reference (e.g., alveolar cells in lung
  cancers) are the target — scWizard may assign them to an Unknown cluster
- Input is not cancer/TME scRNA-seq but another data modality (feature tables, bulk
  RNA-seq) with no per-cell expression matrix
- The goal is de novo discovery of novel cellular states rather than classification
  into the fixed 7 major / 47 subtype reference labels
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: scWizard
  role: Point-and-click shiny GUI packaging the hierarchical deep neural network annotation
    model, quality control, and Seurat-based clustering used for the two-step annotation
  repo: https://github.com/Dulab2020/scWizard
- name: Seurat
  role: Provides the standardized scRNA-seq pipeline (dimension reduction, cell clustering,
    differential expression) used between major-type and subtype annotation steps
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:3de396a186c294b55fd023da180a570cd9b7be3d1c8ce6ff9f4ec60ef47afd75
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Dulab2020/scWizard
  repo_url: https://github.com/Dulab2020/scWizard
  tools:
  - Seurat
  - scWizard
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# neural-network-cell-type-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automated cell-type annotation of cancer scRNA-seq data using scWizard's hierarchical deep neural network (ReLU hidden layers, softmax output, PCA-reduced input), pre-trained on 113,976 cells across 13 cancer types to classify 7 major TME cell types and 47 subtypes.

## When to use

Use when annotating single cells from cancer scRNA-seq data (Seurat object, counts matrix csv/rds/h5, or 10X folder) into tumor microenvironment cell types without marker-gene curation or programming.

## When NOT to use

- Tissue-specific cell types outside the TME reference (e.g., alveolar cells in lung cancers) are the target — scWizard may assign them to an Unknown cluster
- Input is not cancer/TME scRNA-seq but another data modality (feature tables, bulk RNA-seq) with no per-cell expression matrix
- The goal is de novo discovery of novel cellular states rather than classification into the fixed 7 major / 47 subtype reference labels

## Inputs

- Seurat object (Data-rds)
- Counts matrix in csv, rds, or h5 format with genes as rows and cells as columns (Data-counts)
- 10X data folder (Data-10X)
- Optional custom trainset in H5 format (genes = rows, cells = columns)

## Outputs

- Per-cell predicted major cell-type labels among 7 TME types (or Unknown)
- Per-subcluster subtype labels among 47 TME cell subtypes
- UMAP 'single-cell map' colored by predicted major type and subtype
- Cell-type composition summary (fraction of each cluster)

## How to apply

Run the scWizard GUI via `scWizard::run_app()` and load data through 'Input Data' (Data-rds for Seurat objects, Data-counts for csv/rds/h5 matrices, Data-10X for 10X folders); first run Quality Control with default filters (remove cells with <200 genes, >10,000 genes, or >5% mitochondrial reads) and optionally batch-correct with Seurat or Harmony. Apply the 'Cell annotation' module as a two-step model: classify individual cells into 7 major TME cell types before clustering using the pre-trained network, then Seurat-cluster each major type and run subcluster annotation into one of 47 subtypes, accepting the predicted subcluster with the largest cell count per subcluster. Keep the default ReLU/softmax network with hidden nodes per the documented criteria (between input and output layer sizes, 2/3 input + 2/3 output, or less than twice input size); custom trainsets must be H5 format (genes as rows, cells as columns), and Python must be installed via the 'install Python' button with the trainset copied to app/www/python. Label any tissue-specific cells not covered by the reference as Unknown.

## Related tools

- **scWizard** (Point-and-click shiny GUI packaging the hierarchical deep neural network annotation model, quality control, and Seurat-based clustering used for the two-step annotation) — https://github.com/Dulab2020/scWizard
- **Seurat** (Provides the standardized scRNA-seq pipeline (dimension reduction, cell clustering, differential expression) used between major-type and subtype annotation steps)

## Examples

```
devtools::install_github("Dulab2020/scWizard"); scWizard::run_app()
```

## Evaluation signals

- Every cell receives a label among the 7 major TME cell types or 47 subtypes, or is assigned Unknown for tissue-specific types not covered
- Major-type annotation accuracy on labeled benchmark data (e.g., the 11,024-cell MCC dataset) should approach the reported 0.98
- Subtype annotation robustness on myeloid (~0.85) and T/NK (~0.79) benchmarks should be reproduced when re-running reference evaluations
- Each subcluster is assigned the predicted subtype with the largest number of cells, and annotations are consistent with marker gene expression
- QC-filtered input retains cells passing <200 genes, >10,000 genes, and >5% mitochondrial read thresholds before annotation

## Limitations

- Tissue-specific cell types, such as alveolar cells in lung cancers, may not be identified and are named in the Unknown cluster
- Identification of tissue-specific subtypes and cellular states is deferred to future updates of scWizard
- No software version number is stated for scWizard in the article
- Annotation requires the Python environment and the trainset (Baidu link, extraction code 0ccq) copied into the app/www/python folder of the installation directory

## Evidence
