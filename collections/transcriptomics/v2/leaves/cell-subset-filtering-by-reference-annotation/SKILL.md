---
name: cell-subset-filtering-by-reference-annotation
description: Use when a heterogeneous PBMC scRNA-seq object has coarse reference-based
  (SingleR) lineage labels — e.g. a T/NK subset — and the goal is to recluster that
  subset to expose rare subtypes or doublets hidden by the annotation.
when_to_use_negative:
- The input is already a purified, single-cell-type population with no coarse reference
  annotation to refine — direct clustering without subsetting is appropriate.
- The annotation reference panel already contains platelet and RBC transcriptomes,
  so reference-based labels alone resolve the contaminating cell types (the paper
  notes mis-classification arose from their absence).
- The goal is whole-dataset deconvolution rather than deconvolving a pre-annotated
  subset — run feature selection/clustering on all cells instead.
edam_operation: http://edamontology.org/operation_3648
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3934
tools:
- name: SingleR
  role: Reference-based annotation used to define the T/NK cell subset before reclustering
- name: Seurat
  role: Object model, subsetting, normalization, PCA, neighbor graph, clustering (FindNeighbors/FindClusters),
    and UMAP on the annotated subset
- name: hdf5r
  role: Loads the 10X h5 count matrix into the Seurat object
- name: DUBStepR
  role: Correlation-based feature selection on the subset (optimise.features = TRUE)
    to define variable features for clustering
  repo: github.com/prabhakarlab/DUBStepR
- name: R
  role: Implementation language; DUBStepR requires R >= 3.5.0
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-26085-2
    title: DUBStepR is a scalable correlation-based feature selection method for accurately
      clustering single-cell data
  source_content_hash: sha256:bec06c1cd1183225310d25c4bb6a153d0d721309d86c5dd1f97a586df0dca3d1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DUBStepR
  - R
  - Seurat
  - SingleR
  - hdf5r
  edam_operation: http://edamontology.org/operation_3648
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3934
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# cell-subset-filtering-by-reference-annotation

## Summary

Subset a scRNA-seq Seurat object to the cells assigned to a lineage (here the SingleR-annotated T/NK compartment of rheumatoid arthritis PBMCs, 5329 of 8312 cells) so that downstream feature selection and clustering (DUBStepR) can deconvolve substructure — including platelet and RBC 'contaminants' that the reference annotation missed.

## When to use

Use when a heterogeneous PBMC scRNA-seq object has coarse reference-based (SingleR) lineage labels — e.g. a T/NK subset — and the goal is to recluster that subset to expose rare subtypes or doublets hidden by the annotation.

## When NOT to use

- The input is already a purified, single-cell-type population with no coarse reference annotation to refine — direct clustering without subsetting is appropriate.
- The annotation reference panel already contains platelet and RBC transcriptomes, so reference-based labels alone resolve the contaminating cell types (the paper notes mis-classification arose from their absence).
- The goal is whole-dataset deconvolution rather than deconvolving a pre-annotated subset — run feature selection/clustering on all cells instead.

## Inputs

- Filtered feature-by-cell 10X count matrix (.h5, HDF5 format) for RA patient PBMCs (8312 cells, 4 patients)
- Seurat object with log-normalized expression
- SingleR reference-based cell-type annotations identifying the T/NK compartment (5329 cells)

## Outputs

- Seurat object subset to the annotated T/NK cells
- Cluster table resolving 10 discrete T/NK subtypes with cluster-level frequencies
- 2D UMAP coordinates with cluster annotations (dims 1:10)

## How to apply

Load the filtered feature-by-cell count matrix (RA PBMC dataset, 8312 cells from 4 patients, 10X h5 via hdf5r/Read10X_h5) into a Seurat object and log-normalize. Subset the object to the SingleR-annotated T/NK cells (5329 cells), then assign DUBStepR-selected optimal feature genes as variable features by running DUBStepR on the log-normalized subset with min.cells = 0.05*ncol, optimise.features = TRUE, k = 10, num.pcs = 20, error = 0. Scale all genes, run PCA (npcs = 30), build the neighbor graph on PCs 1:10, cluster with FindClusters, and compute UMAP on dims 1:10. Annotate the resulting clusters by marker expression (MS4A1, NKG7, CD3E, IL7R, CD8A, FCGR3A); expect 10 subtypes including rare RBC (~1.8%), proliferating (~2%), platelet-T (~3.4%), and platelet-NK (~3%) clusters — these arise because SingleR misclassified cells in the absence of platelet and RBC transcriptomes in its reference panel.

## Related tools

- **SingleR** (Reference-based annotation used to define the T/NK cell subset before reclustering)
- **Seurat** (Object model, subsetting, normalization, PCA, neighbor graph, clustering (FindNeighbors/FindClusters), and UMAP on the annotated subset)
- **hdf5r** (Loads the 10X h5 count matrix into the Seurat object)
- **DUBStepR** (Correlation-based feature selection on the subset (optimise.features = TRUE) to define variable features for clustering) — github.com/prabhakarlab/DUBStepR
- **R** (Implementation language; DUBStepR requires R >= 3.5.0)

## Examples

```
dubstepR.res <- DUBStepR(input.data = lognorm.data, min.cells = 0.05*ncol(lognorm.data), optimise.features = TRUE, k = 10, num.pcs = 20, error = 0)
```

## Evaluation signals

- The cluster table resolves exactly 10 discrete subtypes in the SingleR-annotated T/NK subset.
- Rare cluster frequencies match the reported values: RBC ~1.8%, proliferating ~2%, platelet-T doublets ~3.4%, platelet-NK doublets ~3% (Fig 6; Supp. Figs. S5, S6).
- The subset size equals the number of SingleR T/NK calls (5329 of 8312 cells).
- Cluster marker annotations are consistent with expected marker genes (e.g. MS4A1 for B-like/RBC-contaminant clusters, NKG7/FCGR3A for NK, CD3E/CD8A/IL7R for T).
- DUBStepR-selected features (assigned as Seurat variable features) yield the only method that robustly deconvolves T and NK heterogeneity versus other feature selection methods.

## Limitations

- SingleR misclassified platelet and RBC cells as T/NK due to the absence of platelet and RBC transcriptomes in the reference panel, so subset boundaries are only as good as the reference.
- Key supporting results are deferred to supplementary figures (Supp. Figs. S3–S6) and notes (Supp. Note 1B) not fully described in the main text.

## Evidence

- [discussion] SingleR reference-panel gap motivates subsetting/reclustering: "It is likely that these cells were mis-classified as T/NK by SingleR due to the absence of platelet and RBC transcriptomes in the reference panel."
- [abstract] Subsetting enables deconvolution of T/NK heterogeneity: "DUBStepR was the only method to robustly deconvolve T and NK heterogeneity by identifying disease-associated common and rare cell types and subtypes in PBMCs from rheumatoid arthritis patients."
- [methods] DUBStepR applied downstream on the subset for feature selection: "`DUBStepR(input.data, min.cells, species, optimise.features, k, num.pcs, error)`"
- [methods] Seurat integration for the clustering pipeline: "we recommend using DUBStepR with the [Seurat](https://satijalab.org/seurat/) (Satija, R. et al. Nature Biotechnol. 33.5(2015):495.) package"
- [readme] Rationale for correlation-based feature selection on the subset: "DUBStepR is based on the intuition that cell-type-specific marker genes tend to be well correlated with each other, i.e. they typically have strong positive and negative correlations with other"
