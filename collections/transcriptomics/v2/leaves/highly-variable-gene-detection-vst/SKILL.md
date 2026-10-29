---
name: highly-variable-gene-detection-vst
description: Use when a normalized scRNA-seq matrix (e.g., PBMC3k UMI counts after
  QC filtering and log-normalization) needs a reduced set of informative features
  before PCA, clustering, or downstream SCALA/Seurat analyses.
when_to_use_negative:
- 'Non-human or non-mouse datasets: SCALA currently supports ''the analysis of only
  human and mouse datasets'' for scRNA-seq and scATAC-seq.'
- Datasets > 2 GB on the online SCALA server, where dataset size is restricted and
  no more than 2 CPU threads are allocated per session; some steps cannot complete
  for > 50,000 cells online.
- Input is already a reduced feature set or the variable-feature table exists — rerunning
  VST detection would be redundant.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCALA
  role: Interactive pipeline whose Variable Features Detection module wraps the MVP
    method for selecting highly variable genes
  repo: https://github.com/PavlopoulosLab/SCALA
- name: Seurat
  role: R library implementing the VST-based variable feature detection (mean-variance-plot)
    applied to the normalized PBMC3k matrix
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.10.032
    title: 'SCALA: A complete solution for multimodal analysis of single-cell Next
      Generation Sequencing data'
  source_content_hash: sha256:1d1e5e7ecd99c59e1933318588e6ebe08abc0f8ab2f6965506e23880845f7c99
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PavlopoulosLab/SCALA
  repo_url: https://github.com/PavlopoulosLab/SCALA
  tools:
  - R
  - SCALA
  - Seurat
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# highly-variable-gene-detection-vst

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detects highly variable genes in scRNA-seq data as implemented in SCALA's scRNA-seq pipeline, which relies on the mean-variance-plot (MVP) method of the Seurat package backed by Variance Stabilizing Transformation (VST), identifying 1535 variable genes on the PBMC3k dataset.

## When to use

Use when a normalized scRNA-seq matrix (e.g., PBMC3k UMI counts after QC filtering and log-normalization) needs a reduced set of informative features before PCA, clustering, or downstream SCALA/Seurat analyses.

## When NOT to use

- Non-human or non-mouse datasets: SCALA currently supports 'the analysis of only human and mouse datasets' for scRNA-seq and scATAC-seq.
- Datasets > 2 GB on the online SCALA server, where dataset size is restricted and no more than 2 CPU threads are allocated per session; some steps cannot complete for > 50,000 cells online.
- Input is already a reduced feature set or the variable-feature table exists — rerunning VST detection would be redundant.

## Inputs

- scRNA-seq UMI count matrix in 10x cellranger filtered_bc_matrix format
- Tab-delimited gene-by-cell count table
- Pre-filtered Seurat object with QC metrics (unique features, UMIs, percent mitochondrial content)

## Outputs

- Table of highly variable genes (CSV), e.g. 1535 variable genes on PBMC3k
- Preprocessed Seurat object saved as RDS
- QC violin plots (PNG) of genes detected, total reads, and mitochondrial percentage

## How to apply

Load the PBMC3k UMI count matrix (10x cellranger filtered_bc_matrix or tab-delimited gene-by-cell table) into a Seurat object, then apply quality-control filtering on per-cell unique features, UMIs, and mitochondrial percentage to drop dying cells (high mito %) and putative multiplets (very high feature/UMI counts). Normalize each barcode by total counts, multiply by 10,000, log-transform, and scale to z-scores. Run variable-feature detection with Seurat's VST selection method: fit a local polynomial regression to the log-variance vs log-mean relationship, standardize features by observed mean and expected variance, clip to a maximum, and return a fixed number of features (default nfeatures = 2000). In SCALA's pipeline this corresponds to the mean-variance-plot (MVP) method, which on the PBMC3k use-case led to 1535 variable genes rather than the 2000 default. Save the preprocessed Seurat object (RDS) and the variable-feature table (CSV) for downstream PCA (25 informative PCs in the paper's k-fold cross-validation).

## Related tools

- **SCALA** (Interactive pipeline whose Variable Features Detection module wraps the MVP method for selecting highly variable genes) — https://github.com/PavlopoulosLab/SCALA
- **Seurat** (R library implementing the VST-based variable feature detection (mean-variance-plot) applied to the normalized PBMC3k matrix) — https://github.com/satijalab/seurat

## Examples

```
seurat_obj <- FindVariableFeatures(seurat_obj, selection.method = "vst", nfeatures = 2000)
```

## Evaluation signals

- The HVG table contains the expected number of features: 1535 variable genes on PBMC3k (paper's result) or 2000 under Seurat's VST default.
- QC violin plots display the three expected metrics (genes detected, total reads, mitochondrial percentage) as in the paper's Fig. 2A.
- Downstream SVD k-fold cross-validation yields the expected number of informative principal components (25 PCs on PBMC3k).
- Variable-feature variance appears clipped at a maximum after standardization by observed mean and expected variance, consistent with VST.
- The saved RDS object loads in R and its variable-feature metadata matches the exported CSV row-for-row.

## Limitations

- SCALA's online version restricts supported dataset size to < 2 GB with no more than 2 CPU threads per session; users should prefer the desktop version for larger datasets (> 50,000 cells).
- Only human and mouse datasets are currently supported in both modalities.
- Software version numbers for Seurat employed in SCALA are not stated in the results section, so exact reproduction of the 1535-gene count may depend on version differences versus other publications.

## Evidence
