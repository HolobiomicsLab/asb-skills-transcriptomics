---
name: gene-gene-correlation-matrix-analysis
description: Use when building or analyzing a gene-gene correlation matrix from log-normalized
  scRNA-seq data to score genes, identify well-correlated marker genes, or derive
  feature sets for cell-type clustering.
when_to_use_negative:
- Input is already a curated feature/variable gene list — GGC-based prioritization
  is meant to replace gene detection-based feature selection
- Data has not been normalized or filtered for low-detection genes; correlation structure
  on raw counts is dominated by library-size and dropout artifacts
- Gene-gene correlations are not of interest, e.g. single-cell ATAC-seq peak clustering
  where the marker-scale correlation assumption underlying DUBStepR does not directly
  transfer
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: DUBStepR
  role: Provides getGGC to compute the gene-gene correlation matrix and getCorrelationRange/getFilteredData/logNormalize
    for the surrounding normalization, filtering, and correlation-range scoring steps
  repo: github.com/prabhakarlab/DUBStepR
- name: R
  role: Implementation environment for the DUBStepR package (requires R >= 3.5.0)
- name: hdf5r
  role: Loads the 10X filtered feature-barcode HDF5 matrix (pbmc_1k_v2) into R for
    GGC construction
- name: Seurat
  role: Recommended for building the Seurat object and log-normalizing (LogNormalize)
    the counts that feed getGGC
provenance:
  source_task_ids:
  - task_007
  - task_008
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-26085-2
    title: DUBStepR is a scalable correlation-based feature selection method for accurately
      clustering single-cell data
  source_content_hash: sha256:6b3adf8e0025b3320cce96168dc4b4d4e1d3ffe59fe61429bd60365d5ebca70c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DUBStepR
  - Matrix
  - R
  - Seurat
  - graphics
  - hdf5r
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# gene-gene-correlation-matrix-analysis

## Summary

Compute and analyze a gene-gene correlation (GGC) matrix from normalized single-cell RNA-seq data, as done in DUBStepR, where structure in the GGC matrix (correlation range z-scores, stepwise regression, guilt-by-association ordering) is exploited to prioritize cell-type-specific marker genes as clustering features.

## When to use

Use when building or analyzing a gene-gene correlation matrix from log-normalized scRNA-seq data to score genes, identify well-correlated marker genes, or derive feature sets for cell-type clustering.

## When NOT to use

- Input is already a curated feature/variable gene list — GGC-based prioritization is meant to replace gene detection-based feature selection
- Data has not been normalized or filtered for low-detection genes; correlation structure on raw counts is dominated by library-size and dropout artifacts
- Gene-gene correlations are not of interest, e.g. single-cell ATAC-seq peak clustering where the marker-scale correlation assumption underlying DUBStepR does not directly transfer

## Inputs

- Log-normalized filtered gene x cell scRNA-seq count matrix (e.g. from 10X pbmc_1k_v2 HDF5 loaded via hdf5r into R)
- Gene filtering parameters such as min.cells = 0.05 * ncol(seuratObj)

## Outputs

- Gene-gene correlation (GGC) matrix (R matrix, e.g. saved as candidate_ggc_matrix.rds)
- Gene-level score table with correlation range scores and expression-binned z-scores (gene_correlation_range_scores.csv)
- Candidate feature gene set (genes with correlation range z-score > 0.7)

## How to apply

Log-normalize the raw counts (DUBStepR's logNormalize with a scale.factor) and filter genes detected in too few cells (e.g. min.cells = 0.05*ncol with getFilteredData, or fewer than 5% of cells in the pbmc_1k_v2 workflow). After removing mitochondrial, ribosomal, spike-in, and pseudogenes, compute the GGC matrix with DUBStepR::getGGC on the filtered data. Downstream, use getCorrelationRange to score each gene as c_i = max3(G_i) - 0.75*min(G_i), bin genes by mean expression, convert range scores to within-bin z-scores, and retain genes with z-score > 0.7 as well-correlated candidates. The GGC submatrix of candidate genes can then feed stepwise regression (runStepwiseReg) and guilt-by-association expansion to derive the final feature set.

## Related tools

- **DUBStepR** (Provides getGGC to compute the gene-gene correlation matrix and getCorrelationRange/getFilteredData/logNormalize for the surrounding normalization, filtering, and correlation-range scoring steps) — github.com/prabhakarlab/DUBStepR
- **R** (Implementation environment for the DUBStepR package (requires R >= 3.5.0))
- **hdf5r** (Loads the 10X filtered feature-barcode HDF5 matrix (pbmc_1k_v2) into R for GGC construction)
- **Seurat** (Recommended for building the Seurat object and log-normalizing (LogNormalize) the counts that feed getGGC)

## Examples

```
ggc <- DUBStepR::getGGC(DUBStepR::getFilteredData(logNormalize(raw.data), min.cells = 0.05*ncol(seuratObj), species = 'human')$filt.data)
```

## Evaluation signals

- The GGC matrix is symmetric with unit diagonal and its dimensions equal the filtered gene count x filtered gene count
- Gene score table contains one row per gene with correlation range-score and z-score columns
- Retained candidate set consists exactly of genes with correlation range z-score > 0.7 within their expression bins
- Input genes excluded mitochondrial, ribosomal, spike-in, and pseudogene entries before GGC construction
- Genes passed min.cells filtering (detected in at least 5% of cells, e.g. 0.05*ncol) prior to correlation computation

## Limitations

- Only a handful of systematic benchmarking studies of scRNA-seq feature selection methods exist, and existing methods perform inconsistently across datasets
- Benchmarks and thresholds (e.g. z-score > 0.7, min.cells = 0.05*ncol) were validated on PBMC-type scRNA-seq data such as 10X pbmc_1k_v2; transfer to other data types needs care although the authors claim applicability to e.g. single-cell ATAC-seq
- Genes detected in fewer than min.cells cells are dropped before GGC computation, so very sparse markers never enter the correlation analysis

## Evidence

- [intro] After filtering out mitochondrial, ribosomal, spike-in, and pseudogenes, DUBStepR constructs a GGC matrix: "After filtering out mitochondrial, ribosomal, spike-in, and pseudogenes, DUBStepR constructs a GGC matrix"
- [intro] DUBStepR bins genes by expression to compute correlation range z-scores: "DUBStepR constructs a GGC matrix and bins genes by expression to compute their correlation range z-scores, which are used to select well-correlated genes."
- [readme] README: DUBStepR exploits structure in the gene-gene correlation matrix to prioritize genes as features for clustering: "After filtering genes based on a correlation range score, DUBStepR exploits structure in the gene-gene correlation matrix to prioritize genes as features for clustering."
- [methods] Methods: correlation range function signature: "`getCorrelationRange(correlation_matrix)`"
- [methods] Methods: GGC computation and filtering function signatures: "`getFilteredData(data, min.cells, species)`"
- [methods] Methods: R version requirement for DUBStepR: "DUBStepR requires your R version to be >= 3.5.0"
