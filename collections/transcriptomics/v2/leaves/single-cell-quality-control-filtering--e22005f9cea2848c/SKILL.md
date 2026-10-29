---
name: single-cell-quality-control-filtering
description: Use when raw scRNA-seq count matrices (e.g. GEO datasets like GSE145926,
  GSE132509, GSE123926/GSE113197) must be QC-filtered and normalized prior to Seurat
  integration, clustering, and downstream ASGARD drug scoring.
when_to_use_negative:
- Input is already a QC-filtered, integrated Seurat object with cell-type annotations
- Bulk RNA-seq or microarray data is the input — ASGARD QC thresholds (e.g. >6000
  genes, 10% mitochondrial) are specific to single-cell count data
- The dataset lacks enough cells per condition (fewer than min.cells=3 cells per group),
  which ASGARD requires for differential expression
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: QC filtering (min.cells=3, min.features=200), normalization, cell-cycle regression,
    integration, and Louvain clustering of scRNA-seq data
  repo: https://github.com/lanagarmire/Asgard
- name: Asgard
  role: R package whose pipeline consumes the QC-filtered data for drug repurposing
  repo: https://github.com/lanagarmire/Asgard
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-36637-3
    title: ASGARD is A Single-cell Guided Pipeline to Aid Repurposing of Drugs
  source_content_hash: sha256:4cc5d59002d2939d7994d3ab42b31f14899c97044a16620f860b0c656330b03a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/Asgard
  tools:
  - ASGARD
  - Asgard
  - Monocle3
  - R
  - Rtsne
  - SCORPIUS
  - Seurat
  - SingleR
  - celldex
  - cmapR
  - mclust
  - scran
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# single-cell-quality-control-filtering

## Summary

ASGARD's scRNA-seq quality-control and preprocessing step: filter low-quality cells, sparse genes, and cell-cycle effects in Seurat before clustering and drug repurposing, using dataset-specific thresholds per disease cohort.

## When to use

Use when raw scRNA-seq count matrices (e.g. GEO datasets like GSE145926, GSE132509, GSE123926/GSE113197) must be QC-filtered and normalized prior to Seurat integration, clustering, and downstream ASGARD drug scoring.

## When NOT to use

- Input is already a QC-filtered, integrated Seurat object with cell-type annotations
- Bulk RNA-seq or microarray data is the input — ASGARD QC thresholds (e.g. >6000 genes, 10% mitochondrial) are specific to single-cell count data
- The dataset lacks enough cells per condition (fewer than min.cells=3 cells per group), which ASGARD requires for differential expression

## Inputs

- Raw scRNA-seq count matrices from GEO (e.g. GSE145926 BALF, GSE132509 Pre-T ALL, GSE123926/GSE113197 breast epithelial .txt expression matrices)
- Cell-type annotation files (e.g. Normal_celltype.txt from the Single-cell-drug-repositioning repo)

## Outputs

- QC-filtered, normalized, and (optionally) cell-cycle-corrected Seurat object ready for integration
- Highly variable gene set (top 2000 standardized-variance genes) for downstream analysis

## How to apply

In Seurat, remove cells with fewer than 200 unique genes; for the COVID-19 BALF dataset (GSE145926) additionally drop cells with more than 6000 unique genes or more than 10% mitochondrial genes, and remove genes detected in fewer than 3 cells (rowSums(expr>0)<3). For the Pre-T ALL dataset (GSE132509) and breast cancer datasets, also regress out cell cycle using the cc.genes S and G2M gene sets with linear transformation scaling, and use the 2000 genes with highest standardized variance for downstream analysis. Retain cell types with more than min.cells=3 cells in each case/control group before computing limma differential expression signatures. Rationale: single-cell sparsity and heterogeneity cause false DEG discoveries, so careful QC and DE-method choice (limma Bayes showed the best average AUC among Seurat Wilcoxon, DESeq2, edgeR, limma) directly affects ASGARD's drug-ranking accuracy.

## Related tools

- **Seurat** (QC filtering (min.cells=3, min.features=200), normalization, cell-cycle regression, integration, and Louvain clustering of scRNA-seq data) — https://github.com/lanagarmire/Asgard
- **Asgard** (R package whose pipeline consumes the QC-filtered data for drug repurposing) — https://github.com/lanagarmire/Asgard

## Examples

```
epithelial2 <- CreateSeuratObject(counts=data, project="Epithelial", min.cells=3, min.features=200, meta.data=metadata)
```

## Evaluation signals

- No cells with fewer than 200 unique genes, and for BALF data no cells with >6000 genes or >10% mitochondrial genes remain
- Genes detected in fewer than 3 cells (rowSums(expr>0)<3) are absent from the filtered matrix
- Each retained cell type has more than min.cells=3 cells in both case and control groups
- Downstream analysis uses the top 2000 genes by standardized variance
- Post-QC Seurat object integrates all patient samples (e.g. 15 COVID-19 BALF samples or 5 Pre-T ALL samples) and supports Louvain clustering at resolution 0.4

## Limitations

- Sparsity and heterogeneity of single-cell data can cause false discoveries of differentially expressed genes even after QC
- Differential expression method choice affects downstream AUC and must be selected per dataset; limma (Bayes) showed the best average AUC among the four methods tested
- AUC performance varies with cell population sizes, proportions, and differential expression levels

## Evidence

- [discussion] BALF cells from COVID-19 patients with fewer than 200 unique genes, more than 6000 unique genes or have a proportion of mitochondrial genes larger than 10%: "BALF cells from COVID-19 patients with fewer than 200 unique genes, more than 6000 unique genes or have a proportion of mitochondrial genes larger than 10%"
- [discussion] genes removed in few cells: "genes identified in fewer than three cells are removed from the dataset"
- [discussion] cell cycle removal: "We used cell cycle marker genes and linear transformation to scale the expression of each gene and remove the effects of the cell cycle on gene expression."
- [discussion] top 2000 genes: "We used the 2000 genes with the highest standardized variance for downstream analysis"
- [discussion] sparsity caveat: "Sparsity and heterogeneity are two major challenges in analyzing single-cell data, and usually cause false discoveries of differentially expressed genes"
- [readme] README CreateSeuratObject thresholds: "epithelial2 <- CreateSeuratObject(counts=data, project="Epithelial", min.cells=3, 
								  min.features=200, meta.data=metada)"
