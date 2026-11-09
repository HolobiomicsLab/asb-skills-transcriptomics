---
name: tumor-microenvironment-subtype-classification
description: Use when annotating cells from cancer scRNA-seq data (Seurat object,
  counts matrix in csv/rds/h5, or 10X folder) into tumor-microenvironment major types
  and subtypes, especially T/NK and myeloid subtypes, without marker-based manual
  labeling.
when_to_use_negative:
- Input contains tissue-specific cell types outside the TME panel (e.g., alveolar
  cells in lung cancers), which will be misrouted into the Unknown cluster instead
  of correctly labeled
- Data is single-cell RNA-seq from non-cancer tissue, where the 7-type/47-subtype
  TME taxonomy and cancer-trained reference may not apply
- A fully unsupervised marker-based annotation is required without dependence on a
  pre-trained model or Python environment
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: scWizard
  role: Provides the pre-trained two-step hierarchical deep neural network that classifies
    7 major TME cell types before clustering and 47 subtypes after Seurat subclustering
  repo: https://github.com/Dulab2020/scWizard
- name: Seurat
  role: Used to subcluster each predicted major cell type prior to subtype annotation,
    and for dimension reduction and clustering within scWizard's pipeline
  repo: https://github.com/Dulab2020/scWizard
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:6768da8a7269d24afc29533f2222753c5c5da542f5f868f8f6d7d34fb166ea9d
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
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# tumor-microenvironment-subtype-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classify and annotate single cells from cancer scRNA-seq data into 7 major TME cell types and 47 subtypes using scWizard's two-step hierarchical deep neural network (ReLU hidden layers, softmax output, PCA-reduced input), pre-trained on 113,976 cells across 13 cancer types.

## When to use

Use when annotating cells from cancer scRNA-seq data (Seurat object, counts matrix in csv/rds/h5, or 10X folder) into tumor-microenvironment major types and subtypes, especially T/NK and myeloid subtypes, without marker-based manual labeling.

## When NOT to use

- Input contains tissue-specific cell types outside the TME panel (e.g., alveolar cells in lung cancers), which will be misrouted into the Unknown cluster instead of correctly labeled
- Data is single-cell RNA-seq from non-cancer tissue, where the 7-type/47-subtype TME taxonomy and cancer-trained reference may not apply
- A fully unsupervised marker-based annotation is required without dependence on a pre-trained model or Python environment

## Inputs

- Seurat object (Data-rds)
- Counts matrix in csv, rds, or h5 format (genes as rows, cells as columns)
- 10X data folder
- scWizard training reference H5 file (113,976 cells, 10 datasets, 199 samples, 13 cancer types)

## Outputs

- Per-cell predicted major cell-type labels among 7 TME types (or Unknown)
- Per-subcluster subtype labels among 47 TME subtypes
- UMAP 'single-cell map' colored by predicted major type and subtype
- Cell-type composition summary (fraction of each cluster)

## How to apply

Install the scWizard R package from GitHub (requires R >= 4.0.2 and JAGS) and run `scWizard::run_app()`; before annotation, click 'install Python' and download the H5-format trainset (genes as rows, cells as columns) into the app/www/python folder of the scWizard installation directory. First run Quality Control with the default filters (remove cells with <200 genes, >10,000 genes, or >5% mitochondrial reads) and optionally remove batch effects with Seurat or Harmony. Run the 'Cell annotation' module for major cell types using the pre-trained hierarchical neural network, keeping default hidden-layer nodes or adjusting them per the documented criteria (between input and output sizes, 2/3 input + 2/3 output, or less than twice input size). Then Seurat-subcluster each predicted major cell type and run the subtype annotation step so each subcluster is assigned one of 47 TME subtypes, accepting the predicted subcluster with the largest number of cells per subcluster; cells of tissue-specific types not covered (e.g., alveolar cells) fall into the 'Unknown' cluster. Validate performance against the reference benchmarks: ~0.98 accuracy for major cell types, ~0.85 for myeloid and ~0.79 for T/NK subtypes on independent datasets (MCC 11,024 cells; CRC 11,717 myeloid cells; LC 45,555 T cells).

## Related tools

- **scWizard** (Provides the pre-trained two-step hierarchical deep neural network that classifies 7 major TME cell types before clustering and 47 subtypes after Seurat subclustering) — https://github.com/Dulab2020/scWizard
- **Seurat** (Used to subcluster each predicted major cell type prior to subtype annotation, and for dimension reduction and clustering within scWizard's pipeline) — https://github.com/Dulab2020/scWizard

## Examples

```
devtools::install_github("Dulab2020/scWizard"); scWizard::run_app()
```

## Evaluation signals

- Major cell type annotation accuracy should approximate 0.98 on labeled benchmarks such as the independent MCC dataset of 11,024 cells
- Subtype accuracy should be around 0.85 for myeloid cells (CRC dataset, 11,717 cells) and 0.79 for T/NK cells (LC dataset, 45,555 cells)
- Every cell receives a label among the 7 major types / 47 subtypes, or 'Unknown' for tissue-specific types not covered
- Annotated identities are supported by marker gene expression in the UMAP 'single-cell map'
- In 10-fold cross-validation on the reference, average concordance is ~0.97 for major types with 0.81-0.89 for subtypes; folds deviating strongly from the average should be flagged

## Limitations

- Tissue-specific cell types (e.g., alveolar cells in lung cancers) may not be identified and are named in an Unknown cluster
- Identification of tissue-specific subtypes and cellular states is deferred to future updates
- Custom trainsets must be H5 format with rows as genes and columns as cells, and Python must be installed plus the trainset copied into the app/www/python folder before annotation

## Evidence

- [methods] We designed a 'two-step' annotation model for the cell-type classification, which was applied to individual cells before clustering for major cell types and after clustering for subtype annotation.: [evidence span withheld — anchor status: not_found]
