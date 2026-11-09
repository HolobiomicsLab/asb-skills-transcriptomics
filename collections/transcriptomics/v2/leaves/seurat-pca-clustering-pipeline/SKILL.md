---
name: seurat-pca-clustering-pipeline
description: Use when you have a log-normalized single-cell Seurat object with a biologically
  informed feature set (e.g., DUBStepR optimal features) and need PCA-based graph
  clustering to resolve discrete cell subtypes.
when_to_use_negative:
- The input is raw unnormalized counts — log-normalization and DUBStepR feature selection
  must precede PCA/clustering.
- You only need the feature selection step itself (gene-gene correlation range scoring,
  stepwise regression) — that is the DUBStepR method, not this clustering pipeline.
- You need cell-level quality control or annotation against a reference — those are
  separate steps (QC discarding low-quality cells; SingleR reference-based labeling).
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: R toolkit used for scaling, PCA (npcs = 30), FindNeighbors on PCs 1:10, FindClusters
    clustering, UMAP projection, and marker-based cluster annotation
  repo: https://satijalab.org/seurat/
- name: DUBStepR
  role: upstream correlation-based feature selection whose optimal feature genes are
    set as Seurat variable features before PCA and clustering
  repo: github.com/prabhakarlab/DUBStepR
- name: hdf5r
  role: used to load the 10X h5 count matrix into the Seurat object via Read10X_h5
- name: SingleR
  role: reference-based annotation used upstream to define the T/NK cell subset that
    this pipeline then clusters into subtypes
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-26085-2
    title: DUBStepR is a scalable correlation-based feature selection method for accurately
      clustering single-cell data
  source_content_hash: sha256:564a57caaeef11ae0107aa7d5a5cb8659a88a7735f6726a39b8f78fbf4cb542a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DUBStepR
  - R
  - Seurat
  - SingleR
  - hdf5r
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# seurat-pca-clustering-pipeline

## Summary

Seurat-based downstream clustering workflow that takes DUBStepR-selected variable features, scales all genes, runs PCA, builds a neighbor graph, clusters cells, and projects UMAP so subtypes (e.g., the 10 T/NK subtypes in RA PBMCs) can be annotated by marker expression.

## When to use

Use when you have a log-normalized single-cell Seurat object with a biologically informed feature set (e.g., DUBStepR optimal features) and need PCA-based graph clustering to resolve discrete cell subtypes.

## When NOT to use

- The input is raw unnormalized counts — log-normalization and DUBStepR feature selection must precede PCA/clustering.
- You only need the feature selection step itself (gene-gene correlation range scoring, stepwise regression) — that is the DUBStepR method, not this clustering pipeline.
- You need cell-level quality control or annotation against a reference — those are separate steps (QC discarding low-quality cells; SingleR reference-based labeling).

## Inputs

- Seurat object with log-normalized, SingleR-annotated cell subset (e.g., 5329 T/NK cells from RA PBMC data loaded from 10X h5 via hdf5r/Read10X_h5)
- DUBStepR optimal feature gene list (assigned as Seurat variable features)

## Outputs

- Seurat object with scaled data, PCA (npcs = 30), SNN graph, cluster identities (FindClusters), and 2D UMAP embedding (dims 1:10)
- Cluster-by-frequency table of annotated subtypes with marker-based cell type labels

## How to apply

Assign the DUBStepR-returned optimal feature genes as the Seurat variable features, then scale all genes and run PCA with npcs = 30. Build the shared nearest-neighbor graph on PCs 1:10 with FindNeighbors and cluster cells with FindClusters (the DUBStepR paper pairs this with k = 10, num.pcs = 20, error = 0 in the DUBStepR call and min.cells = 0.05*ncol for gene filtering). Compute 2D UMAP on dims 1:10 for visualization and annotate each cluster from canonical marker genes (MS4A1, NKG7, CD3E, IL7R, CD8A, FCGR3A, PPBP for platelet doublets, HBB for RBCs, MKI67/top2a for proliferating cells). Correct application is judged by whether the clustering resolves discrete subtypes with rare populations present, matching the paper's reported frequencies (RBC ~1.8%, proliferating ~2%, platelet-T ~3.4%, platelet-NK ~3%).

