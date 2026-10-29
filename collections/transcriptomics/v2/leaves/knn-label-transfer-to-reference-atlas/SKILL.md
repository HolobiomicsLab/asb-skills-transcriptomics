---
name: knn-label-transfer-to-reference-atlas
description: Use when new query datasets have been integrated onto an annotated reference
  (e.g., a STACAS seed map) and cell subtype labels must be propagated to unlabeled
  cells using nearest-neighbor voting in the shared embedding.
when_to_use_negative:
- The query data have not been batch-corrected into a shared embedding with the reference
  — nearest-neighbor voting on uncorrected spaces reflects batch, not biology
- No trustworthy cell type annotations exist on the reference — the article notes
  CiLISI-type and label-based approaches require annotations, and semi-supervised
  STACAS presumes prior labels
- Cell types of interest are absent from the reference — neighbor voting cannot assign
  labels for types not represented in the seed map
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: STACAS
  role: Semi-supervised integration (IntegrateData.STACAS with semisupervised=TRUE)
    places query cells into a shared embedding with the annotated seed, prerequisite
    for KNN label transfer
  repo: https://github.com/carmonalab/STACAS
- name: annotate.by.neighbors
  role: STACAS helper function performing the K-nearest-neighbor label classification
    (obj, ref.cells, reduction, ndim, k, ncores, bg.pseudocount, labels.col)
  repo: https://github.com/carmonalab/STACAS
- name: Seurat
  role: Data structure (Seurat objects) holding the seed map, query datasets, and
    integrated assay on which neighbor voting runs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-45240-z
    title: Semi-supervised integration of single-cell transcriptomics data
  source_content_hash: sha256:0f46caaae64868027e6d72d46580917b843155a32064493e830c20486a68697d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/STACAS
  tools:
  - STACAS
  - Seurat
  - annotate.by.neighbors
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# KNN label transfer to reference atlas

## Summary

Transfers cell subtype annotations from an annotated seed/reference map to newly integrated single-cell datasets by K-nearest-neighbor classification in the integrated embedding (annotate.by.neighbors), enabling label propagation at atlas scale while preserving biological subtype structure.

## When to use

Use when new query datasets have been integrated onto an annotated reference (e.g., a STACAS seed map) and cell subtype labels must be propagated to unlabeled cells using nearest-neighbor voting in the shared embedding.

## When NOT to use

- The query data have not been batch-corrected into a shared embedding with the reference — nearest-neighbor voting on uncorrected spaces reflects batch, not biology
- No trustworthy cell type annotations exist on the reference — the article notes CiLISI-type and label-based approaches require annotations, and semi-supervised STACAS presumes prior labels
- Cell types of interest are absent from the reference — neighbor voting cannot assign labels for types not represented in the seed map

## Inputs

- Annotated seed/reference Seurat object with cell subtype labels (e.g., CD8 T cell subtypes from task_005 seed map)
- Integrated Seurat object containing query cells mapped onto the reference space (e.g., integrated_atlas.rds)
- Integrated embedding/reduction computed by STACAS (e.g., PCA or UMAP of the integrated assay)
- Parameters: k=20 nearest neighbors, labels column in reference metadata

## Outputs

- Label-annotated Seurat object where every query cell carries a transferred subtype label (e.g., integrated_atlas.rds with 571,672 cells across 285 datasets)
- Per-subtype average marker-gene expression profiles for validation against expected CD8 T cell subtype signatures

## How to apply

After semi-supervised integration with IntegrateData.STACAS (semisupervised=TRUE, so the reference's cell type labels constrain batch correction and preserve biological variability), run annotate.by.neighbors on the integrated object, supplying the annotated reference cells (ref.cells), the integrated dimensionality reduction, and k=20 so each new cell inherits the majority label of its top-20 nearest neighbors from the seed map. Root the integration at the annotated seed (e.g., via SampleTree.STACAS) so all query cells lie in a space where neighbor voting is meaningful. In the Utility collection case study, this transferred CD8 T cell subtype labels from the seed to 553,077 newly integrated cells, yielding an integrated collection of 571,672 cells from 285 datasets. Validate by checking per-label average marker-gene expression against expected CD8 T cell subtype signatures rather than relying on label frequency alone.

## Related tools

- **STACAS** (Semi-supervised integration (IntegrateData.STACAS with semisupervised=TRUE) places query cells into a shared embedding with the annotated seed, prerequisite for KNN label transfer) — https://github.com/carmonalab/STACAS
- **annotate.by.neighbors** (STACAS helper function performing the K-nearest-neighbor label classification (obj, ref.cells, reduction, ndim, k, ncores, bg.pseudocount, labels.col)) — https://github.com/carmonalab/STACAS
- **Seurat** (Data structure (Seurat objects) holding the seed map, query datasets, and integrated assay on which neighbor voting runs)

## Examples

```
integrated <- annotate.by.neighbors(obj = integrated_atlas, ref.cells = seed_map, reduction = "STACAS", k = 20, labels.col = "subtype")
```

## Evaluation signals

- Cell count invariant: the integrated, labeled atlas contains exactly 571,672 cells across 285 datasets
- Per-subtype average marker-gene expression profiles of transferred labels are concordant with the seed-derived CD8 T cell subtype signatures (expert review of marker concordance)
- Every query cell receives exactly one label from the reference label set (no NAs or novel labels)
- Labels are stable with respect to k: top-20 neighbor voting (k=20) yields coherent subtype assignments in the integrated embedding

## Limitations

- Label transfer quality depends on the reference annotation; the article notes that even partial and imperfect annotations help, but CiLISI-like cell type-aware evaluation requires annotations and cannot be computed for cell types present in only one dataset
- KNN classification in the integrated space can be suboptimal when datasets have highly unequal numbers of cells (a LISI-related limitation acknowledged in the discussion)
- Neighbor voting inherits the granularity of the seed map: subtypes absent from the reference cannot be discovered in query data

## Evidence

- [methods] annotate.by.neighbors performs KNN label transfer in the integrated embedding: "`annotate.by.neighbors(obj, ref.cells, reduction, ndim, k, ncores, bg.pseudocount, labels.col)`"
- [abstract] Semi-supervised integration leverages prior labels to preserve biological variability: "we present STACAS, a batch correction method for scRNA-seq that leverages prior knowledge on cell types to preserve biological variability upon integration."
- [other] Workflow uses k=20 neighbor classification to label newly integrated cells against the seed map: [evidence span withheld — anchor status: not_found]
- [other] Validation of the labeled collection against expected CD8 T cell subtype signatures: [evidence span withheld — anchor status: not_found]
- [readme] README confirms semi-supervised STACAS integration with prior scGate labels for the CD8 TIL reference: [evidence span withheld — anchor status: not_found]
