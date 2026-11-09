---
name: rare-cell-subtype-frequency-quantification
description: Use when clustering an already-annotated single-cell subset (e.g. SingleR-annotated
  T/NK cells from RA patient PBMCs) and the goal is to resolve rare subtypes and doublets
  and report their cluster frequencies.
when_to_use_negative:
- Input is raw whole-PBMC data without an upstream cell-type annotation — deconvolve
  broad types first rather than applying this subtype-resolution workflow
- Cells are being assigned purely by reference-based annotation (e.g. SingleR) without
  clustering — platelet and RBC transcriptomes are absent from the reference panel,
  so rare doublets are mis-classified
- The dataset lacks disease-associated or rare subpopulations and a generic feature-selection
  pipeline would suffice
edam_operation: http://edamontology.org/operation_3432
tools:
- name: DUBStepR
  role: Correlation-based feature selection (GGC matrix + stepwise regression) with
    DI optimization to select feature genes that resolve rare subtypes
  repo: github.com/prabhakarlab/DUBStepR
- name: Seurat
  role: Log-normalization, subsetting, PCA, FindNeighbors/FindClusters clustering,
    and UMAP visualization of the annotated subset
  repo: https://satijalab.org/seurat/
- name: hdf5r
  role: Loads the 10X HDF5 count matrix into the Seurat object
- name: SingleR
  role: Upstream annotation used to define the T/NK subset prior to DUBStepR deconvolution
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-26085-2
    title: DUBStepR is a scalable correlation-based feature selection method for accurately
      clustering single-cell data
  source_content_hash: sha256:20b572749d9a74445bde270178a59783e37ebfc8a2efcd7aa3e16a7aebfa1ef9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DUBStepR
  - R
  - Seurat
  - SingleR
  - hdf5r
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# rare-cell-subtype-frequency-quantification

## Summary

Deconvolution of an annotated single-cell subset (e.g. SingleR-annotated T/NK cells from rheumatoid arthritis PBMCs) using DUBStepR correlation-based feature selection with DI optimization, to resolve discrete subtypes — including rare clusters like RBCs, proliferating cells, and platelet-T/NK doublets — and quantify their frequencies.

## When to use

Use when clustering an already-annotated single-cell subset (e.g. SingleR-annotated T/NK cells from RA patient PBMCs) and the goal is to resolve rare subtypes and doublets and report their cluster frequencies.

## When NOT to use

- Input is raw whole-PBMC data without an upstream cell-type annotation — deconvolve broad types first rather than applying this subtype-resolution workflow
- Cells are being assigned purely by reference-based annotation (e.g. SingleR) without clustering — platelet and RBC transcriptomes are absent from the reference panel, so rare doublets are mis-classified
- The dataset lacks disease-associated or rare subpopulations and a generic feature-selection pipeline would suffice

## Inputs

- Filtered feature-by-cell scRNA-seq count matrix (10X HDF5 format, loaded with hdf5r/Read10X_h5)
- Seurat object log-normalized and subset to a SingleR-annotated cell-type population (e.g. 5329 T/NK cells from 8312 RA PBMC cells across 4 patients)

## Outputs

- Seurat object with DUBStepR optimal feature genes assigned as variable features
- Cluster table resolving discrete subtypes (e.g. 10 T/NK subtypes) with per-cluster frequencies
- 2D UMAP coordinates (dims 1:10) with marker-gene-based cluster annotations

## How to apply

Log-normalize the filtered feature-by-cell count matrix (e.g. loaded via hdf5r/Read10X_h5) in Seurat, then subset to the SingleR-annotated T/NK population. Run DUBStepR on the log-normalized data with min.cells = 0.05*ncol, optimise.features = TRUE, k = 10, num.pcs = 20, error = 0, and assign the returned optimal feature genes as the Seurat variable features — this correlation-based selection is what lets rare populations be recovered. Scale all genes, run PCA (npcs = 30), build the neighbor graph on PCs 1:10 with FindNeighbors, and cluster with FindClusters. Compute 2D UMAP coordinates on dims 1:10 and annotate the 10 clusters by marker gene expression (e.g. MS4A1, NKG7, CD3E, IL7R, CD8A, FCGR3A). Validate that the cluster table resolves discrete subtypes with rare clusters matching expected frequencies — RBC ~1.8%, proliferating ~2%, platelet-T doublets ~3.4%, platelet-NK doublets ~3%.

## Related tools

- **DUBStepR** (Correlation-based feature selection (GGC matrix + stepwise regression) with DI optimization to select feature genes that resolve rare subtypes) — github.com/prabhakarlab/DUBStepR
- **Seurat** (Log-normalization, subsetting, PCA, FindNeighbors/FindClusters clustering, and UMAP visualization of the annotated subset) — https://satijalab.org/seurat/
- **hdf5r** (Loads the 10X HDF5 count matrix into the Seurat object)
- **SingleR** (Upstream annotation used to define the T/NK subset prior to DUBStepR deconvolution)

## Examples

```
dub.step <- DUBStepR(input.data = tnk.seurat@assays$RNA@data, min.cells = 0.05*ncol(tnk.seurat), optimise.features = TRUE, k = 10, num.pcs = 20, error = 0)
```

## Evaluation signals

- Cluster table resolves 10 discrete T/NK subtypes with sharply distinct gene expression signatures
- Rare cluster frequencies match the reported values: RBC ~1.8%, proliferating ~2%, platelet-T doublets ~3.4%, platelet-NK doublets ~3%
- Marker gene expression (MS4A1, NKG7, CD3E, IL7R, CD8A, FCGR3A) is consistent with assigned cluster labels on the UMAP
- DUBStepR returns an optimal feature set near its maximum silhouette index (approached at 200–500 feature genes)
- Rare doublet/RBC clusters recovered here should be absent or only partially detected by alternative feature selection methods

## Limitations

- Reference-based annotators like SingleR can mis-classify cells as T/NK due to the absence of platelet and RBC transcriptomes in the reference panel, so rare-cluster identity must be confirmed by markers
- Key validation results are deferred to supplementary figures (Supp. Figs. S5, S6) and Supp. Note 1B, not the main text
- The elbow point computation used by DUBStepR is described only in the supplementary notes
- The RA PBMC benchmark dataset is not named or accessioned in the main text

## Evidence

- [abstract] DUBStepR was the only method to robustly deconvolve T and NK heterogeneity: "DUBStepR was the only method to robustly deconvolve T and NK heterogeneity by identifying disease-associated common and rare cell types and subtypes in PBMCs from rheumatoid arthritis patients."
- [discussion] SingleR mis-classification of rare clusters: "It is likely that these cells were mis-classified as T/NK by SingleR due to the absence of platelet and RBC transcriptomes in the reference panel."
- [methods] DUBStepR function signature: "`DUBStepR(input.data, min.cells, species, optimise.features, k, num.pcs, error)`"
- [readme] DUBStepR correlation intuition: "DUBStepR is based on the intuition that cell-type-specific marker genes tend to be well correlated with each other, i.e. they typically have strong positive and negative correlations with other"
