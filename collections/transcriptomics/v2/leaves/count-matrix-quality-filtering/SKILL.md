---
name: count-matrix-quality-filtering
description: Use when raw spliced/unspliced scKB count matrices need background-noise
  cell removal without arbitrary thresholds, e.g. downstream RNA velocity or trajectory
  inference on scRNA-seq data.
when_to_use_negative:
- You only have aligned reads or FASTQ files and have not yet produced scKB spliced/unspliced
  count matrices — run scKB alignment/counting first
- You want cell-type or developmental-stage annotation on tissue other than Arabidopsis
  thaliana root — COPILOT annotation only supports that species/tissue
- You need absolute, externally-validated quality cutoffs for cross-sample comparisons
  rather than distribution-based per-sample filtering
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: COPILOT
  role: Performs non-arbitrary quality filtering by comparing distributions of low-quality
    and high-quality cells and outputs filtered spliced/unspliced matrices
  repo: github.com/Hsu-Che-Wei/COPILOT
- name: scKB
  role: Upstream producer of the raw spliced and unpliced counts matrices used as
    COPILOT input
- name: Seurat
  role: Downstream normalization, PCA, UMAP, and clustering on the filtered cells
    (do.seurat=TRUE)
- name: kallisto
  role: Pseudoalignment engine underlying scKB/COPILOT, contributing to ~30x faster
    processing than Cell Ranger 3.1
- name: bustools
  role: Bus-tools transcript counting engine wrapped by scKB for count matrix generation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xpro.2022.101729
    title: Protocol for fast scRNA-seq raw data processing using scKB and non-arbitrary
      quality control with COPILOT
  source_content_hash: sha256:1f9e1e6f2b856e96880e07de28ea6946ebb5cfeb0397bd4dad3fa12b38b50668
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ohlerlab/COPILOT
  tools:
  - COPILOT
  - Seurat
  - bustools
  - kallisto
  - scKB
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# count-matrix-quality-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Non-arbitrary quality filtering of gene-by-cell UMI count matrices using COPILOT, which iteratively compares distributions of low-quality (mitochondria-enriched) and high-quality cells instead of imposing arbitrary UMI cutoffs, producing quality-filtered spliced and unspliced count matrices.

## When to use

Use when raw spliced/unspliced scKB count matrices need background-noise cell removal without arbitrary thresholds, e.g. downstream RNA velocity or trajectory inference on scRNA-seq data.

## When NOT to use

- You only have aligned reads or FASTQ files and have not yet produced scKB spliced/unspliced count matrices — run scKB alignment/counting first
- You want cell-type or developmental-stage annotation on tissue other than Arabidopsis thaliana root — COPILOT annotation only supports that species/tissue
- You need absolute, externally-validated quality cutoffs for cross-sample comparisons rather than distribution-based per-sample filtering

## Inputs

- Raw spliced gene-by-cell UMI count matrix with row and column names (from scKB)
- Raw unspliced gene-by-cell UMI count matrix with row and column names (from scKB)
- Mitochondrial gene-id pattern (e.g. 'ATMG') or list of mitochondrial genes

## Outputs

- Quality-filtered spliced and unspliced count matrices (written to filtered.mtx.output.dir)
- COPILOT summary file (HTML) with UMI histogram, feature plots, and statistics
- Seurat object with normalized expression, PCA, UMAP, and clustering (when do.seurat=TRUE)

## How to apply

Feed COPILOT the raw spliced and unpliced matrices produced by the scKB pipeline (kallisto/bustools wrapper), placing them in a directory whose name matches the sample.name argument. Set mt.pattern (e.g. 'ATMG' for Arabidopsis) to identify mitochondrial genes used for initial dying-cell detection via the mt.threshold (default 5% mitochondrial expression). Provide min.UMI.low.quality=100 and min.UMI.high.quality=300 to define barcode-to-cell and high-quality populations, top.percent=1 to drop high-UMI cells, and set filtering.ratio=0 for the most stringent non-arbitrary filtering (1 is lenient, 0 is strict, intermediate values are moderate). COPILOT iterates the filtering cut-off at the chosen ratio until no cells remain more similar to the low-quality than the high-quality expression profile, then with remove.doublet=TRUE removes doublets (optionally estimating the rate per 10X Genomics) and with do.seurat=TRUE normalizes, runs PCA, UMAP, and clustering via Seurat on the filtered cells.

## Related tools

- **COPILOT** (Performs non-arbitrary quality filtering by comparing distributions of low-quality and high-quality cells and outputs filtered spliced/unspliced matrices) — github.com/Hsu-Che-Wei/COPILOT
- **scKB** (Upstream producer of the raw spliced and unpliced counts matrices used as COPILOT input)
- **Seurat** (Downstream normalization, PCA, UMAP, and clustering on the filtered cells (do.seurat=TRUE))
- **kallisto** (Pseudoalignment engine underlying scKB/COPILOT, contributing to ~30x faster processing than Cell Ranger 3.1)
- **bustools** (Bus-tools transcript counting engine wrapped by scKB for count matrix generation)

## Examples

```
devtools::install_github('Hsu-Che-Wei/COPILOT'); copilot(sample.name='col0_toy', mt.pattern='ATMG', top.percent=1, remove.doublet=TRUE, filtering.ratio=0, do.seurat=TRUE)
```

## Evaluation signals

- The copilot() call completes without error on the toy (col0) data
- Filtered spliced and unspliced matrices exist in filtered.mtx.output.dir
- A COPILOT summary file (UMI histogram, feature plots, HTML statistics) is generated
- With filtering.ratio=0, iteration terminates when no cells are more similar to the low-quality than the high-quality expression profile
- Wall-clock runtime on the toy data is within a few minutes of the reported ~2 min behavior

## Limitations

- COPILOT cell-type and developmental-stage annotation only supports the root of Arabidopsis thaliana
- The required genome may not be available as a BSgenome object, requiring conversion of customized genomes
- X11 device unavailability in conda-installed R environments can cause COPILOT plotting (feature plots in the summary file) to fail
- Default filtering.ratio=1 is lenient; strict filtering requires explicitly setting filtering.ratio=0

## Evidence

- [full_text] Iteration termination condition: [evidence span withheld — anchor status: not_found]
