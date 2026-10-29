---
name: single-cell-clustering-of-pbmc-transcriptomes
description: Use when a gene-cell-barcode matrix from droplet scRNA-seq (tens of thousands
  of cells) must be partitioned into cell subpopulations and characterized by cluster-specific
  marker genes.
when_to_use_negative:
- Input is raw sequencing output (BCL/FASTQ) that still requires Cell Ranger demultiplexing,
  barcode processing, and 3' gene counting before a gene-cell matrix exists.
- The goal is SNV-based host/donor chimerism inference from bone marrow mononuclear
  cells rather than unsupervised population discovery.
- The sample is a purified or low-complexity cell mixture where pre-defined k (e.g.,
  k=10 tuned to heterogeneous PBMCs) is inappropriate.
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: R framework into which the log-transformed gene-cell-barcode matrix is loaded
    for variable-gene selection, PCA, k-means clustering, tSNE embedding, and marker
    identification
- name: Cell Ranger Single-Cell Software Suite
  role: upstream pipeline producing the gene-cell-barcode matrix via sample demultiplexing,
    barcode processing, and single-cell 3' gene counting
  repo: https://github.com/10XGenomics/cellranger
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:1f00cd606d3ffff0bdbece73ef14b8e1eedfe18439c2bbe4712ea8f6fb526548
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/cellranger
  tools:
  - Cell Ranger Single-Cell Software Suite
  - Seurat
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# single-cell-clustering-of-pbmc-transcriptomes

## Summary

Unsupervised clustering of large droplet-based scRNA-seq matrices (e.g., 68k PBMCs) in Seurat: select top 1,000 variable genes, run PCA, k-means on the top 50 PCs, and identify per-cluster marker genes to recover canonical immune populations.

## When to use

Use when a gene-cell-barcode matrix from droplet scRNA-seq (tens of thousands of cells) must be partitioned into cell subpopulations and characterized by cluster-specific marker genes.

## When NOT to use

- Input is raw sequencing output (BCL/FASTQ) that still requires Cell Ranger demultiplexing, barcode processing, and 3' gene counting before a gene-cell matrix exists.
- The goal is SNV-based host/donor chimerism inference from bone marrow mononuclear cells rather than unsupervised population discovery.
- The sample is a purified or low-complexity cell mixture where pre-defined k (e.g., k=10 tuned to heterogeneous PBMCs) is inappropriate.

## Inputs

- gene-cell-barcode count matrix from the 68k fresh PBMC 10x Genomics dataset (log-transformed before Seurat input)
- analysis R code from github.com/10XGenomics/single-cell-3prime-paper
- choice of k (10) and number of PCs (50) for k-means clustering

## Outputs

- per-cell cluster assignments for the 68k PBMCs (10 clusters)
- per-cluster marker gene table written to pbmc_clusters_markers.csv
- tSNE embedding for cluster visualization

## How to apply

Download the 68k PBMC gene-cell-barcode matrix (support.10xgenomics.com/single-cell/datasets) and clone the analysis code from github.com/10XGenomics/single-cell-3prime-paper. Load the matrix in R and log-transform it as input to Seurat. Identify the top 1,000 most variable genes based on mean and dispersion (variance/mean), following the Macosko-style approach, run PCA on these genes, and retain the top 50 principal components. Cluster cells with k-means (k=10) on the 50 PCs and embed with tSNE for visualization. For each cluster, extract top marker genes and check recovery of canonical PBMC markers: CD3D (T cells), NKG7 (NK cells), CD79A (B cells), S100A8/S100A9 (monocytes), PF4 (megakaryocytes/platelets), and FCER1A (dendritic cells).

## Related tools

- **Seurat** (R framework into which the log-transformed gene-cell-barcode matrix is loaded for variable-gene selection, PCA, k-means clustering, tSNE embedding, and marker identification)
- **Cell Ranger Single-Cell Software Suite** (upstream pipeline producing the gene-cell-barcode matrix via sample demultiplexing, barcode processing, and single-cell 3' gene counting) — https://github.com/10XGenomics/cellranger

## Examples

```
git clone https://github.com/10XGenomics/single-cell-3prime-paper && Rscript pbmc3k.R  # per README: data from support.10xgenomics.com/single-cell/datasets, matrix log-transformed into Seurat
```

## Evaluation signals

- Exactly 10 clusters are recovered from k-means (k=10) on the first 50 PCs of the 68k PBMC data.
- Clusters 1–3 and 6 are enriched for CD3D and together account for >80% T cells, matching expected PBMC ratios.
- Cluster marker profiles match canonical populations: NKG7 for NK cells, CD79A for B cells, S100A8/S100A9 for monocytes, PF4 for megakaryocytes/platelets, FCER1A for dendritic cells.
- pbmc_clusters_markers.csv is produced with per-cell cluster assignments and a per-cluster marker gene table that passes schema checks.
- tSNE embedding shows spatially coherent, separated clusters consistent with the k-means partition.

## Limitations

- Cell type assignment by correlation was low-confidence for some cells due to small correlation differences (e.g., cytotoxic T vs NK cells).
- k=10 and the top-1,000 variable gene selection were tuned to heterogeneous fresh PBMCs; other tissues or purified samples may require different k or gene-set size.
- Selecting too few clusters (analogous to K=1 selection elsewhere in the paper) only indicates populations below the method's detection level, so rare subpopulations may be missed.

## Evidence

- [discussion] The gene-cell-barcode matrix of 68k PBMCs was log-transformed as an input to Seurat.: "The gene-cell-barcode matrix of 68k PBMCs was log-transformed as an input to Seurat."
- [discussion] Top 1,000 variable genes by mean and dispersion: "The top 1,000 most variable genes were identified based on their mean and dispersion (variance/mean), which is similar to the approach used by Macoscko et al.7"
- [discussion] R code for the 68k PBMC analysis: "The R code used to analyse 68k PBMCs and purified PBMCs can be found here: https://github.com/10XGenomics/single-cell-3prime-paper."
- [discussion] Dataset availability: "Data are also available at http://support.10xgenomics.com/single-cell/datasets."
- [readme] All described data downloadable from 10x datasets site: [evidence span withheld — anchor status: not_found]
