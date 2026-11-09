---
name: highly-variable-gene-selection-by-mean-dispersion
description: Use when preparing a raw single-cell UMI count matrix (e.g. gene-cell-barcode
  matrix from Cell Ranger) for dimensionality reduction or clustering, and you need
  a compact set of informative genes.
when_to_use_negative:
- Input is already a reduced feature set or a pre-computed PCA/embedding — variable-gene
  selection would discard information or be redundant
- Data are not single-cell UMI counts (e.g. bulk RNA-seq or a feature-barcode table),
  where the mean-dispersion relationship used here is not the appropriate variability
  model
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: R package that receives the log-transformed gene-cell-barcode matrix and is
    used to compute per-gene mean/dispersion and select the top 1,000 variable genes
    prior to PCA and k-means clustering
  repo: https://github.com/10XGenomics/single-cell-3prime-paper
- name: Cell Ranger Single-Cell Software Suite
  role: Upstream pipeline that produces the gene-cell-barcode matrix (demultiplexing,
    barcode processing, single-cell 3' gene counting) consumed by this variable-gene
    selection step
  repo: https://github.com/10XGenomics/cellranger
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:fc9096379e1915fd75eec5da18044e7020800923d97454e5f53b0f6329021b34
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/single-cell-3prime-paper
  tools:
  - Cell Ranger Single-Cell Software Suite
  - R
  - SSCC
  - Seurat
  - sscClust
  edam_operation: http://edamontology.org/operation_3946
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- variable-gene-selection-by-dispersion
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Highly variable gene selection by mean-dispersion

## Summary

Identifies the top 1,000 most variable genes from a log-transformed gene-cell-barcode matrix based on their mean expression and dispersion (variance/mean), following the Macosko-style approach used in the Zheng et al. 68k PBMC analysis. This feature-selection step focuses downstream PCA and clustering on genes carrying biological signal rather than technical noise.

## When to use

Use when preparing a raw single-cell UMI count matrix (e.g. gene-cell-barcode matrix from Cell Ranger) for dimensionality reduction or clustering, and you need a compact set of informative genes.

## When NOT to use

- Input is already a reduced feature set or a pre-computed PCA/embedding — variable-gene selection would discard information or be redundant
- Data are not single-cell UMI counts (e.g. bulk RNA-seq or a feature-barcode table), where the mean-dispersion relationship used here is not the appropriate variability model

## Inputs

- Gene-cell-barcode count matrix from 10x Genomics Cell Ranger output (e.g. 68k fresh PBMC dataset)
- Log-transformed expression matrix loaded into Seurat

## Outputs

- Ranked list of the top 1,000 most variable genes (by mean and dispersion)
- Feature-reduced expression matrix (1,000 variable genes x 68,000 cells) used as PCA input

## How to apply

Load the gene-cell-barcode matrix (68k PBMC data from support.10xgenomics.com/single-cell/datasets) in R and log-transform it as input to Seurat. For each gene, compute its mean expression and its dispersion, defined as variance/mean, mirroring the approach of Macosko et al. Rank genes by this mean-dispersion relationship and retain the top 1,000 most variable genes. These variable genes then serve as the input features for PCA (top 50 principal components retained) and k-means clustering (k=10) in the downstream 68k PBMC analysis. The rationale is that genes variable across cells after accounting for mean expression capture cell-type identity (e.g. CD3D, NKG7, CD79A, S100A8/S100A9) rather than pure technical drop-out noise.

## Related tools

- **Seurat** (R package that receives the log-transformed gene-cell-barcode matrix and is used to compute per-gene mean/dispersion and select the top 1,000 variable genes prior to PCA and k-means clustering) — https://github.com/10XGenomics/single-cell-3prime-paper
- **Cell Ranger Single-Cell Software Suite** (Upstream pipeline that produces the gene-cell-barcode matrix (demultiplexing, barcode processing, single-cell 3' gene counting) consumed by this variable-gene selection step) — https://github.com/10XGenomics/cellranger

## Examples

```
Rscript -e 'library(Seurat); pbmc <- CreateSeuratObject(read.csv("matrix.csv")); pbmc <- ScaleLog1p(pbmc); hv.genes <- TopVariableGenes(pbmc, n=1000)'
```

## Evaluation signals

- Exactly 1,000 genes are retained as the variable-gene set for downstream PCA
- Per-gene dispersion is computed as variance/mean, consistent with the Macosko-style approach described in the paper
- Selected genes include known canonical PBMC markers (CD3D, NKG7, CD79A, S100A8/S100A9, PF4, FCER1A) so that downstream k-means (k=10) clustering recovers clusters enriched for these markers
- PCA on the 1,000 variable genes retains 50 principal components that support tSNE visualization of 10 distinct clusters matching expected PBMC subtype ratios (e.g. >80% T cells enriched for CD3D in clusters 1–3 and 6)

## Limitations

- Mean-dispersion ranking was tuned for the 68k PBMC dataset; different tissues or lower-depth data may require a different number of variable genes or normalization
- Cell type assignment by correlation can be low-confidence for closely related populations (e.g. cytotoxic T vs NK cells) when correlation differences are small
- Only confidently mapped (MAPQ=255), non-PCR-duplicate reads with valid barcodes and UMIs feed the gene-barcode matrix; filtering artifacts upstream propagate into the variable-gene ranking

## Evidence

- [discussion] The top 1,000 most variable genes were identified based on their mean and dispersion (variance/mean), which is similar to the approach used by Macoscko et al.7: "The top 1,000 most variable genes were identified based on their mean and dispersion (variance/mean), which is similar to the approach used by Macoscko et al.7"
- [discussion] The gene-cell-barcode matrix of 68k PBMCs was log-transformed as an input to Seurat.: "The gene-cell-barcode matrix of 68k PBMCs was log-transformed as an input to Seurat."
- [discussion] The R code used to analyse 68k PBMCs and purified PBMCs can be found here: https://github.com/10XGenomics/single-cell-3prime-paper.: "The R code used to analyse 68k PBMCs and purified PBMCs can be found here: https://github.com/10XGenomics/single-cell-3prime-paper."
- [abstract] We profiled 68k peripheral blood mononuclear cells to demonstrate the system's ability to characterize large immune populations.: "We profiled 68k peripheral blood mononuclear cells to demonstrate the system's ability to characterize large immune populations."
