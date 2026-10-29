---
name: single-cell-rna-seq-cell-type-annotation
description: Use when annotating cells from cancer scRNA-seq data (counts matrix,
  Seurat object, h5, or 10X folder) into tumor microenvironment major cell types and
  subtypes, or when benchmarking annotation accuracy against original study labels.
when_to_use_negative:
- The data are tissue-specific non-TME cell types — e.g., alveolar cells in lung cancers
  may not be identified and land in the 'Unknown' cluster
- The goal is annotation of non-cancer scRNA-seq outside the tumor microenvironment
  scope of the built-in 13-cancer-type reference
- You already have trusted manual/canonical-marker annotations and only need downstream
  analyses (GSVA, inferCNV, Monocle, CellphoneDB) rather than cell-type classification
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: scWizard
  role: Provides the hierarchical deep neural network (ReLU + softmax) two-step classifier
    for major cell types and 47 TME subtypes, plus the Shiny GUI, custom trainset
    option, and Python environment installer
  repo: https://github.com/Dulab2020/scWizard
- name: Seurat
  role: Used within scWizard for subclustering each predicted major cell type (dimension
    reduction, clustering) prior to the subtype-annotation step
  repo: https://github.com/Dulab2020/scWizard
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:d20eb032b809218089ff777d91d2e898829316b4ca75046c4b70a723af1e13f0
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
yes_rate: 0.7272727272727273
n_claims: 11
---

# single-cell-rna-seq-cell-type-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automated cell-type annotation of cancer scRNA-seq data using scWizard's hierarchical two-step deep neural network classifier, which assigns cells to 7 major tumor-microenvironment cell types and 47 cell subtypes using a pre-trained reference of 113,976 cells across 13 cancer types.

## When to use

Use when annotating cells from cancer scRNA-seq data (counts matrix, Seurat object, h5, or 10X folder) into tumor microenvironment major cell types and subtypes, or when benchmarking annotation accuracy against original study labels.

## When NOT to use

- The data are tissue-specific non-TME cell types — e.g., alveolar cells in lung cancers may not be identified and land in the 'Unknown' cluster
- The goal is annotation of non-cancer scRNA-seq outside the tumor microenvironment scope of the built-in 13-cancer-type reference
- You already have trusted manual/canonical-marker annotations and only need downstream analyses (GSVA, inferCNV, Monocle, CellphoneDB) rather than cell-type classification

## Inputs

- Counts matrix (csv, rds, or h5; rows = genes, columns = cells), Seurat object, or 10X-format folder
- scWizard pre-trained trainset in H5 format (113,976 cells, 199 samples, 13 cancer types) from https://pan.baidu.com/s/1_J_17OKRLo4BQB33C_Ux3Q
- Optional: custom trainset (H5) and custom trainset labels for re-training
- Labeled independent evaluation datasets (e.g., MCC 11,024 cells; CRC 11,717 myeloid cells; LC 45,555 T cells)

## Outputs

- Major cell-type labels per cell (endothelial, myeloid, T/NK, stromal, etc.)
- Cell subtype annotations (47 subtypes) after Seurat subclustering of each major type
- Per-cluster accuracy metrics (consistent/inconsistent/unknown counts) e.g. annotation_accuracy.csv
- Cross-validation accuracy metrics e.g. cv_accuracy.csv

## How to apply

Install scWizard from GitHub with devtools::install_github('Dulab2020/scWizard'), click 'install Python' to set up the Python environment, and copy the H5-format trainset (rows = genes, columns = cells) from the Baidu link into the app/www/python folder. Apply the two-step model: step one performs PCA on the combined training+prediction data and uses a DNN (ReLU and softmax activations, max-likelihood assignment) to label major cell types; step two runs Seurat subclustering of each predicted major cell type, then a DNN with a majority-vote subcluster rule annotates 47 subtypes. Hidden-layer node counts can be adjusted (hidden neurons should be between input and output layer sizes, ideally 2/3 of input plus 2/3 of output, and less than twice the input size) or defaults used. Validate by labeling each predicted cell as 'consistent', 'inconsistent', or 'unknown' against original study annotations and computing per-cluster accuracy; reported performance is 0.98 for major cell types (MCC), 0.85 for CRC myeloid subtypes, and 0.79 for LC T/NK subtypes, with ten-fold cross-validation concordance of ~0.97-0.98 for major types.

## Related tools

- **scWizard** (Provides the hierarchical deep neural network (ReLU + softmax) two-step classifier for major cell types and 47 TME subtypes, plus the Shiny GUI, custom trainset option, and Python environment installer) — https://github.com/Dulab2020/scWizard
- **Seurat** (Used within scWizard for subclustering each predicted major cell type (dimension reduction, clustering) prior to the subtype-annotation step) — https://github.com/Dulab2020/scWizard

## Examples

```
devtools::install_github("Dulab2020/scWizard"); scWizard::run_app()
```

## Evaluation signals

- Ten-fold cross-validation on the built-in reference should yield average concordance ~0.97 for major cell types; flag any fold deviating strongly from the average
- Independent-dataset accuracy should approximate reported values within a few percent: 0.98 (MCC major), 0.85 (CRC myeloid subtypes), 0.79 (LC T/NK subtypes), 0.71 (T/NK_BRCA), 0.66 (T/NK_KIRC), 0.64 (Mye_PAAD)
- Per-cluster accuracy equals number of 'consistent' cells divided by total cells in each cluster, with every cell labeled consistent/inconsistent/unknown against original study annotations
- Trainset file is H5 format with rows as genes and columns as cells, placed in app/www/python under the scWizard installation directory
- Hidden-layer node counts satisfy the stated criteria (between input and output sizes; 2/3 input + 2/3 output; less than twice input size) when non-default parameters are used

## Limitations

- Tissue-specific cell types (e.g., alveolar cells in lung cancers) may not be identified and are named in the Unknown cluster; tissue-specific subtypes and cellular states are deferred to future updates
- The training reference covers 7 major cell types and 47 subtypes across 13 cancer types; annotation of cell types outside this TME taxonomy is unsupported (custom trainsets can be supplied instead)
- The Python environment and downloaded trainset must be installed before annotation; trainset access is via a Baidu link with extraction code 0ccq

## Evidence

- [abstract] scWizard gives higher accuracy for annotating subtypes of tumor-derived T/NK cells and myeloid cells: [evidence span withheld — anchor status: not_found]
- [methods] two-step annotation model: [evidence span withheld — anchor status: not_found]
