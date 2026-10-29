---
name: pca-cell-embedding-feature-set-sweep
description: Use when an scRNA-seq pipeline must decide how many top-ranked feature
  genes (DUBStepR-ordered) to retain before PCA and clustering, and cell type annotations
  may not exist for tuning that number.
when_to_use_negative:
- Input is already a fixed feature table or expression matrix with a predetermined
  feature count — no size sweep is needed
- Data has not been QC'd and normalized (low-quality cells discarded, reads log-normalized)
  as required before DUBStepR feature ordering
- Goal is general benchmarking of feature selection methods rather than choosing a
  feature set size for one DUBStepR-ordered ranking
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: DUBStepR
  role: Performs correlation-based feature ordering (GGC matrix, correlation range
    z-scores, stepwise regression seed features) that supplies the feature ranking
    swept over
  repo: github.com/prabhakarlab/DUBStepR
- name: Seurat
  role: Recommended environment for loading the scRNA-seq data and performing the
    PCA cell embeddings used at each feature set size
- name: R
  role: Implementation language; DUBStepR requires R version >= 3.5.0
- name: hdf5r
  role: Used to load 10X HDF5 data into the Seurat object
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-26085-2
    title: DUBStepR is a scalable correlation-based feature selection method for accurately
      clustering single-cell data
  source_content_hash: sha256:0b0f2b4d6396d2e19f7cad9cb26d8f4dbf4c1e66766ba6bbfcc86a5cd9b01911
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DUBStepR
  - R
  - Seurat
  - hdf5r
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# pca-cell-embedding-feature-set-sweep

## Summary

Sweeps increasing DUBStepR-ordered feature set sizes, embedding cells in PCA space at each size to compute label-free (Density Index) and label-based (Silhouette Index) clustering-quality curves, so the optimal feature set size can be chosen without cell type labels. Grounded in the DUBStepR Nature Communications 2021 workflow (Seurat/DUBStepR, k = 10, num.pcs = 20).

## When to use

Use when an scRNA-seq pipeline must decide how many top-ranked feature genes (DUBStepR-ordered) to retain before PCA and clustering, and cell type annotations may not exist for tuning that number.

## When NOT to use

- Input is already a fixed feature table or expression matrix with a predetermined feature count — no size sweep is needed
- Data has not been QC'd and normalized (low-quality cells discarded, reads log-normalized) as required before DUBStepR feature ordering
- Goal is general benchmarking of feature selection methods rather than choosing a feature set size for one DUBStepR-ordered ranking

## Inputs

- Preprocessed scRNA-seq count matrix loaded into a Seurat object (e.g. pbmc_1k_v2 from 10X Genomics, read via hdf5r)
- DUBStepR-ordered feature gene list from correlation range z-score ranking
- Optional per-cell cell type annotations for computing the Silhouette Index

## Outputs

- Per-dataset table of Density Index vs feature set size
- Per-dataset table of Silhouette Index vs feature set size
- Agreement record (di_si_agreement_record) of DI-argmax vs SI-argmax concordance per dataset (expected: 5 of 7)

## How to apply

Load the annotated benchmark scRNA-seq datasets in R (>= 3.5.0) within the DUBStepR/Seurat environment, run the DUBStepR correlation-based feature ordering, and embed cells in PC space (DUBStepR defaults: k = 10, num.pcs = 20) for a sweep of feature set sizes. At each size, compute the Density Index in PC space as the root-mean-squared pairwise distance over the mean k-NN distance (RMS term via the Frobenius norm of the PC coordinate matrix), and, when labels are available, compute the Silhouette Index of the labeled clustering in the same PC space. Tabulate DI and SI versus feature set size per dataset and identify each metric's argmax (the paper reports maximum SI at 200–500 feature genes). Compare DI-argmax and SI-argmax per dataset to record agreement; the expected outcome is that DI and SI peak at approximately the same size and coincide in 5 of 7 benchmark datasets, validating DI as a label-free proxy for SI (Fig 5c,d).

## Related tools

- **DUBStepR** (Performs correlation-based feature ordering (GGC matrix, correlation range z-scores, stepwise regression seed features) that supplies the feature ranking swept over) — github.com/prabhakarlab/DUBStepR
- **Seurat** (Recommended environment for loading the scRNA-seq data and performing the PCA cell embeddings used at each feature set size)
- **R** (Implementation language; DUBStepR requires R version >= 3.5.0)
- **hdf5r** (Used to load 10X HDF5 data into the Seurat object)

## Examples

```
library(DUBStepR); res <- DUBStepR(input.data, min.cells = 5, species = "Homo_sapiens", optimise.features = TRUE, k = 10, num.pcs = 20, error = 0)
```

## Evaluation signals

- The agreement record states DI-maximizing and SI-maximizing feature set sizes coincide in 5 of 7 benchmark datasets, matching Fig 5c,d
- DI and SI curves per dataset are positively correlated and attain maxima at approximately the same feature set size
- Maximum SI is reached within the reported 200–500 feature gene range (Supp. Fig. S3)
- PCA embeddings use DUBStepR defaults k = 10 and num.pcs = 20 consistently across the sweep
- Both curves are computed in the same PC space for each feature set size, so DI vs SI comparisons are apples-to-apples

## Limitations

- The DI–SI concordance was validated on a stringently annotated collection of only 7 benchmark datasets (reference 28); agreement is not guaranteed on every dataset
- Key results (max SI at 200–500 genes, elbow point algorithm) are deferred to supplementary figures/notes (Supp. Fig. S3, Supp. Note 1B) not fully described in the main text
- Benchmark dataset collection is identified only by reference number 28, without named accessions, limiting exact reproduction

## Evidence

- [discussion] we used a stringently defined collection of single-cell datasets for which cell type annotations could be independently ascertained28.: "we used a stringently defined collection of single-cell datasets for which cell type annotations could be independently ascertained28."
- [discussion] DUBStepR approaches its maximum silhouette index value at 200–500 feature genes: "Intriguingly, DUBStepR approaches its maximum silhouette index value at 200–500 feature genes (Supp. Fig. S3)"
- [methods] DUBStepR(input.data, min.cells, species, optimise.features, k, num.pcs, error): "`DUBStepR(input.data, min.cells, species, optimise.features, k, num.pcs, error)`"
- [methods] getOptimalFeatureSet(filt.data, ordered.genes, elbow.pt, k, num.pcs, error): "`getOptimalFeatureSet(filt.data, ordered.genes, elbow.pt, k, num.pcs, error)`"
- [methods] DUBStepR requires your R version to be >= 3.5.0: "DUBStepR requires your R version to be >= 3.5.0"
- [methods] we recommend using DUBStepR with the Seurat package: "we recommend using DUBStepR with the [Seurat](https://satijalab.org/seurat/) (Satija, R. et al. Nature Biotechnol. 33.5(2015):495.) package"
- [readme] After filtering genes based on a correlation range score, DUBStepR exploits structure in the gene–gene correlation matrix: "After filtering genes based on a correlation range score, DUBStepR exploits structure in the gene-gene correlation matrix to prioritize genes as features for clustering."
