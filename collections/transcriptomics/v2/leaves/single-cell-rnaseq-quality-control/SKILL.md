---
name: single-cell-rnaseq-quality-control
description: Use when raw scRNA-seq counts (Seurat object, csv/rds/h5 counts matrix,
  or 10X folder) enter the scWizard workflow and low-quality cells must be filtered
  prior to annotation of TME cell types.
when_to_use_negative:
- Data has already passed QC or was downloaded pre-filtered (e.g. example_data.rds
  testing set already curated) — re-filtering may remove valid cells.
- The goal is tissue-specific cell-type identification (e.g. alveolar cells in lung
  cancer); scWizard's 7-major-type/47-subtype TME model is not designed for this.
- Cell-type annotation or downstream analyses (GSVA, inferCNV, CellphoneDB, Monocle)
  are the goal without QC being needed first.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scWizard
  role: Provides the point-and-click 'Input Data', 'Quality Control', and 'Batch processing'
    GUI modules that implement the QC filtering thresholds and save filtered results
  repo: https://github.com/Dulab2020/scWizard
- name: Seurat
  role: Standardized scRNA-seq pipeline inside scWizard (dimension reduction, clustering)
    and one of the two batch-effect removal options applied before annotation
- name: Harmony
  role: Alternative batch-effect removal method in scWizard's 'Batch processing' module,
    suited to datasets with common cell types across techniques and faster for initial
    exploration of large datasets
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:23834675bfbf34e6493e4c886617edc10666e1f3229e3906458fe84f600f12cc
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
  - Harmony
  - Seurat
  - scWizard
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-cell-rnaseq-quality-control

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quality-control step of the scWizard cancer scRNA-seq pipeline, which removes low-quality cells using default thresholds (<200 genes detected, >10,000 genes detected, >5% mitochondrial reads) before cell annotation and downstream analysis.

## When to use

Use when raw scRNA-seq counts (Seurat object, csv/rds/h5 counts matrix, or 10X folder) enter the scWizard workflow and low-quality cells must be filtered prior to annotation of TME cell types.

## When NOT to use

- Data has already passed QC or was downloaded pre-filtered (e.g. example_data.rds testing set already curated) — re-filtering may remove valid cells.
- The goal is tissue-specific cell-type identification (e.g. alveolar cells in lung cancer); scWizard's 7-major-type/47-subtype TME model is not designed for this.
- Cell-type annotation or downstream analyses (GSVA, inferCNV, CellphoneDB, Monocle) are the goal without QC being needed first.

## Inputs

- Seurat object (.rds) via 'Data-rds' input
- Counts matrix (csv, rds, or h5) with rows = genes and columns = cells via 'Data-counts'
- 10X-format data folder via 'Data-10X'
- Batch/metadata labels for optional Harmony or Seurat integration

## Outputs

- Filtered Seurat object / counts matrix excluding low-quality cells
- Raw-data quality overview plots from the 'view quality' button
- QC-filtered results auto-saved to the data input folder
- Optionally, batch-effect-corrected data (Harmony/Seurat) for downstream annotation

## How to apply

Load the input data via scWizard's 'Input Data' module (formats: 'Data-rds' Seurat object, 'Data-counts' csv/rds/h5 with genes as rows, or 'Data-10X' folder). Use the 'Quality Control' module and first click 'view quality' to inspect the original data quality (genes detected per cell, percent mitochondrial reads). Apply scWizard's default filters, which discard cells with <200 genes detected, >10,000 genes detected, or >5% of reads mapping to mitochondrial RNA; parameters can be adjusted per dataset via the GUI. Optionally run the 'Batch processing' module (Seurat or Harmony integration) before annotation if multiple patients, technologies, or experiments are present. Filtered results are automatically saved to the folder where the data was entered, ready for the deep-neural-network cell annotation step.

## Related tools

- **scWizard** (Provides the point-and-click 'Input Data', 'Quality Control', and 'Batch processing' GUI modules that implement the QC filtering thresholds and save filtered results) — https://github.com/Dulab2020/scWizard
- **Seurat** (Standardized scRNA-seq pipeline inside scWizard (dimension reduction, clustering) and one of the two batch-effect removal options applied before annotation)
- **Harmony** (Alternative batch-effect removal method in scWizard's 'Batch processing' module, suited to datasets with common cell types across techniques and faster for initial exploration of large datasets)

## Examples

```
if (!requireNamespace("devtools", quietly=TRUE)) install.packages("devtools"); devtools::install_github("Dulab2020/scWizard"); scWizard::run_app()
```

## Evaluation signals

- Every retained cell has ≥200 detected genes, ≤10,000 detected genes, and ≤5% mitochondrial reads — no cell violating the defaults remains after QC.
- The 'view quality' inspection on the raw data was run before thresholds were chosen (documented parameter-selection behavior).
- Filtered results are present in the folder where the input data was entered (README-stated save location).
- Downstream annotation assigns every remaining cell a label among the 7 major TME cell types or 47 subtypes (or Unknown for uncovered tissue-specific types), indicating QC did not corrupt the object.
- If multiple batches/patients exist, pre- and post-correction visualizations from the Batch processing module show reduced batch separation before annotation.

## Limitations

- Default thresholds are dataset-agnostic; the README states users should 'select parameters for quality control according to the situation' rather than blindly using defaults.
- Tissue-specific cell types may be dropped or mislabeled later: 'the alveolar cells in lung cancers may not be identified and named in Unknown cluster'.
- Harmony vs Seurat batch correction is a choice point — Harmony suits datasets with common cell types across techniques; neither guarantees removal of strong batch effects.
- No software version number is stated for scWizard, complicating exact reproduction of QC behavior.

## Evidence
