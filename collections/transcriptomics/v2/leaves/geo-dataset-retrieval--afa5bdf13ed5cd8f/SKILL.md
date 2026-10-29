---
name: geo-dataset-retrieval
description: Use when a task specifies a public scRNA-seq dataset by GEO accession
  (e.g. GSE119945) and the raw count matrix must be downloaded, QC'd, log-normalized,
  and loaded into Seurat before downstream feature-selection benchmarking.
when_to_use_negative:
- The dataset is already a preprocessed/normalized expression matrix or feature table
  rather than raw counts requiring retrieval and QC
- The target data are not scRNA-seq counts hosted under a GEO accession (e.g. local
  proprietary data with no public identifier)
- The task requires the 10X PBMC pbmc_1k_v2 dataset directly, which the article distributes
  via the 10X Genomics support site rather than GEO
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: R framework used to build the Seurat object, run quality control, and log-normalize
    the retrieved count matrix
  repo: https://satijalab.org/seurat/
- name: hdf5r
  role: R package used to load the HDF5-stored count matrix into the Seurat object
- name: DUBStepR
  role: Feature-selection package that defines the R >= 3.5.0 requirement and provides
    logNormalize/getFilteredData for preprocessing the retrieved data
  repo: github.com/prabhakarlab/DUBStepR
- name: R
  role: Runtime environment (>= 3.5.0) in which retrieval, preprocessing, and benchmarking
    are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-26085-2
    title: DUBStepR is a scalable correlation-based feature selection method for accurately
      clustering single-cell data
  source_content_hash: sha256:7dbedb526710c7432f1a1210b54d3a1b924a099f4b77d8e02d591824418fcea0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DUBStepR
  - GiniClust
  - M3Drop
  - R
  - Seurat
  - hdf5r
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# geo-dataset-retrieval

## Summary

Retrieve a public scRNA-seq count matrix from a GEO accession (e.g. the Cao et al. mouse organogenesis dataset, GSE119945) and load it into a Seurat object in R (>= 3.5.0) using hdf5r, so that downsampling and method-benchmarking workflows can operate on the raw counts.

## When to use

Use when a task specifies a public scRNA-seq dataset by GEO accession (e.g. GSE119945) and the raw count matrix must be downloaded, QC'd, log-normalized, and loaded into Seurat before downstream feature-selection benchmarking.

## When NOT to use

- The dataset is already a preprocessed/normalized expression matrix or feature table rather than raw counts requiring retrieval and QC
- The target data are not scRNA-seq counts hosted under a GEO accession (e.g. local proprietary data with no public identifier)
- The task requires the 10X PBMC pbmc_1k_v2 dataset directly, which the article distributes via the 10X Genomics support site rather than GEO

## Inputs

- GEO accession ID (e.g. GSE119945) for the Cao et al. mouse organogenesis scRNA-seq study
- Raw count matrix file (HDF5/h5 or MatrixMarket format) downloaded from GEO
- R >= 3.5.0 environment with Seurat, hdf5r, and DUBStepR packages installed

## Outputs

- A QC-filtered, log-normalized Seurat object of the full million-plus-cell mouse organogenesis dataset
- Random 10,000-cell and 100,000-cell downsamplings of the same dataset for scale-point comparison
- Normalized input matrices ready for feature-selection methods (DUBStepR, HVGDisp, HVGVST, trendVar, devianceFS, M3DropDANB, GiniClust, HLG)

## How to apply

Resolve the GEO accession (GSE119945 for the Cao et al. mouse organogenesis scRNA-seq study) and download the raw count matrix from the GEO depository. Load the counts into a Seurat object using the hdf5r package (the authors' recommended path for HDF5-stored matrices) in R >= 3.5.0, the minimum version required by DUBStepR. Perform quality control to discard low-quality cells, then apply log-normalization with a scale factor (logNormalize in the DUBStepR package wraps this step). For scalability benchmarks, additionally generate random 10,000-cell and 100,000-cell downsamplings while retaining the full million-plus-cell matrix as the largest scale point. Verify success by checking that the Seurat object's dimensions match the expected cells x genes and that normalization precedes feature selection in the pipeline (QC -> normalization -> feature selection -> PCA -> clustering).

## Related tools

- **Seurat** (R framework used to build the Seurat object, run quality control, and log-normalize the retrieved count matrix) — https://satijalab.org/seurat/
- **hdf5r** (R package used to load the HDF5-stored count matrix into the Seurat object)
- **DUBStepR** (Feature-selection package that defines the R >= 3.5.0 requirement and provides logNormalize/getFilteredData for preprocessing the retrieved data) — github.com/prabhakarlab/DUBStepR
- **R** (Runtime environment (>= 3.5.0) in which retrieval, preprocessing, and benchmarking are executed)

## Examples

```
wget 'https://www.ncbi.nlm.nih.gov/geo/download/?acc=GSE119945&format=file' && Rscript -e 'library(Seurat); library(hdf5r); if(!require(DUBStepR)) install.packages("DUBStepR")'
```

## Evaluation signals

- The Seurat object's cell count matches the expected dataset size (1M+ cells for GSE119945; ~1,000 cells for pbmc_1k_v2)
- Quality control removed low-quality cells and log-normalization was applied before any feature selection (QC -> normalization -> feature selection order preserved)
- Downsamplings contain exactly 10,000 and 100,000 cells and are random subsets of the same full matrix
- The R environment reports version >= 3.5.0, satisfying DUBStepR's stated requirement
- The full-scale object is available in the same session so the 100k-to-1M memory comparison (6.4x) can be computed

## Limitations

- Key scalability results are only in supplementary figures (Supp. Fig. S4), so no quantitative thresholds or effect sizes are given in the main text to validate the retrieved data's benchmark behavior
- Benchmark datasets may be identified only by reference number without a named accession (e.g. the benchmark collection is 'reference number 28'), requiring extra resolution work
- Methods like GiniClust and HLG fail at the 100,000-cell scale, so retrieval alone does not guarantee all downstream methods will run

## Evidence

- [methods] load the data in to your Seurat object using the **hdf5r** package: "load the data in to your Seurat object using the **hdf5r** package"
- [methods] R version requirement: "DUBStepR requires your R version to be >= 3.5.0"
- [methods] 10X PBMC dataset retrieval: "we use a publicly available PBMC dataset generated by 10X Genomics. Here's a [link](https://support.10xgenomics.com/single-cell-gene-expression/datasets/3.0.0/pbmc_1k_v2)"
- [intro] pipeline order QC then normalization then feature selection: "Low-quality cells are first discarded in a quality control step3"
- [discussion] scalability claim supporting the 1M-cell retrieval target: "DUBStepR scales effectively to datasets of over a million cells without sharp increases in time or memory consumption (Supp. Fig. S4)."
- [readme] README vignette for usage: "A vignette for using DUBStepR on scRNA-seq data can be accessed using `browseVignettes("DUBStepR")`."
