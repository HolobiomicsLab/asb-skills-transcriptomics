---
name: deep-neural-network-classification-of-tumor-microenvironment-cells
description: Use when annotating unlabeled cancer scRNA-seq cells into TME major cell
  types or T/NK and myeloid subtypes without retraining a model, and when benchmarking
  annotation accuracy on independent labeled datasets.
when_to_use_negative:
- Input contains tissue-specific cell types (e.g., alveolar cells in lung cancers)
  that are outside the 47 TME subtypes — these will land in the 'Unknown' cluster
- The goal is terminal cell-type assignment without respecting hierarchical major-type/subtype
  relationships
- Data is not cancer/TME scRNA-seq and no custom trainset is available for model retraining
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scWizard
  role: Shiny-based R package implementing the two-step DNN annotation model (PCA
    + ReLU/softmax DNN for major types; Seurat subclustering + DNN majority-vote for
    subtypes) with the 113,976-cell pre-trained reference
  repo: https://github.com/Dulab2020/scWizard
- name: Seurat
  role: Used inside the second annotation step for subclustering each major cell type
    before DNN subtype prediction
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:2f28d5c1421885d6de0362b0e3eca07cfb7774bf0ddd16060be78dc96b7df3ff
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# deep-neural-network-classification-of-tumor-microenvironment-cells

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Two-step hierarchical deep neural network annotation (as implemented in scWizard) that classifies cancer scRNA-seq cells into 7 major cell types and 47 tumor-microenvironment (TME) subtypes using a pre-trained reference of 113,976 cells across 13 cancer types.

## When to use

Use when annotating unlabeled cancer scRNA-seq cells into TME major cell types or T/NK and myeloid subtypes without retraining a model, and when benchmarking annotation accuracy on independent labeled datasets.

## When NOT to use

- Input contains tissue-specific cell types (e.g., alveolar cells in lung cancers) that are outside the 47 TME subtypes — these will land in the 'Unknown' cluster
- The goal is terminal cell-type assignment without respecting hierarchical major-type/subtype relationships
- Data is not cancer/TME scRNA-seq and no custom trainset is available for model retraining

## Inputs

- Cancer scRNA-seq counts matrix or Seurat object (csv, rds, h5, or 10X folder format)
- Pre-trained scWizard trainset in H5 format (rows = genes, columns = cells; from Baidu link)
- Optional custom trainset (H5) with label file for user-supplied model training

## Outputs

- Per-cell major cell-type annotations (7 major TME cell types)
- Per-cell subtype annotations (47 cell subtypes, via Seurat subclustering + DNN majority vote)
- Per-cluster annotation accuracy table (consistent/inconsistent/unknown labels; e.g., annotation_accuracy.csv)

## How to apply

Install scWizard from GitHub with devtools::install_github("Dulab2020/scWizard"), click the 'install Python' button, and download the pre-trained trainset (Baidu link, extraction code 0ccq) into the app/www/python folder. Run the first annotation step on the combined training+prediction data: PCA followed by a DNN with ReLU and softmax activation, assigning each cell by max-likelihood. For subtype annotation, run the second step: Seurat subclustering of each major cell type at a chosen resolution, then DNN prediction with a majority-vote subcluster rule; hidden-layer nodes can be adjusted between input and output layer sizes (e.g., 2/3 input + 2/3 output, less than 2x input). Label each predicted cell as 'consistent', 'inconsistent', or 'unknown' against original study annotations and compute per-cluster accuracy as consistent cells over total cells in each cluster. Validate by comparing reproduced accuracies against reported values: 0.98 (MCC major types), 0.85 (CRC myeloid subtypes), 0.79 (LC T/NK subtypes), within a few percent tolerance.

## Related tools

- **scWizard** (Shiny-based R package implementing the two-step DNN annotation model (PCA + ReLU/softmax DNN for major types; Seurat subclustering + DNN majority-vote for subtypes) with the 113,976-cell pre-trained reference) — https://github.com/Dulab2020/scWizard
- **Seurat** (Used inside the second annotation step for subclustering each major cell type before DNN subtype prediction)

## Examples

```
if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools"); devtools::install_github("Dulab2020/scWizard"); scWizard::run_app()
```

## Evaluation signals

- Reproduced major-cell-type accuracy on the independent MCC dataset (11,024 labeled cells) is ~0.98
- Reproduced myeloid subtype accuracy on the CRC dataset (11,717 cells) is ~0.85
- Reproduced T/NK subtype accuracy on the LC dataset (45,555 cells) is ~0.79
- Accuracy on comparison datasets falls in expected ranges: >0.66 (LYM), 0.71 (T/NK_BRCA), 0.66 (T/NK_KIRC), 0.64 (Mye_PAAD), within a few percent tolerance
- Every predicted cell carries a 'consistent', 'inconsistent', or 'unknown' label and per-cluster accuracy = consistent cells / total cells per cluster

## Limitations

- Tissue-specific cell types (e.g., alveolar cells in lung cancers) may not be identified and are named in the Unknown cluster
- Identification of tissue-specific subtypes and cellular states is deferred to future updates
- Python environment must be installed via the 'install Python' button and the trainset manually placed in app/www/python before annotation works

## Evidence
