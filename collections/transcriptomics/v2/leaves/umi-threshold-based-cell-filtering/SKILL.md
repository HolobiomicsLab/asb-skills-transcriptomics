---
name: umi-threshold-based-cell-filtering
description: Use when you have raw gene-by-cell spliced/unspliced UMI count matrices
  from scKB (kallisto/bustools) and need to remove low-quality or empty droplets non-arbitrarily
  before downstream Seurat or RNA-velocity analysis.
when_to_use_negative:
- Input is already quality-filtered (e.g. a processed Seurat object or previously
  filtered 10x matrices) rather than raw scKB count matrices
- The goal includes doublet removal or Seurat normalization/PCA/UMAP/clustering in
  the same step and a different pipeline with remove.doublet=TRUE/do.seurat=TRUE is
  required
- You need cell type or developmental stage annotation outside the root of Arabidopsis
  thaliana, which COPILOT does not support
edam_operation: http://edamontology.org/operation_3237
edam_topics:
- http://edamontology.org/topic_3934
tools:
- name: COPILOT
  role: Performs the non-arbitrary UMI/mitochondrial-percentage-based cell quality
    filtering and writes the filtered matrices and HTML summary
  repo: https://github.com/Hsu-Che-Wei/COPILOT
- name: scKB
  role: Produces the raw spliced and unspliced UMI count matrices that serve as the
    recommended input
- name: R2HTML
  role: Renders the COPILOT HTML summary report with cell statistics and UMI histogram
    plots
- name: DropletUtils
  role: Required R dependency for COPILOT's empty-droplet/cell-calling machinery
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xpro.2022.101729
    title: Protocol for fast scRNA-seq raw data processing using scKB and non-arbitrary
      quality control with COPILOT
  source_content_hash: sha256:19efe48d8d3c5ddc4596e431104bf55d48b024d810cd6c46694983f51874921f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ohlerlab/COPILOT
  repo_url: https://github.com/Hsu-Che-Wei/COPILOT
  tools:
  - COPILOT
  - DropletUtils
  - R2HTML
  - scKB
  edam_operation: http://edamontology.org/operation_3237
  edam_topics:
  - http://edamontology.org/topic_3934
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# umi-threshold-based-cell-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Non-arbitrary quality filtering of scRNA-seq gene-by-cell UMI count matrices using COPILOT, which separates low-quality (empty-droplet-like) from high-quality cells by comparing UMI count distributions rather than imposing arbitrary cutoffs, and outputs filtered spliced/unspliced matrices plus a CellRanger-3.1-comparable HTML summary.

## When to use

Use when you have raw gene-by-cell spliced/unspliced UMI count matrices from scKB (kallisto/bustools) and need to remove low-quality or empty droplets non-arbitrarily before downstream Seurat or RNA-velocity analysis.

## When NOT to use

- Input is already quality-filtered (e.g. a processed Seurat object or previously filtered 10x matrices) rather than raw scKB count matrices
- The goal includes doublet removal or Seurat normalization/PCA/UMAP/clustering in the same step and a different pipeline with remove.doublet=TRUE/do.seurat=TRUE is required
- You need cell type or developmental stage annotation outside the root of Arabidopsis thaliana, which COPILOT does not support

## Inputs

- Raw spliced gene-by-cell UMI count matrix with row and column names (from scKB output)
- Raw unspliced gene-by-cell UMI count matrix with row and column names (from scKB output)
- Optional total UMI count matrix and sample metadata data.frame
- Mitochondrial gene name pattern or gene list (e.g. 'ATMG' for Arabidopsis thaliana)

## Outputs

- Quality-filtered spliced gene-by-cell count matrix in 10x format
- Quality-filtered unspliced gene-by-cell count matrix in 10x format
- COPILOT HTML summary report with cell statistics and UMI histogram plots (comparable to CellRanger 3.1)

## How to apply

Read the raw spliced and unspliced gene-by-cell count matrices (with row and column names) from the scKB output directory named after the 'sample.name' argument. Call the exported copilot() function with the required mt.pattern argument (e.g. 'ATMG' for Arabidopsis), letting COPILOT detect the distribution of cells enriched in mitochondrial expression to set the low-quality-cell threshold non-arbitrarily. Control stringency via filtering.ratio (lenient: 1; strict: 0; moderate: between 0 and 1) and keep or adjust the default thresholds mt.threshold=5, top.percent=1, min.UMI.low.quality=100, min.UMI.high.quality=300. For pure quality filtering without doublet removal or Seurat downstream steps, set remove.doublet=FALSE and do.seurat=FALSE. Write the filtered matrices to filtered.mtx.output.dir in 10x format and generate the built-in R2HTML summary report; validate that the summary and filtered matrices exist and contain the expected cell-filtering statistics.

## Related tools

- **COPILOT** (Performs the non-arbitrary UMI/mitochondrial-percentage-based cell quality filtering and writes the filtered matrices and HTML summary) — https://github.com/Hsu-Che-Wei/COPILOT
- **scKB** (Produces the raw spliced and unspliced UMI count matrices that serve as the recommended input)
- **R2HTML** (Renders the COPILOT HTML summary report with cell statistics and UMI histogram plots)
- **DropletUtils** (Required R dependency for COPILOT's empty-droplet/cell-calling machinery)

## Examples

```
copilot(sample.name="col0", mt.pattern="ATMG", filtering.ratio=1, remove.doublet=FALSE, do.seurat=FALSE, filtered.mtx.output.dir="filtered")
```

## Evaluation signals

- Filtered spliced and unspliced matrices both exist in filtered.mtx.output.dir in 10x format with consistent gene and cell identifiers
- The COPILOT HTML summary file exists and contains expected cell-filtering statistics, directly comparable to CellRanger 3.1 summary output
- Cells with mitochondrial expression percentage above mt.threshold=5 are removed as dying cells
- No barcode with fewer than min.UMI.low.quality=100 UMIs is retained as a cell; low- vs high-quality cell populations (split at min.UMI.high.quality=300 UMIs) are clearly separated in the summary histograms
- With filtering.ratio=1 (lenient), more cells are retained than with strict (0) or moderate settings, consistent with the stringency metric definition

## Limitations

- COPILOT only supports cell type and developmental stage annotation for the root of Arabidopsis thaliana
- The required genome may not be available as a BSgenome object, requiring conversion of a customized genome
- X11 device unavailability in conda-installed R environments causes COPILOT feature-plot generation (summary file) to fail
- Requires R (>= 3.6.1) with Seurat (>= 3.1.4), DoubletFinder, DropletUtils, R2HTML and the Python module umap-learn installed

## Evidence
