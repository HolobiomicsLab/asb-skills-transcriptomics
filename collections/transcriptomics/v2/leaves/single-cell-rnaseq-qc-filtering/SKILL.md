---
name: single-cell-rnaseq-qc-filtering
description: Use when raw scRNAseq FASTQ/output from a time-series (e.g., GSE223109
  mouse dermis) must be QC-filtered, integrated across samples, clustered, and annotated
  into major cell types with marker genes.
when_to_use_negative:
- When the goal is circadian-time prediction itself rather than cell QC/annotation
  — tauFisher cannot predict circadian time at single-cell level due to scRNAseq dropout
  and requires pseudobulk or bulk input
- When input is already a QC-filtered, annotated single-cell object (e.g., a cell-by-type
  count table) — re-filtering would discard valid cells
- When the tissue or organism differs such that the marker genes Col1a1, Acta1, Fabp4,
  Cd52/Cd74 are not valid identity markers
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellRanger
  role: Aligns the raw GSE223109 single-cell sequencing output using the MM10 reference
    genome (version 3.1.0)
- name: Seurat
  role: Performs SCTransform normalization per sample, SelectIntegrationFeatures (3250
    features), PCA, and Louvain clustering of the integrated dataset
- name: tauFisher
  role: Downstream consumer of the QC-passed, cell-type-annotated dermis scRNAseq
    data (applied at cell-type level on pseudobulk expression)
  repo: github.com/micnngo/tauFisher
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-48041-6
    title: tauFisher predicts circadian time from a single sample of bulk and single-cell
      pseudobulk transcriptomic data
  source_content_hash: sha256:a522b4aa55dde7fe14840c8c5d1cc7716e01fb42bf50aae7fdd755c0c2241d8d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CellRanger
  - Seurat
  - tauFisher
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# single-cell-rnaseq-qc-filtering

## Summary

Quality-control and cell-type annotation workflow for the GSE223109 mouse dermis scRNAseq time series: align raw sequencing output with CellRanger 3.1.0 (MM10), filter cell barcodes, normalize/integrate with Seurat, cluster with Louvain, and tabulate cells per dermal cell type using canonical marker genes.

## When to use

Use when raw scRNAseq FASTQ/output from a time-series (e.g., GSE223109 mouse dermis) must be QC-filtered, integrated across samples, clustered, and annotated into major cell types with marker genes.

## When NOT to use

- When the goal is circadian-time prediction itself rather than cell QC/annotation — tauFisher cannot predict circadian time at single-cell level due to scRNAseq dropout and requires pseudobulk or bulk input
- When input is already a QC-filtered, annotated single-cell object (e.g., a cell-by-type count table) — re-filtering would discard valid cells
- When the tissue or organism differs such that the marker genes Col1a1, Acta1, Fabp4, Cd52/Cd74 are not valid identity markers

## Inputs

- Raw single-cell sequencing output from GSE223109 mouse dermis (FASTQ/BCL for CellRanger)
- MM10 mouse reference genome for CellRanger alignment
- Per-cell feature counts and mitochondrial gene percentage matrix (e.g., Seurat object)

## Outputs

- Matrix of QC-passing cells (16,866 cells for GSE223109) with cluster and cell-type annotations
- cell_counts.csv table of cells per annotated type (fibroblasts 12,649, immune 3,353, muscle 722, endothelial 142)
- Integrated Seurat object with SCTransform normalization and 3250 integration features

## How to apply

Align raw sequencing output with CellRanger version 3.1.0 using the MM10 reference genome. QC the cell barcodes by keeping cells with 850-7800 detected features and less than 13% mitochondrial genes. Normalize each sample with SCTransform and integrate samples using 3250 features selected via SelectIntegrationFeatures. Run principal component analysis on the integrated dataset and generate clusters with the Louvain algorithm, then annotate clusters by marker gene expression: Col1a1 for fibroblasts, Acta1 for muscle cells, Fabp4 for endothelial cells, and Cd52/Cd74 for immune cells. Tabulate cells per annotated type and verify the totals (e.g., 16,866 QC-passing cells: fibroblasts 12,649, immune 3,353, muscle 722, endothelial 142) before saving the counts table.

## Related tools

- **CellRanger** (Aligns the raw GSE223109 single-cell sequencing output using the MM10 reference genome (version 3.1.0))
- **Seurat** (Performs SCTransform normalization per sample, SelectIntegrationFeatures (3250 features), PCA, and Louvain clustering of the integrated dataset)
- **tauFisher** (Downstream consumer of the QC-passed, cell-type-annotated dermis scRNAseq data (applied at cell-type level on pseudobulk expression)) — github.com/micnngo/tauFisher

## Examples

```
devtools::install_github("micnngo/tauFisher", build_vignettes = TRUE)
```

## Evaluation signals

- Total QC-passing cells equals 16,866 for GSE223109 (thresholds: 850-7800 features, <13% mitochondrial genes)
- Per-type counts match fibroblasts N = 12,649, immune cells N = 3,353, muscle cells N = 722, endothelial cells N = 142
- Sum of the per-type cell counts in cell_counts.csv equals the QC-passing total
- SelectIntegrationFeatures returns exactly 3250 integration features across samples
- Marker genes are restricted to expected clusters (e.g., Col1a1 in the largest fibroblast cluster, Acta1 in muscle, Fabp4 in endothelial, Cd52/Cd74 in immune)

## Limitations

- tauFisher applied downstream cannot overcome the high sequencing dropout rate in scRNAseq and thus cannot predict circadian time at a single cell level — prediction requires pseudobulking by cell type
- Claims on differences between cell types (e.g., phase heterogeneity in dermal fibroblasts vs immune cells) rely only on computational analysis, with no experimental validation
- The QC thresholds (850-7800 features, <13% mitochondrial) are dataset-specific and were not justified as universally optimal

## Evidence

- [discussion] CellRanger alignment with MM10: "We used CellRanger version 3.1.0 with MM10 reference to process the raw sequencing output."
- [discussion] Cell barcode QC thresholds: "Cells with 850-7800 features and less than 13% of mitochondrial genes were kept."
- [discussion] SCTransform and integration: "The SCTransform function was performed on each sample and 3250 integration features were selected using SelectIntegrationFeatures"
- [discussion] Clustering method: "Principal component analysis was then done on the integrated dataset and the Louvain algorithm was used to generate the clusters"
- [readme] tauFisher installation from GitHub (README): [evidence span withheld — anchor status: external_doc]
