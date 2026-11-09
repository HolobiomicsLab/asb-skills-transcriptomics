---
name: scrna-seq-annotation-integration
description: Use when a cancer scRNA-seq dataset (counts matrix, Seurat object, csv/rds/h5,
  or 10X folder) needs automated cell-type annotation within the tumor microenvironment
  and batch-effect-corrected integration before downstream analyses like inferCNV.
when_to_use_negative:
- The dataset contains tissue-specific cell types (e.g., alveolar cells in lung cancers)
  — scWizard may leave them as 'Unknown' and other annotation strategies are needed.
- The analysis goal is not cancer/TME single-cell annotation, e.g., already-annotated
  data or non-scRNA-seq feature tables.
- Only 300-cell or exploratory toy data without batch structure and without need for
  cell-type labels.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: scWizard
  role: Point-and-click shiny package providing QC, Seurat/Harmony batch integration,
    and the two-step deep-neural-network cell annotation model
  repo: https://github.com/Dulab2020/scWizard
- name: Seurat
  role: Standardized scRNA-seq pipeline inside scWizard (dimension reduction, clustering,
    differential expression) and one of the two batch-effect removal methods
- name: Harmony
  role: Alternative batch-effect removal method, suited to datasets with common cell
    types across techniques and faster runtime for large-dataset exploration
- name: infercnv
  role: 'Downstream consumer of the annotated Seurat object: malignant epithelial
    cells vs non-malignant populations are used as observation/reference sets for
    large-scale CNA analysis'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:b33cd6752a2d111546e45e1153e9a2fbe5db4d97951436915601629902166fed
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
  - infercnv
  - scWizard
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# scrna-seq-annotation-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies scWizard's deep neural network hierarchical cell annotation (7 major TME cell types, 47 subtypes) together with Seurat/Harmony batch integration to cancer scRNA-seq data, yielding an annotated Seurat object that feeds downstream analyses such as inferCNV CNA profiling.

## When to use

Use when a cancer scRNA-seq dataset (counts matrix, Seurat object, csv/rds/h5, or 10X folder) needs automated cell-type annotation within the tumor microenvironment and batch-effect-corrected integration before downstream analyses like inferCNV.

## When NOT to use

- The dataset contains tissue-specific cell types (e.g., alveolar cells in lung cancers) — scWizard may leave them as 'Unknown' and other annotation strategies are needed.
- The analysis goal is not cancer/TME single-cell annotation, e.g., already-annotated data or non-scRNA-seq feature tables.
- Only 300-cell or exploratory toy data without batch structure and without need for cell-type labels.

## Inputs

- scRNA-seq counts matrix (rows = genes, columns = cells) as csv, rds, or h5
- Seurat object ('Data-rds' input)
- 10X-format data folder ('Data-10X' input)
- Custom model training data in H5 format with trainset labels (genes as rows, cells as columns)

## Outputs

- Annotated Seurat object with 7 major cell types and 47 TME cell subtypes
- Batch-effect-corrected integrated object with before/after comparison plots
- QC-filtered dataset saved to the data input folder
- Annotated cell labels usable as inferCNV reference (non-malignant) and observation (malignant epithelial) sets

## How to apply

Install the scWizard R package from GitHub (requires R v4.0.2+, jags) and launch the shiny GUI with scWizard::run_app(). First run Quality Control, which by default filters cells with <200 genes detected, >10,000 genes detected, or >5% of reads mapping to mitochondrial RNA. If multiple batches (different technologies, patients, or experiments) are present, run Batch Processing, choosing between Seurat and Harmony integration — Harmony is preferred for datasets with common cell types across techniques and shorter runtime on large datasets. Then run Cell Annotation using the built-in 'two-step' annotation model: major cell types are predicted on individual cells before clustering (hierarchical deep neural network, ReLU + softmax) and subtypes are annotated after clustering, using the pre-trained reference of 113,976 cells across 13 cancer types (7 major types, 47 subtypes) or a custom H5 trainset; hidden-layer nodes can be tuned or defaults kept. The resulting annotated Seurat object designates epithelial/malignant versus non-epithelial compartments, which is how non-malignant populations become the inferCNV reference and malignant epithelial cells the observation set for copy-number alteration analysis. Results are saved automatically to folders in the current working directory.

## Related tools

- **scWizard** (Point-and-click shiny package providing QC, Seurat/Harmony batch integration, and the two-step deep-neural-network cell annotation model) — https://github.com/Dulab2020/scWizard
- **Seurat** (Standardized scRNA-seq pipeline inside scWizard (dimension reduction, clustering, differential expression) and one of the two batch-effect removal methods)
- **Harmony** (Alternative batch-effect removal method, suited to datasets with common cell types across techniques and faster runtime for large-dataset exploration)
- **infercnv** (Downstream consumer of the annotated Seurat object: malignant epithelial cells vs non-malignant populations are used as observation/reference sets for large-scale CNA analysis)

## Examples

```
if (!requireNamespace("devtools", quietly=TRUE)) install.packages("devtools"); devtools::install_github("Dulab2020/scWizard"); scWizard::run_app()
```

## Evaluation signals

- Annotation accuracy should approach published independent-dataset benchmarks: 0.98 for major cell types, 0.85 for myeloid subtypes, and 0.79 for T/NK subtypes.
- QC output should contain no cells with <200 genes, >10,000 genes, or >5% mitochondrial reads.
- Batch-processing view should show pre- and post-correction UMAP/graphs with reduced batch separation while preserving cell-type structure.
- Downstream validation: the inferCNV clustered CNA heatmap should visually separate the malignant epithelial compartment from non-malignant populations (endothelial cells, fibroblasts), with epithelial CNA exceeding non-malignant CNA.
- Outputs are saved automatically to folders in the current working directory (e.g., 'inforcnv_res' for inferCNV), confirming the run completed.

## Limitations

- Tissue-specific cell types (e.g., alveolar cells in lung cancers) may not be identified and are named 'Unknown'; tissue-specific subtypes and cellular states are deferred to future updates.
- The annotation module requires extra setup: installing the Python environment via the 'install Python' button, downloading the trainset (pan.baidu.com link) into the 'app/www/python' folder, and jags as an install dependency.
- Before using inferCNV, users must separately prepare/upload a gene location file mapping genes to chromosomal coordinates.

## Evidence
