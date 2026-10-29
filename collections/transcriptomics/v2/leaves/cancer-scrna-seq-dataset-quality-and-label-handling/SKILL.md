---
name: cancer-scrna-seq-dataset-quality-and-label-handling
description: Use when a labeled cancer scRNA-seq dataset (e.g., Seurat object or counts
  matrix in csv/rds/h5/10X format) must be quality-controlled, run through scWizard's
  two-step annotation, and its predictions graded against the original study's cell
  labels to compute per-cluster accuracy.
when_to_use_negative:
- The input data are tissue-specific non-TME cell types (e.g., alveolar cells in lung
  cancers) that scWizard is known to miss and name in an Unknown cluster
- The goal is de novo cell-type discovery without labeled reference annotations, rather
  than evaluating or applying scWizard's pretrained hierarchical annotation
- The dataset is not cancer scRNA-seq data from the tumor microenvironment, since
  scWizard's reference covers TME cells across 13 cancer types
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: scWizard
  role: 'Primary tool: provides the two-step hierarchical DNN annotation model, QC,
    batch processing (Seurat/Harmony), and Shiny GUI used to annotate and evaluate
    the independent cancer datasets'
  repo: https://github.com/Dulab2020/scWizard
- name: Seurat
  role: Underlying engine for scWizard's standardized scRNA-seq pipeline (dimension
    reduction, clustering, differential expression) and for subclustering major cell
    types before subtype annotation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:5a2c9ea5747863757166a9cbff5a00f0f0888bd9e593599bf443a003d190ffe9
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
yes_rate: 0.9090909090909091
n_claims: 11
---

# cancer-scrna-seq-dataset-quality-and-label-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A method for preparing and evaluating independent cancer scRNA-seq datasets (MCC, CRC, LC, LYM, T/NK_BRCA, T/NK_KIRC, Mye_PAAD) against scWizard's hierarchical deep-neural-network annotation: quality control, batch-aware loading of labeled cells, per-cluster consistency labeling of predictions, and accuracy computation against original study annotations.

## When to use

Use when a labeled cancer scRNA-seq dataset (e.g., Seurat object or counts matrix in csv/rds/h5/10X format) must be quality-controlled, run through scWizard's two-step annotation, and its predictions graded against the original study's cell labels to compute per-cluster accuracy.

## When NOT to use

- The input data are tissue-specific non-TME cell types (e.g., alveolar cells in lung cancers) that scWizard is known to miss and name in an Unknown cluster
- The goal is de novo cell-type discovery without labeled reference annotations, rather than evaluating or applying scWizard's pretrained hierarchical annotation
- The dataset is not cancer scRNA-seq data from the tumor microenvironment, since scWizard's reference covers TME cells across 13 cancer types

## Inputs

- Independent Merkel cell carcinoma (MCC) dataset with 11,024 labeled cells for major-cell-type evaluation
- CRC myeloid cells dataset (11,717 cells) and LC T cells dataset (45,555 cells) for subtype evaluation
- Comparison datasets: LYM, T/NK_BRCA, T/NK_KIRC, Mye_PAAD
- Counts matrix (csv, rds, or h5) or 10X folder, or a Seurat object (Data-rds/Data-counts/Data-10X)
- scWizard pretrained trainset (H5 format, rows = genes, columns = cells) placed in app/www/python
- Original study's cell-type label metadata for each dataset

## Outputs

- Per-cell annotation labels for 7 major cell types and 47 TME subtypes
- Per-cluster accuracy table (consistent cells / total cells per cluster)
- annotation_accuracy.csv with per-dataset accuracy values for all seven datasets
- QC-filtered Seurat object after removing low-quality cells

## How to apply

Install scWizard via devtools::install_github("Dulab2020/scWizard"), click the 'install Python' button in the Shiny GUI, and download the pretrained trainset from the Baidu link, copying it to the app/www/python folder. Apply scWizard's default QC before annotation: drop cells with <200 genes detected, >10,000 genes detected, or >5% mitochondrial reads; optionally run batch-effect removal (Seurat or Harmony) before annotation when multiple batches are present. Run the two-step model — first major cell-type annotation (PCA on combined training+prediction data, DNN with ReLU and softmax activation, max-likelihood assignment), then subtype annotation (Seurat subclustering of each major cell type at a chosen resolution followed by DNN prediction with majority-vote subcluster rule). Label each predicted cell as 'consistent', 'inconsistent', or 'unknown' against the original study's annotation and compute per-cluster accuracy as consistent cells divided by total cells in each cluster. Assemble per-dataset accuracies (e.g., 0.98 MCC major types, 0.85 CRC myeloid subtypes, 0.79 LC T/NK subtypes) into an annotation_accuracy.csv and verify they match reported values within a few percent.

## Related tools

- **scWizard** (Primary tool: provides the two-step hierarchical DNN annotation model, QC, batch processing (Seurat/Harmony), and Shiny GUI used to annotate and evaluate the independent cancer datasets) — https://github.com/Dulab2020/scWizard
- **Seurat** (Underlying engine for scWizard's standardized scRNA-seq pipeline (dimension reduction, clustering, differential expression) and for subclustering major cell types before subtype annotation)

## Examples

```
if (!requireNamespace("devtools", quietly=TRUE)) install.packages("devtools"); devtools::install_github("Dulab2020/scWizard"); scWizard::run_app()
```

## Evaluation signals

- Reproduced MCC major-cell-type accuracy is 0.98, CRC myeloid-subtype accuracy 0.85, and LC T/NK-subtype accuracy 0.79, within a few percent tolerance of the reported figures
- Comparison-dataset accuracies fall at or above reported values: >0.66 for LYM, 0.71 for T/NK_BRCA, 0.66 for T/NK_KIRC, 0.64 for Mye_PAAD
- QC removes cells with <200 genes, >10,000 genes, or >5% mitochondrial reads, matching scWizard defaults
- Every predicted cell carries exactly one of the labels 'consistent', 'inconsistent', or 'unknown'; per-cluster accuracy equals consistent cells divided by total cells in each cluster
- The annotation_accuracy.csv contains one accuracy value per dataset for all seven datasets (MCC, CRC, LC, LYM, T/NK_BRCA, T/NK_KIRC, Mye_PAAD)

## Limitations

- Tissue-specific cell types (e.g., alveolar cells in lung cancers) may not be identified and are named in the Unknown cluster; tissue-specific subtypes and cellular states are deferred to future updates
- The trainset must be manually downloaded from the Baidu link (extraction code 0ccq) and copied to app/www/python, and the Python environment must be installed via the GUI before annotation
- Works with R v4.0.2 or greater; JAGS must be installed separately (download on Windows, conda install -c conda-forge jags on Linux)

## Evidence

- [methods] By default, scWizard defined cells with <200 genes detected, > 10,000 genes detected, >5 % of reads mapping to mitochondrial RNA as low-quality cells and filtered them out of the downstream analysis.: "By default, scWizard defined cells with <200 genes detected, > 10,000 genes detected, >5 % of reads mapping to mitochondrial RNA as low-quality cells"
