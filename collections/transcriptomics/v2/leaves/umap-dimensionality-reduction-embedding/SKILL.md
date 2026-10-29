---
name: umap-dimensionality-reduction-embedding
description: Use when a single-cell (scRNA-seq or scATAC-seq) count matrix has been
  normalized and feature-selected, and you need a 2D embedding for cluster visualization,
  annotation, or Monocle 3 trajectory topology.
when_to_use_negative:
- Input is raw or unnormalized counts with no feature selection or PCA yet computed
  — UMAP on uninformative features will not resolve cell types (all-peaks and HVF
  sets failed to recover the hematopoietic topology)
- The goal is quantitative differential expression or marker testing rather than visualization/embedding
  for clustering or trajectory topology
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: Seurat
  role: Hosts the normalized data, runs PCA, FindNeighbors/FindClusters, and computes
    the 2D UMAP coordinates (RunUMAP on dims 1:10); DUBStepR recommends use with Seurat
  repo: https://satijalab.org/seurat/
- name: Monocle 3
  role: Consumes the UMAP embedding to derive differentiation topology (e.g., the
    lymphoid/myeloid/megakaryocyte-erythroid branching structure in scATAC-seq analysis)
- name: DUBStepR
  role: Supplies the upstream feature set whose PCA space the UMAP is computed from;
    without informative features the embedding fails to separate biologically distinct
    clusters
  repo: github.com/prabhakarlab/DUBStepR
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-26085-2
    title: DUBStepR is a scalable correlation-based feature selection method for accurately
      clustering single-cell data
  source_content_hash: sha256:768512c72bbba25fa84a976c06ab701067dadd8d3935474b93f796e99a67454f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DUBStepR
  - Monocle 3
  - R
  - Seurat
  - hdf5r
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# umap-dimensionality-reduction-embedding

## Summary

Generates 2D UMAP cell embeddings from a reduced-dimension (PCA) representation of single-cell data, downstream of DUBStepR feature selection, to visualize and support clustering or trajectory inference of cell types/subtypes.

## When to use

Use when a single-cell (scRNA-seq or scATAC-seq) count matrix has been normalized and feature-selected, and you need a 2D embedding for cluster visualization, annotation, or Monocle 3 trajectory topology.

## When NOT to use

- Input is raw or unnormalized counts with no feature selection or PCA yet computed — UMAP on uninformative features will not resolve cell types (all-peaks and HVF sets failed to recover the hematopoietic topology)
- The goal is quantitative differential expression or marker testing rather than visualization/embedding for clustering or trajectory topology

## Inputs

- Log-normalized single-cell count matrix (gene-by-cell or peak-by-cell) in a Seurat object
- DUBStepR-selected variable feature set (or comparator sets: all features, HVF-selected features)
- PCA representation of the cells (e.g., npcs = 30, using dims 1:10 for embedding)

## Outputs

- 2D UMAP cell-coordinate embedding (dims 1:10) stored in the Seurat object for plotting and annotation
- UMAP embeddings per feature set (DUBStepR vs. all-peaks vs. HVF) used as input to Monocle 3 trajectory inference

## How to apply

After log-normalizing the count matrix (e.g., a peak-by-cell matrix from Buenrostro et al. human bone marrow scATAC-seq, GSE96583, or an 8312-cell RA PBMC Seurat object) and assigning DUBStepR-selected features (optimise.features = TRUE, k = 10, num.pcs = 20, error = 0, min.cells = 0.05*ncol) as variable features, scale features and run PCA (e.g., npcs = 30). Build the neighbor graph on a PC subset — the RA T/NK analysis uses PCs 1:10 with FindNeighbors/FindClusters — then compute 2D UMAP coordinates on the same dims (RunUMAP, dims 1:10). For comparator feature sets (all peaks, HVF-selected peaks), re-run PCA and UMAP per feature set so embeddings are comparable. The resulting UMAP feeds cluster annotation by marker genes (MS4A1, NKG7, CD3E, IL7R, CD8A, FCGR3A) or Monocle 3 trajectory inference; the embedding is only as good as the upstream feature selection, which is why DUBStepR-selected features are recommended.

## Related tools

- **Seurat** (Hosts the normalized data, runs PCA, FindNeighbors/FindClusters, and computes the 2D UMAP coordinates (RunUMAP on dims 1:10); DUBStepR recommends use with Seurat) — https://satijalab.org/seurat/
- **Monocle 3** (Consumes the UMAP embedding to derive differentiation topology (e.g., the lymphoid/myeloid/megakaryocyte-erythroid branching structure in scATAC-seq analysis))
- **DUBStepR** (Supplies the upstream feature set whose PCA space the UMAP is computed from; without informative features the embedding fails to separate biologically distinct clusters) — github.com/prabhakarlab/DUBStepR

## Examples

```
RunUMAP(seurat_obj, dims = 1:10)
```

## Evaluation signals

- UMAP is computed on the same PC dims used for clustering (dims 1:10 in the RA T/NK workflow), keeping embedding and graph consistent
- Cluster annotation on the UMAP must resolve 10 discrete T/NK subtypes with rare clusters at reported frequencies: RBC ~1.8%, proliferating ~2%, platelet-T ~3.4%, platelet-NK ~3% (Fig 6; Supp. Figs. S5, S6)
- For scATAC-seq validation, only the DUBStepR-peak UMAP (via Monocle 3) should recover the three-branch lymphoid/myeloid/megakaryocyte-erythroid topology; all-peaks and HVF UMAPs should fail, matching Fig 7
- Clusters on the UMAP must be distinguishable by marker expression (MS4A1, NKG7, CD3E, IL7R, CD8A, FCGR3A)

## Limitations

- Embedding quality is contingent on upstream feature selection: existing methods can yield poorer clustering accuracy than no feature selection, and DUBStepR approaches its maximum silhouette index value only at 200–500 feature genes (Supp. Fig. S3)
- Key supporting results (elbow-point computation, Supp. Note 1B) are deferred to supplementary materials not included in the main text, so parameter tuning details for the preceding dimensionality choices are not fully documented
- SingleR-style reference panels can mislabel cells (e.g., platelet and RBC transcriptomes absent from the reference), so UMAP-based cluster annotation requires careful marker-based validation

## Evidence

- [other] Source card workflow (RA T/NK): UMAP on dims 1:10: [evidence span withheld — anchor status: not_found]
- [other] Source card workflow (scATAC trajectory): PCA followed by UMAP per feature set: [evidence span withheld — anchor status: not_found]
- [intro] Article pipeline: PCA precedes reduced-dimension embedding/clustering: "which are then typically reduced to a small number of dimensions using principal component analysis (PCA)5"
- [methods] DUBStepR call supplying the feature set upstream of embedding: "`DUBStepR(input.data, min.cells, species, optimise.features, k, num.pcs, error)`"
- [readme] README: DUBStepR rationale for informative features driving the embedding: "DUBStepR is based on the intuition that cell-type-specific marker genes tend to be well correlated with each other"
