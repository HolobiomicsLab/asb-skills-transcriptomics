---
name: kmeans-clustering-on-principal-components
description: Use when analyzing a large scRNA-seq gene-cell-barcode matrix (e.g. 68k
  PBMCs from 10x Genomics) to discover and annotate cell subpopulations via PCA plus
  k-means clustering with marker-based validation.
when_to_use_negative:
- Input is already a per-cluster feature/marker table or precomputed embedding rather
  than a raw gene-cell-barcode matrix
- The goal is chimerism or SNV-based genotype calling (e.g. transplant BMMC samples),
  which uses Freebayes SNV filtering rather than PCA/k-means clustering
- Fewer cells or a different platform where k=10 and the top 1,000 variable genes
  are not appropriate; the paper's parameters were tuned for 68k fresh PBMCs
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3316
tools:
- name: Seurat
  role: R package receiving the log-transformed gene-cell-barcode matrix; used for
    variable gene selection, PCA, k-means clustering and tSNE visualization
  repo: https://github.com/10XGenomics/single-cell-3prime-paper
- name: Cell Ranger Single-Cell Software Suite
  role: Upstream pipeline producing the gene-cell-barcode matrix by sample demultiplexing,
    barcode processing and single-cell 3' gene counting
  repo: 10XGenomics/cellranger
- name: single-cell-3prime-paper R code
  role: Published R analysis code reproducing the 68k PBMC secondary analysis
  repo: https://github.com/10XGenomics/single-cell-3prime-paper
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:7f31ed137b61efaa0807f8bf01e7edbc757e5c505df92298882c9e023222bf2c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/single-cell-3prime-paper
  tools:
  - Cell Ranger Single-Cell Software Suite
  - Seurat
  - single-cell-3prime-paper R code
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# kmeans-clustering-on-principal-components

## Summary

Clustering tens of thousands of single cells (e.g. 68k PBMCs) by running PCA on the top 1,000 most variable genes, retaining the first 50 PCs, and applying k-means (k=10) in R/Seurat to recover PBMC subpopulations defined by canonical markers (CD3D, NKG7, CD79A, S100A8/S100A9, PF4, FCER1A).

## When to use

Use when analyzing a large scRNA-seq gene-cell-barcode matrix (e.g. 68k PBMCs from 10x Genomics) to discover and annotate cell subpopulations via PCA plus k-means clustering with marker-based validation.

## When NOT to use

- Input is already a per-cluster feature/marker table or precomputed embedding rather than a raw gene-cell-barcode matrix
- The goal is chimerism or SNV-based genotype calling (e.g. transplant BMMC samples), which uses Freebayes SNV filtering rather than PCA/k-means clustering
- Fewer cells or a different platform where k=10 and the top 1,000 variable genes are not appropriate; the paper's parameters were tuned for 68k fresh PBMCs

## Inputs

- 68k PBMC gene-cell-barcode count matrix from 10x Genomics (support.10xgenomics.com/single-cell/datasets), log-transformed for Seurat
- Published R analysis code from github.com/10XGenomics/single-cell-3prime-paper
- Human reference genome hg19 alignment produced via Cell Ranger/STAR

## Outputs

- pbmc_clusters_markers.csv containing per-cell cluster assignments and per-cluster marker gene table
- tSNE embedding of 68k PBMCs colored by k-means cluster (k=10)
- Cluster-level marker gene profiles matching canonical PBMC populations

## How to apply

Download the 68k PBMC gene-cell-barcode matrix from support.10xgenomics.com/single-cell/datasets and clone the R analysis code from github.com/10XGenomics/single-cell-3prime-paper. Log-transform the gene-cell-barcode matrix as input to Seurat. Select the top 1,000 most variable genes based on their mean and dispersion (variance/mean), following the Macosko-style approach used in the paper. Run PCA on these variable genes and retain the top 50 principal components, then cluster cells with k-means (k=10) on those 50 PCs and embed with tSNE for visualization. For each cluster, identify top marker genes and confirm recovery of canonical populations: CD3D (T cells), NKG7 (NK cells), CD79A (B cells), S100A8/S100A9 (monocytes), PF4 (megakaryocytes/platelets), and FCER1A (dendritic cells). Write per-cell cluster assignments and per-cluster marker genes to pbmc_clusters_markers.csv.

## Related tools

- **Seurat** (R package receiving the log-transformed gene-cell-barcode matrix; used for variable gene selection, PCA, k-means clustering and tSNE visualization) — https://github.com/10XGenomics/single-cell-3prime-paper
- **Cell Ranger Single-Cell Software Suite** (Upstream pipeline producing the gene-cell-barcode matrix by sample demultiplexing, barcode processing and single-cell 3' gene counting) — 10XGenomics/cellranger
- **single-cell-3prime-paper R code** (Published R analysis code reproducing the 68k PBMC secondary analysis) — https://github.com/10XGenomics/single-cell-3prime-paper

## Examples

```
Rscript pbmc_analysis.R  # clone https://github.com/10XGenomics/single-cell-3prime-paper; log-transform 68k PBMC matrix in Seurat, PCA on top 1,000 variable genes, kmeans k=10 on 50 PCs, write pbmc_clusters_markers.csv
```

## Evaluation signals

- Exactly 10 clusters are recovered from k-means on the first 50 PCs
- Clusters 1–3 and 6 are enriched for CD3D and together account for >80% T cells, matching expected PBMC ratios
- Top cluster-specific genes match canonical markers: CD3D (T), NKG7 (NK), CD79A (B), S100A8/S100A9 (monocytes), PF4 (megakaryocytes/platelets), FCER1A (dendritic cells)
- Variable gene selection identifies the top 1,000 genes ranked by mean and dispersion (variance/mean)
- Output file pbmc_clusters_markers.csv contains both per-cell cluster assignments and per-cluster marker gene rows

## Limitations

- Cell type assignment by correlation was low-confidence for some cells (e.g. cytotoxic T vs NK) due to small correlation differences between top assignments
- Analysis code is provided 'only for informational purposes'; 10x does not support interpreting, modifying, building, or running the cellranger source
- Cluster structure and k=10 choice were validated for the 68k fresh PBMC dataset; other tissues or cell numbers require re-tuning

## Evidence

- [discussion] Top 1,000 most variable genes selected by mean and dispersion for PBMC analysis: "The top 1,000 most variable genes were identified based on their mean and dispersion (variance/mean), which is similar to the approach used by Macoscko et al.7"
- [discussion] Log-transformed matrix as Seurat input: "The gene-cell-barcode matrix of 68k PBMCs was log-transformed as an input to Seurat."
- [discussion] R code repository for PBMC analysis: "The R code used to analyse 68k PBMCs and purified PBMCs can be found here: https://github.com/10XGenomics/single-cell-3prime-paper."
- [discussion] Dataset availability: "Data are also available at http://support.10xgenomics.com/single-cell/datasets."
- [readme] Readme describing included secondary analysis: [evidence span withheld — anchor status: external_doc]
