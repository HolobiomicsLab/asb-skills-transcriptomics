---
name: tsne-embedding-visualization-of-single-cell-clusters
description: Use when you have a log-transformed gene-cell-barcode matrix from tens
  of thousands of droplet-based 3' mRNA counting cells and need a 2D map that shows
  whether k-means clusters (e.g., k=10 on 50 PCs) form spatially distinct subpopulations.
when_to_use_negative:
- Input is raw sequencing data (FASTQ/BAM) that still requires Cell Ranger demultiplexing,
  barcode processing, and gene counting before a gene-cell matrix exists
- The analysis goal is quantitative cluster assignment or differential expression
  rather than 2D visualization — k-means on PCs is the clustering method here; tSNE
  only visualizes it
- Dataset is small or low-dimensional enough that PCA plus tSNE adds no separable
  structure (e.g., purified single cell lines used for sensitivity validation in the
  paper)
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: R package that receives the log-transformed gene-cell-barcode matrix and is
    used to run PCA (top 1,000 variable genes, top 50 PCs), k-means clustering, and
    the tSNE embedding for visualization of the 68k PBMC clusters
  repo: https://github.com/10XGenomics/single-cell-3prime-paper
- name: Cell Ranger Single-Cell Software Suite
  role: Upstream pipeline that performs sample demultiplexing, barcode processing,
    and single-cell 3' gene counting to produce the gene-cell-barcode matrix fed into
    Seurat for tSNE visualization
  repo: https://github.com/10XGenomics/cellranger
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:3d019a3c7ebafa9a85fb2009372b7fe5b10c65df057c5092f30286bfeedba444
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/single-cell-3prime-paper
  tools:
  - Cell Ranger Single-Cell Software Suite
  - Seurat
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# tSNE embedding visualization of single-cell clusters

## Summary

A Seurat-based visualization step that embeds tens of thousands of single cells (e.g., the 68k PBMC dataset) in 2D with tSNE so that k-means clusters computed on the first 50 principal components can be inspected spatially and cross-checked against cluster-specific markers such as CD3D, NKG7, CD79A, S100A8/S100A9, PF4, and FCER1A.

## When to use

Use when you have a log-transformed gene-cell-barcode matrix from tens of thousands of droplet-based 3' mRNA counting cells and need a 2D map that shows whether k-means clusters (e.g., k=10 on 50 PCs) form spatially distinct subpopulations.

## When NOT to use

- Input is raw sequencing data (FASTQ/BAM) that still requires Cell Ranger demultiplexing, barcode processing, and gene counting before a gene-cell matrix exists
- The analysis goal is quantitative cluster assignment or differential expression rather than 2D visualization — k-means on PCs is the clustering method here; tSNE only visualizes it
- Dataset is small or low-dimensional enough that PCA plus tSNE adds no separable structure (e.g., purified single cell lines used for sensitivity validation in the paper)

## Inputs

- Gene-cell-barcode count matrix from droplet-based single-cell 3' counting (e.g., 68k fresh PBMCs from support.10xgenomics.com/single-cell/datasets)
- Log-transformed expression matrix loaded into Seurat
- Per-cell cluster assignments from k-means (k=10) on the first 50 PCs

## Outputs

- 2D tSNE embedding of all cells with points colored by k-means cluster identity
- Per-cell cluster assignment table written to pbmc_clusters_markers.csv
- Per-cluster top marker gene table written to pbmc_clusters_markers.csv

## How to apply

Load the gene-cell-barcode matrix in R and log-transform it as input to Seurat, then select the top 1,000 most variable genes based on their mean and dispersion (variance/mean), following the Macosko-style approach used in the paper. Run PCA on those variable genes and retain the top 50 principal components. Cluster cells with k-means (k=10) on the 50 PCs, then compute the tSNE embedding of the same 50-PC space so each cell is plotted as one point colored by its cluster assignment. Rationale: tSNE is used purely for visualization of the PC-space structure — the clustering itself is k-means on PCs, so disagreement between tSNE layout and cluster boundaries signals that a different PC count or k should be tried. Validate the embedding by checking that clusters occupying distinct tSNE regions are enriched for canonical PBMC markers (CD3D for T cells in clusters 1–3 and 6, NKG7 for NK cells, CD79A for B cells, S100A8/S100A9 for monocytes, PF4 for megakaryocytes/platelets, FCER1A for dendritic cells).

## Related tools

- **Seurat** (R package that receives the log-transformed gene-cell-barcode matrix and is used to run PCA (top 1,000 variable genes, top 50 PCs), k-means clustering, and the tSNE embedding for visualization of the 68k PBMC clusters) — https://github.com/10XGenomics/single-cell-3prime-paper
- **Cell Ranger Single-Cell Software Suite** (Upstream pipeline that performs sample demultiplexing, barcode processing, and single-cell 3' gene counting to produce the gene-cell-barcode matrix fed into Seurat for tSNE visualization) — https://github.com/10XGenomics/cellranger

## Examples

```
pbmc <- CreateSeuratObject(log(mat)); pbmc <- FindVariableGenes(pbmc); pbmc <- RunPCA(pbmc); pbmc <- RunTSNE(pbmc, dims.embed = 1:50)
```

## Evaluation signals

- 10 distinct clusters are recovered from k-means on the first 50 PCs, and each occupies a spatially coherent region in the tSNE embedding
- Cluster-specific marker enrichment matches canonical PBMC populations: CD3D in T-cell clusters (1–3 and 6, >80% T cells), NKG7 in NK, CD79A in B, S100A8/S100A9 in monocytes, PF4 in megakaryocytes/platelets, FCER1A in dendritic cells
- Top 1,000 variable genes were selected by mean and dispersion (variance/mean), Macosko-style, before PCA — check this filter was applied prior to the PC/tSNE computation
- Subpopulations appear at expected PBMC ratios, consistent with the published finding
- Per-cell cluster assignments and per-cluster marker table are written to pbmc_clusters_markers.csv

## Limitations

- tSNE layout is only a visualization of the 50-PC space; it is not itself the clustering method, so cluster boundaries from k-means may cut across visual tSNE groupings and require re-tuning of PC count or k
- The paper notes cell-type assignment by correlation was low-confidence for some cells (e.g., cytotoxic T vs NK) because the difference between the highest and second highest correlation was small — such ambiguities will also be hard to resolve visually in tSNE
- Some earlier libraries used 5 nt UMI and 5 bp Read2 instead of the standard configuration, introducing protocol heterogeneity that can affect the count matrix feeding the embedding

## Evidence

- [discussion] The gene-cell-barcode matrix of 68k PBMCs was log-transformed as an input to Seurat.: "The gene-cell-barcode matrix of 68k PBMCs was log-transformed as an input to Seurat."
- [discussion] Top 1,000 variable genes by mean and dispersion: "The top 1,000 most variable genes were identified based on their mean and dispersion (variance/mean), which is similar to the approach used by Macoscko et al.7"
- [discussion] K-means on first 50 PCs recovers 10 clusters with canonical markers: [evidence span withheld — anchor status: not_found]
- [discussion] R code for PBMC analysis is in the public repo: "The R code used to analyse 68k PBMCs and purified PBMCs can be found here: https://github.com/10XGenomics/single-cell-3prime-paper."
- [readme] Data downloadable from the 10x datasets site: [evidence span withheld — anchor status: not_found]
