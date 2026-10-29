---
name: single-cell-rna-seq-quality-filtering
description: Use when a raw scRNA-seq count matrix (e.g., AnnData .h5ad from HuBMAP
  datasets) must be cleaned of low-quality cells and sparsely/ubiquitously expressed
  genes prior to normalization, PCA, and Leiden clustering.
when_to_use_negative:
- Input is already a normalized or scaled expression matrix — filtering cutoffs on
  expressed gene/cell counts are defined for raw count matrices
- Analysis goal is cell-type label transfer on a pre-annotated reference (e.g., scanpy
  Ingest between HBMP2-spleen-2 and HBMP3-spleen-CC3), where the reference has already
  been QC'd
- Data are not single-cell transcriptomic counts (e.g., scATAC-seq cell-by-bin matrices
  or CODEX protein images), where different QC criteria apply
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scanpy
  role: 'Performs preprocessing: cell and gene filtering with the 50/3000 expressed-gene/cell
    cutoffs before normalization'
- name: Cellar
  role: Interactive pipeline whose preprocessing step wraps scanpy filtering; filtered
    data feed PCA/UMAP/Leiden clustering
  repo: https://github.com/euxhenh/cellar
- name: AnnData (Annotated Data object)
  role: In-memory format holding the count matrix during filtering; supports memory-mapping
    for large datasets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-29744-0
    title: Interactive single-cell data analysis using Cellar
  source_content_hash: sha256:3e46945d0f247595b2fa4cb5c9689138e0146084181c9543784f9ab8f3fe439d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/euxhenh/cellar
  tools:
  - AnnData
  - AnnData (Annotated Data object)
  - Cellar
  - Python
  - faiss
  - numpy
  - scanpy
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- single-cell-rna-seq-cell-quality-control-filtering
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# single-cell-rna-seq-quality-filtering

## Summary

Quality-control filtering of single-cell RNA-seq count matrices as implemented in Cellar's scanpy-based preprocessing: removing low/high-quality cells and genes before normalization to make downstream clustering reliable.

## When to use

Use when a raw scRNA-seq count matrix (e.g., AnnData .h5ad from HuBMAP datasets) must be cleaned of low-quality cells and sparsely/ubiquitously expressed genes prior to normalization, PCA, and Leiden clustering.

## When NOT to use

- Input is already a normalized or scaled expression matrix — filtering cutoffs on expressed gene/cell counts are defined for raw count matrices
- Analysis goal is cell-type label transfer on a pre-annotated reference (e.g., scanpy Ingest between HBMP2-spleen-2 and HBMP3-spleen-CC3), where the reference has already been QC'd
- Data are not single-cell transcriptomic counts (e.g., scATAC-seq cell-by-bin matrices or CODEX protein images), where different QC criteria apply

## Inputs

- Raw scRNA-seq cell-by-gene count matrix in AnnData (.h5ad) format, e.g., the 5273-cell HBMP3-spleen-CC2 HuBMAP spleen dataset

## Outputs

- Filtered AnnData object with cells and genes passing the 50/3000 cutoffs, ready for CPM normalization and downstream clustering
- Updated adata with filtered adata.X and adata.var/obs metadata

## How to apply

Load the cell-by-gene count matrix (e.g., 5273-cell HBMP3-spleen-CC2 HuBMAP dataset in AnnData .h5ad format) and apply scanpy preprocessing to filter cells expressing fewer than 50 or more than 3000 genes, and genes expressed in fewer than 50 or more than 3000 cells. These symmetric cutoffs remove likely empty droplets/debris (low gene counts) and doublets or ambient-RNA contamination (high gene counts), while gene filtering drops markers too rare or too common to be informative. Perform filtering before CPM total count normalization (total count = 1e5), log1p transformation, and scaling so that normalization statistics are computed only on retained cells and genes. The filtered matrix then feeds PCA (scikit-learn, randomized SVD, 40 dimensions) and faiss 15-neighbor graph construction for Leiden clustering.

## Related tools

- **scanpy** (Performs preprocessing: cell and gene filtering with the 50/3000 expressed-gene/cell cutoffs before normalization)
- **Cellar** (Interactive pipeline whose preprocessing step wraps scanpy filtering; filtered data feed PCA/UMAP/Leiden clustering) — https://github.com/euxhenh/cellar
- **AnnData (Annotated Data object)** (In-memory format holding the count matrix during filtering; supports memory-mapping for large datasets)

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('HBMP3-spleen-CC2.h5ad'); sc.pp.filter_cells(adata, min_genes=50, max_genes=3000); sc.pp.filter_genes(adata, min_cells=50, max_cells=3000)
```

## Evaluation signals

- Number of cells and genes removed matches expectations for the 50/3000 cutoffs applied to the raw count matrix
- Post-filtering matrix contains only cells expressing between 50 and 3000 genes and only genes expressed in 50–3000 cells (verify min/max of per-cell gene counts and per-gene cell counts)
- Filtered data retain the full cell set needed for the downstream target (e.g., 5273 assignments reproducing 16 Leiden clusters matching Supplementary Fig. 1a)
- Downstream normalization (CPM, total count = 1e5) and scaling to zero mean/unit variance succeed without errors on the filtered matrix
- Session/AnnData output preserves original cell barcodes and gene identifiers after filtering (no misalignment between obs and X)

## Limitations

- No software versions stated for Cellar or scanpy; exact filtering behavior may vary across versions
- The 50/3000 cutoffs are dataset-specific values from the Cellar paper and may be inappropriate for other tissues or sequencing depths
- Cellar reproducibility partly depends on an external web server rather than a locally installable archived artifact

## Evidence

- [methods] Cell filtering cutoffs: "we filtered cells with less than 50 or more than 3000 expressed genes"
- [methods] Gene filtering cutoffs: "We also filtered genes expressed in less than 50 or more than 3000 cells."
- [methods] Preprocessing tool: "Preprocessing of the data was done via scanpy"
- [discussion] AnnData format: "using the Annotated Data object17 in memory-mapping mode which allows the analysis of large datasets"
