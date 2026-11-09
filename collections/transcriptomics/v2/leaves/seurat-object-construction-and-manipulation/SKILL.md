---
name: seurat-object-construction-and-manipulation
description: Use when raw scRNA-seq UMI counts (e.g. 10x cellranger filtered_bc_matrix
  or gene-by-cell tables) must be converted into a Seurat object for QC, normalization,
  scaling and variable-feature detection.
when_to_use_negative:
- Input is already normalized/scaled or a processed feature table rather than raw
  UMI counts — re-running LogNormalize would double-transform
- scATAC-seq fragment files or other non-gene-count modalities, which SCALA handles
  through ArchR-based LSI workflows instead of Seurat objects
- Datasets from organisms other than human or mouse, which are outside SCALA's currently
  supported scope
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: Provides CreateSeuratObject, NormalizeData (LogNormalize ×10,000), ScaleData
    (z-score + covariate regression) and VST-based FindVariableFeatures used for QC
    and HVG detection
- name: SCALA
  role: Web/desktop pipeline embedding the Seurat-based scRNA-seq QC–normalization–scaling–variable-feature
    workflow in a GUI without coding
- name: R
  role: Language in which Seurat object construction and SCALA itself are implemented
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:0a29521f49d23c5418f69b236e902dc4a37a20b60d14df36183375dfe4d2517a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PavlopoulosLab/SCALA
  tools:
  - R
  - SCALA
  - Seurat
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# Seurat object construction and manipulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Construct and manipulate a Seurat object from a scRNA-seq count matrix as the central data container for SCALA's scRNA-seq pipeline: QC filtering, LogNormalize normalization (×10,000, log-transform), z-score scaling, and VST-based highly variable feature detection.

## When to use

Use when raw scRNA-seq UMI counts (e.g. 10x cellranger filtered_bc_matrix or gene-by-cell tables) must be converted into a Seurat object for QC, normalization, scaling and variable-feature detection.

## When NOT to use

- Input is already normalized/scaled or a processed feature table rather than raw UMI counts — re-running LogNormalize would double-transform
- scATAC-seq fragment files or other non-gene-count modalities, which SCALA handles through ArchR-based LSI workflows instead of Seurat objects
- Datasets from organisms other than human or mouse, which are outside SCALA's currently supported scope

## Inputs

- Raw scRNA-seq UMI count matrix in 10x cellranger filtered_bc_matrix format
- Tab-delimited gene-by-cell count table (e.g. PBMC3k dataset)
- Per-cell metadata for optional covariate regression (e.g. mito percentage, nFeature_RNA)

## Outputs

- Preprocessed Seurat object saved as RDS
- Highly variable feature table (CSV, e.g. 1535 variable genes for PBMC3k)
- QC violin plots (PNG) showing genes detected, total UMIs and mitochondrial percentage
- Normalized (LogNormalize, scale factor 10,000) and z-score-scaled expression matrices stored in the Seurat object

## How to apply

Load the PBMC3k-style UMI count matrix (10x filtered_bc_matrix format or tab-delimited gene-by-cell table) into a Seurat object via CreateSeuratObject. Apply QC on per-cell unique features, per-cell UMIs and mitochondrial percentage, excluding dying cells (high mito%) and putative multiplets (very high feature/UMI counts), and render the three metrics as violin plots. Normalize with Seurat's global-scaling LogNormalize: per-barcode counts divided by total barcode counts, multiplied by 10,000, log-transformed; then scale to z-scores (column-wise mean 0, variance 1), optionally regressing out metadata covariates. Detect highly variable features with the VST method, which fits a local polynomial regression to the log-variance vs log-mean relationship, standardizes features by observed mean and expected variance, clips to a maximum, and returns a fixed number of variable features (SCALA's PBMC3k run identified 1535). Save the preprocessed object as RDS, the variable-feature table as CSV, and QC violin plots as PNG for downstream SCALA analysis.

## Related tools

- **Seurat** (Provides CreateSeuratObject, NormalizeData (LogNormalize ×10,000), ScaleData (z-score + covariate regression) and VST-based FindVariableFeatures used for QC and HVG detection)
- **SCALA** (Web/desktop pipeline embedding the Seurat-based scRNA-seq QC–normalization–scaling–variable-feature workflow in a GUI without coding)
- **R** (Language in which Seurat object construction and SCALA itself are implemented)

## Examples

```
pbmc <- CreateSeuratObject(counts = counts.matrix); pbmc <- NormalizeData(pbmc, normalization.method = "LogNormalize", scale.factor = 10000); pbmc <- FindVariableFeatures(pbmc, selection.method = "vst")
```

## Evaluation signals

- Normalization check: LogNormalize output equals log10(counts / total counts × 10,000) per barcode
- Scaling check: scaled feature rows have column-wise mean 0 and variance 1 (z-scores), with specified metadata covariates regressed out
- HVG count check: VST returns a fixed number of variable features; SCALA's PBMC3k run yielded the expected 1535 variable genes
- QC plot check: violin plots display the three metrics — genes detected, total UMIs and mitochondrial percentage — matching the paper's Fig. 2A
- Output artifact check: RDS Seurat object, CSV variable-feature table and PNG QC plots are all produced and non-empty

## Limitations

- SCALA supports only human and mouse datasets for both scRNA-seq and scATAC-seq analysis
- Online SCALA version restricts dataset size to < 2 GB with no more than 2 CPU threads per session; steps may not complete for datasets larger than 50,000 cells
- Software version numbers for Seurat employed in SCALA are not stated, complicating exact reproduction of variable-feature counts

## Evidence

- [results] SCALA's scRNA-seq pipeline performs highly variable feature detection using the mean-variance-plot (MVP) method implemented by the Seurat package, leading to the identification of 1535 variable genes.: [evidence span withheld — anchor status: not_found]
- [methods] Normalize the filtered matrix with a global-scaling normalization in which each barcode's gene counts are normalized by total barcode counts, multiplied by 10,000, and log-transformed.: [evidence span withheld — anchor status: not_found]
- [readme] An R package (compatible with SCope) to create generic .loom files and extend them with other data e.g.: SCENIC regulons, Seurat clusters and markers: [evidence span withheld — anchor status: external_doc]