## Related tools

- **Seurat** (R toolkit used for scaling, PCA (npcs = 30), FindNeighbors on PCs 1:10, FindClusters clustering, UMAP projection, and marker-based cluster annotation) — https://satijalab.org/seurat/
- **DUBStepR** (upstream correlation-based feature selection whose optimal feature genes are set as Seurat variable features before PCA and clustering) — github.com/prabhakarlab/DUBStepR
- **hdf5r** (used to load the 10X h5 count matrix into the Seurat object via Read10X_h5)
- **SingleR** (reference-based annotation used upstream to define the T/NK cell subset that this pipeline then clusters into subtypes)

## Examples

```
ra <- Read10X_h5('ra_pbmc.h5'); seu <- CreateSeuratObject(ra) %>% NormalizeData(); seu <- ScaleData(seu); VariableFeatures(seu) <- dubstep.genes; seu <- RunPCA(seu, npcs = 30) %>% FindNeighbors(dims = 1:10) %>% FindClusters() %>% RunUMAP(dims = 1:10)
```

## Evaluation signals

- FindClusters on PCs 1:10 resolves 10 discrete T/NK subtypes with sharply distinct marker gene signatures.
- Rare clusters match the reported frequencies: RBCs ~1.8%, proliferating cells ~2%, platelet-T doublets ~3.4%, platelet-NK doublets ~3%.
- Cluster marker expression matches expected labels (MS4A1+ B/platelet-T context, NKG7/FCGR3A+ NK, CD3E/IL7R/CD8A+ T subsets).
- Doublet and contaminant clusters (platelet-T, platelet-NK, RBC) are detected — DUBStepR was reported as the only method to robustly deconvolve T and NK heterogeneity including these rare types.
- Cluster stability is reproducible when DUBStepR parameters (min.cells = 0.05*ncol, optimise.features = TRUE, k = 10, num.pcs = 20, error = 0) are held fixed.

## Limitations

- Reference-panel annotation can mislabel cells: SingleR misclassified platelet and RBC transcriptomes as T/NK because platelet and RBC references were absent from the panel; these populations appear as clusters only after clustering.
- DUBStepR performance depends on prior filtering of mitochondrial, ribosomal, spike-in, and pseudogene entries and on the elbow-point choice; elbow point computation details are deferred to supplementary notes.
- Clustering resolution and the number of PCs (the paper uses PCs 1:10 for the graph, 30 for PCA) are analyst-chosen and affect the number of recovered subtypes.

## Evidence

- [abstract] DUBStepR was the only method to robustly deconvolve T and NK heterogeneity by identifying disease-associated common and rare cell types and subtypes in PBMCs from rheumatoid arthritis patients.: "DUBStepR was the only method to robustly deconvolve T and NK heterogeneity by identifying disease-associated common and rare cell"
- [intro] cells are clustered based on their distance from one another (typically, Euclidean distance): "cells are clustered based on their distance from one another (typically, Euclidean distance)"
- [methods] we recommend using DUBStepR with the Seurat package: "we recommend using DUBStepR with the [Seurat](https://satijalab.org/seurat/) (Satija, R. et al. Nature Biotechnol. 33.5(2015):495.) package"
- [intro] the corresponding clusters are assigned a cell type or state label based on the known functions of their differentially expressed (DE) genes: "the corresponding clusters are assigned a cell type or state label based on the known functions of their differentially expressed (DE) genes6"
- [readme] A vignette for using DUBStepR on scRNA-seq data can be accessed using browseVignettes("DUBStepR").: "A vignette for using DUBStepR on scRNA-seq data can be accessed using `browseVignettes("DUBStepR")`."
