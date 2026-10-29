---
name: seurat-clustering-and-umap-visualization
description: Use when a cancer scRNA-seq dataset (Seurat object, counts matrix in
  csv/rds/h5, or 10X folder) needs clustering into clusters/subclusters and UMAP plots
  colored by predicted cell type or subtype within scWizard.
when_to_use_negative:
- Input is not scRNA-seq data (e.g., bulk RNA-seq, a pre-computed feature table, or
  already-clustered results with no expression matrix)
- Only major cell-type labels are needed on individual cells without cluster-level
  exploration — the first annotation step is applied before clustering, so clustering
  is not required for it
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: 'Provides the standardized clustering pipeline in scWizard: dimension reduction,
    cell clustering, differential gene expression detection, and optionally batch
    effect removal before annotation'
  repo: https://github.com/Dulab2020/scWizard
- name: scWizard
  role: Point-and-click shiny R package that wraps the Seurat clustering step together
    with QC, batch processing, and neural-network cell annotation
  repo: https://github.com/Dulab2020/scWizard
- name: Harmony
  role: Alternative batch-effect removal method usable before clustering when datasets
    share cell types across different techniques or have shorter runtime needs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:d094a1f5e0ceeb66b0e01392a4766aabf8d948277b736fa94168e3c9169a3844
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# seurat-clustering-and-umap-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Dimension reduction, cell clustering, and UMAP visualization of scRNA-seq data via Seurat, as standardized inside scWizard's cancer analysis workflow and used to subcluster major cell types prior to subtype annotation.

## When to use

Use when a cancer scRNA-seq dataset (Seurat object, counts matrix in csv/rds/h5, or 10X folder) needs clustering into clusters/subclusters and UMAP plots colored by predicted cell type or subtype within scWizard.

## When NOT to use

- Input is not scRNA-seq data (e.g., bulk RNA-seq, a pre-computed feature table, or already-clustered results with no expression matrix)
- Only major cell-type labels are needed on individual cells without cluster-level exploration — the first annotation step is applied before clustering, so clustering is not required for it

## Inputs

- Seurat object ('Data-rds')
- Counts matrix (genes as rows, cells as columns) in csv, rds, or h5 format ('Data-counts')
- 10X-format data folder ('Data-10X')
- QC-filtered scRNA-seq data from a cancer sample, e.g. the human intrahepatic cholangiocarcinoma (ICC) dataset

## Outputs

- Cell cluster assignments per cell (major clusters and per-major-type subclusters)
- UMAP 'single-cell map' colored by predicted major cell type and subtype
- Cell-type composition summary (fraction of each cluster)
- Differential gene expression results per cluster
- Per-cell predicted labels table, e.g. icc_cell_annotations.csv

## How to apply

After scWizard's Quality Control module removes low-quality cells (<200 genes, >10,000 genes, >5% mitochondrial reads), run the Seurat-based standardized pipeline for dimension reduction, cell clustering, and differential gene expression detection. For the two-step annotation model, run major cell-type annotation on individual cells before clustering, then subcluster each major cell type (e.g., fibroblasts, myeloid, T/NK, endothelial cells from the ICC case study) with Seurat clustering and assign each subcluster to one of 47 TME subtypes, accepting the subcluster with the largest number of predicted cells per subtype. Visualize annotated cells on a UMAP 'single-cell map' colored by predicted major type and subtype, and summarize cell-type composition as cluster fractions. Batch effects between samples can be removed beforehand with Seurat or Harmony integration when multiple datasets, patients, or technologies are combined.

## Related tools

- **Seurat** (Provides the standardized clustering pipeline in scWizard: dimension reduction, cell clustering, differential gene expression detection, and optionally batch effect removal before annotation) — https://github.com/Dulab2020/scWizard
- **scWizard** (Point-and-click shiny R package that wraps the Seurat clustering step together with QC, batch processing, and neural-network cell annotation) — https://github.com/Dulab2020/scWizard
- **Harmony** (Alternative batch-effect removal method usable before clustering when datasets share cell types across different techniques or have shorter runtime needs)

## Examples

```
if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools"); devtools::install_github("Dulab2020/scWizard"); scWizard::run_app()
```

## Evaluation signals

- Every cell receives a label among the 7 major cell types / 47 TME subtypes, or 'Unknown' for tissue-specific types not covered by the model
- Each of the 47 subtypes is assigned at most one subcluster, and the accepted subcluster contains the largest number of cells predicted as that subtype
- Annotated clusters are corroborated by canonical marker gene expression, as in the ICC case study where 8 clusters were verified
- UMAP plots render cells grouped by predicted major type and subtype with composition fractions summing to 1 across clusters
- Per-cell labels and cluster fractions are written to a complete output table (e.g., icc_cell_annotations.csv) with no missing labels

## Limitations

- Tissue-specific cell types (e.g., alveolar cells in lung cancers) may not be identified and fall into the 'Unknown' cluster; tissue-specific subtypes and cellular states are deferred to future updates
- Subtype annotation accuracy is imperfect: 0.85 for myeloid and 0.79 for T/NK subtypes (vs 0.98 for major types), so cluster-level subtype calls need marker-based verification
- Hidden-layer node counts must only be adjusted per the documented criteria (between input and output sizes, 2/3 input + 2/3 output, or less than twice input size), otherwise defaults should be kept

## Evidence
