---
name: scrnaseq-quality-control-filtering
description: Use when a raw counts matrix or Seurat object (csv/rds/h5 or 10X folder)
  enters a scRNA-seq workflow and cells must be filtered for quality before annotation,
  clustering, or gene-set scoring.
when_to_use_negative:
- Input is already a QC-passed, filtered expression matrix and re-filtering would
  remove valid cells (e.g., datasets with naturally low gene counts or high mitochondrial
  content by biology)
- Tissue-specific subtypes or cellular states are the analysis target — scWizard QC
  plus its annotation may leave such cells in an 'Unknown cluster' (e.g., alveolar
  cells in lung cancers)
- The goal is not scRNA-seq cell-level QC but, e.g., batch-effect assessment or copy-number
  inference, which are separate scWizard modules ('Batch processing', 'inferCNV')
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scWizard
  role: Shiny-based GUI whose 'Input Data' and 'Quality Control' modules implement
    threshold-based filtering of low-quality cells with adjustable parameters
  repo: https://github.com/Dulab2020/scWizard
- name: Seurat
  role: Underlying standardized scRNA-seq pipeline in scWizard (dimension reduction,
    clustering, differential expression) that consumes the QC-filtered data
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:a67494d3b968b424df8518fe4e6b005799ae3278d9802442711638064dd8b49d
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
  - GSVA
  - GSVAdata
  - Seurat
  - scWizard
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# scRNA-seq quality control and low-quality cell filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quality control step for single-cell RNA-seq data that filters low-quality cells using scWizard's default thresholds (<200 detected genes, >10,000 detected genes, or >5% mitochondrial read fraction) before downstream analyses such as cell annotation or GSVA scoring.

## When to use

Use when a raw counts matrix or Seurat object (csv/rds/h5 or 10X folder) enters a scRNA-seq workflow and cells must be filtered for quality before annotation, clustering, or gene-set scoring.

## When NOT to use

- Input is already a QC-passed, filtered expression matrix and re-filtering would remove valid cells (e.g., datasets with naturally low gene counts or high mitochondrial content by biology)
- Tissue-specific subtypes or cellular states are the analysis target — scWizard QC plus its annotation may leave such cells in an 'Unknown cluster' (e.g., alveolar cells in lung cancers)
- The goal is not scRNA-seq cell-level QC but, e.g., batch-effect assessment or copy-number inference, which are separate scWizard modules ('Batch processing', 'inferCNV')

## Inputs

- Seurat object ('Data-rds' input format)
- Counts matrix with genes as rows and cells as columns, in csv, rds, or h5 format ('Data-counts')
- Folder containing 10X-formatted data ('Data-10X')

## Outputs

- QC-filtered cell-by-gene matrix / Seurat object with low-quality cells removed, saved in the data input folder
- Original data quality view (via the 'view quality' button) used to guide threshold selection

## How to apply

Load the scRNA-seq data into scWizard via the 'Input Data' module, which accepts a Seurat object ('Data-rds') or a counts matrix in csv, rds, or h5 format, or a 10X folder. Use the 'Quality Control' module and click the 'view quality' button to inspect original data quality before choosing thresholds. By default, scWizard filters cells with <200 genes detected, >10,000 genes detected, or >5% of reads mapping to mitochondrial RNA, removing them from downstream analysis; users may adjust these parameters according to the dataset. Results are automatically saved to the folder where the data was entered. Only QC-passed cells should proceed to Seurat-based clustering, neural-network cell annotation, or GSVA pathway scoring so that low-quality cells do not distort cell-type or angiogenesis-score distributions.

## Related tools

- **scWizard** (Shiny-based GUI whose 'Input Data' and 'Quality Control' modules implement threshold-based filtering of low-quality cells with adjustable parameters) — https://github.com/Dulab2020/scWizard
- **Seurat** (Underlying standardized scRNA-seq pipeline in scWizard (dimension reduction, clustering, differential expression) that consumes the QC-filtered data)

## Examples

```
scWizard::run_app()  # then use the 'Input Data' and 'Quality Control' modules with default filters: genes < 200, genes > 10000, mitochondrial reads > 5%
```

## Evaluation signals

- Per-cell metrics are computed for every cell in the input: detected genes per cell, total detected genes, and fraction of reads mapping to mitochondrial RNA
- Filtered dataset contains no cells with <200 detected genes, no cells with >10,000 detected genes, and no cells with >5% mitochondrial read fraction (or the user-adjusted equivalents)
- QC results are automatically written to the folder where the input data was entered, confirming the step completed
- Downstream steps (e.g., GSVA angiogenesis scores) are computed for every cell in the QC-passed dataset with no missing cells
- The 'view quality' inspection was performed on the original data before thresholds were selected, so the chosen parameters match the dataset's quality distribution

## Limitations

- Default thresholds (<200 genes, >10,000 genes, >5% mitochondrial reads) may be inappropriate for some tissues; parameters are user-adjustable but must be tuned per dataset
- scWizard may not identify tissue-specific cell types (e.g., alveolar cells in lung cancers), which land in an 'Unknown cluster' even after QC; improved tissue-specific subtype identification is deferred to future updates

## Evidence
