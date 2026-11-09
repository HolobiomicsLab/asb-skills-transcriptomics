---
name: scrnaseq-count-matrix-loading
description: Use when raw scRNA-seq count matrices must be ingested into R before
  SCEVAN analysis — e.g., GSE131928 GBM samples (MGH102, MGH104, MGH105, MGH106) or
  multiple myeloma 10X data destined for pipelineCNA or multiSampleComparisonClonalCN.
when_to_use_negative:
- Input is bulk DNA data (low-depth WGS/WES of tumor biopsies such as the 26 GBM bulk
  samples) — those serve as copy-number ground truth, not as count-matrix input
- You already have SCEVAN output files (e.g., *.seg or MGH106_CNAmtx.RData) and want
  to plot or interpret results rather than start the analysis
- The task is classification/segmentation itself — this skill only stages the data;
  run pipelineCNA or the CopyKAT/inferCNV comparisons afterwards
tools:
- name: SCEVAN
  role: Downstream R package that consumes the loaded count matrix via pipelineCNA
    / multiSampleComparisonClonalCN; defines the required matrix orientation (genes
    on rows, cells on columns, Gene Symbol or Ensembl ID)
  repo: github.com/AntonioDeFalco/SCEVAN
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:cf6692f52abc9a1cd46cd329db053f18d38b90396bf8567956509ea0708eb320
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CNVkit v0.9.9
  - DNAcopy
  - SCEVAN
  - inferCNV v1.4.0
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# scrnaseq-count-matrix-loading

## Summary

Loads raw scRNA-seq count matrices (e.g., GSE131928 glioblastoma samples MGH102/MGH104/MGH105/MGH106 or a multiple myeloma 10X matrix) into R in the layout SCEVAN's pipelineCNA expects, as the first step of clonal copy-number analysis.

## When to use

Use when raw scRNA-seq count matrices must be ingested into R before SCEVAN analysis — e.g., GSE131928 GBM samples (MGH102, MGH104, MGH105, MGH106) or multiple myeloma 10X data destined for pipelineCNA or multiSampleComparisonClonalCN.

## When NOT to use

- Input is bulk DNA data (low-depth WGS/WES of tumor biopsies such as the 26 GBM bulk samples) — those serve as copy-number ground truth, not as count-matrix input
- You already have SCEVAN output files (e.g., *.seg or MGH106_CNAmtx.RData) and want to plot or interpret results rather than start the analysis
- The task is classification/segmentation itself — this skill only stages the data; run pipelineCNA or the CopyKAT/inferCNV comparisons afterwards

## Inputs

- Raw scRNA-seq count matrix with genes on rows (Gene Symbol or Ensembl ID) and cells on columns (count_mtx)
- Named list of raw count matrices for multi-sample analysis (listCountMtx)
- Pre-processed sample RData files from the SCEVAN repo (MGH106.RData, listCountMtx.RData, HNSCC26.RData)
- Public GEO dataset GSE131928 (GBM samples MGH102, MGH104, MGH105, MGH106); multiple myeloma 10X count matrix

## Outputs

- R count matrix object (count_mtx) formatted for SCEVAN
- Named list of count matrices (listCountMtx) for multi-sample clonal CN comparison

## How to apply

Obtain the count matrices either from GEO (GSE131928 for MGH102/MGH104/MGH105/MGH106) or from the pre-processed RData files distributed with the SCEVAN repo (MGH106.RData, listCountMtx.RData, HNSCC26.RData). Load them into an R session with load() so that the object bound to count_mtx has genes on rows (gene symbols or Ensembl IDs are both allowed) and cells on columns; for multi-sample comparison, use a named list of matrices (listCountMtx). No preprocessing is required at this step — SCEVAN itself filters cells with <200 detected genes, genes expressed in <1% of cells, and REACTOME cell-cycle genes downstream. Verify dimensions and row/column naming before calling pipelineCNA(count_mtx, sample=..., par_cores=20, SUBCLONES=TRUE) or multiSampleComparisonClonalCN(listCountMtx, par_cores=20).

## Related tools

- **SCEVAN** (Downstream R package that consumes the loaded count matrix via pipelineCNA / multiSampleComparisonClonalCN; defines the required matrix orientation (genes on rows, cells on columns, Gene Symbol or Ensembl ID)) — github.com/AntonioDeFalco/SCEVAN

## Examples

```
load("MGH106_data.RData"); results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE)
```

## Evaluation signals

- count_mtx rownames are valid gene symbols or Ensembl IDs and columns correspond one-to-one to cells
- Matrix orientation matches SCEVAN's contract: genes on rows, cells on columns (transpose otherwise)
- Downstream pipelineCNA runs without annotation errors, producing output files like MGH106_count_mtx_annot.RData and MGH106_CNAmtx.RData
- For multi-sample analysis, listCountMtx is a named list whose names identify each sample (e.g., MGH102, MGH104, MGH105, MGH106)

## Limitations

- Pre-processed sample datasets are hosted externally (DropBox links in the README) and must be downloaded before loading; the multiple myeloma 10X matrix is not pre-packaged
- The README does not state a SCEVAN version; re-check matrix orientation expectations against the installed package
- Loading alone does no QC — cells with <200 detected genes and genes expressed in <1% of cells are only filtered later inside the SCEVAN workflow

## Evidence

- [readme] Count matrix orientation contract: [evidence span withheld — anchor status: external_doc]
- [readme] Pre-packaged sample datasets: [evidence span withheld — anchor status: not_found]
- [methods] Loading GBM sample data from GEO: "We load the scRNA data of MGH106 sample from the public dataset of Gliobastoma (GSE131928)"
- [methods] Downstream call consuming the loaded matrix: "results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)"
