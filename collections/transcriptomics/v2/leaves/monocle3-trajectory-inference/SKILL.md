---
name: monocle3-trajectory-inference
description: Use when a single-cell expression or scATAC-seq dataset must be ordered
  along a differentiation hierarchy — e.g., testing whether a candidate feature set
  (DUBStepR peaks vs all peaks vs HVFs) recovers a known branching trajectory from
  a UMAP embedding.
when_to_use_negative:
- When cell labels are already known and no ordering along a differentiation hierarchy
  is needed — a DUBStepR + clustering benchmark (silhouette-based) is the appropriate
  evaluation instead
- When the feature set itself is not yet defined — trajectory inference on unfiltered
  all-peaks/all-genes data can fail to recover known topology (all-peaks and HVF sets
  failed the hematopoietic topology test)
- When DUBStepR correlation assumptions do not apply — the method presumes cell-type-specific
  markers 'tend to be well correlated with each other'
tools:
- name: Monocle 3
  role: Trajectory inference on UMAP embeddings to derive differentiation topology
    (three-branch hematopoietic hierarchy)
- name: DUBStepR
  role: Correlation-based feature (gene/peak) selection whose output set is evaluated
    by the Monocle 3 trajectory; called with optimise.features = TRUE, k = 10, num.pcs
    = 20, error = 0
  repo: github.com/prabhakarlab/DUBStepR
- name: Seurat
  role: Recommended container for loading data (hdf5r) and building PCA/UMAP embeddings
    that feed Monocle 3
- name: R
  role: Analysis environment; DUBStepR requires R >= 3.5.0
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-26085-2
    title: DUBStepR is a scalable correlation-based feature selection method for accurately
      clustering single-cell data
  source_content_hash: sha256:9398bdb411b617ddb46f67450abed6a0aedc1e36119edcf7de32e7808df9cf24
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DUBStepR
  - Monocle 3
  - R
  - Seurat
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# monocle3-trajectory-inference

## Summary

Applies Monocle 3 trajectory inference on dimensionality-reduced single-cell embeddings (PCA + UMAP) to recover differentiation topology, e.g. the lymphoid/myeloid/megakaryocyte-erythroid branching structure of human hematopoiesis. In this context it serves as the downstream readout of DUBStepR-based feature selection.

## When to use

Use when a single-cell expression or scATAC-seq dataset must be ordered along a differentiation hierarchy — e.g., testing whether a candidate feature set (DUBStepR peaks vs all peaks vs HVFs) recovers a known branching trajectory from a UMAP embedding.

## When NOT to use

- When cell labels are already known and no ordering along a differentiation hierarchy is needed — a DUBStepR + clustering benchmark (silhouette-based) is the appropriate evaluation instead
- When the feature set itself is not yet defined — trajectory inference on unfiltered all-peaks/all-genes data can fail to recover known topology (all-peaks and HVF sets failed the hematopoietic topology test)
- When DUBStepR correlation assumptions do not apply — the method presumes cell-type-specific markers 'tend to be well correlated with each other'

## Inputs

- Log-normalized peak-by-cell scATAC-seq count matrix (e.g., Buenrostro et al. human bone marrow, GSE96583)
- Feature-selected gene/peak matrix from DUBStepR (plus all-peaks and HVF comparator sets)
- PCA + UMAP cell embeddings (e.g., built with Seurat)

## Outputs

- Monocle 3 differentiation trajectory/topology per feature set (branching pseudotime graph)
- Comparison of inferred topologies against the known hematopoietic hierarchy (three-branch lymphoid/myeloid/megakaryocyte-erythroid structure)

## How to apply

First reduce the feature-selected count matrix (e.g., DUBStepR-selected peaks/genes) to low dimension via PCA, then embed cells with UMAP. Run Monocle 3 trajectory inference on each UMAP embedding to derive the differentiation topology. In the scATAC-seq benchmark (Buenrostro et al. human bone marrow, GSE96583), the log-normalized peak accessibility matrix was analyzed with DUBStepR using optimise.features = TRUE, k = 10, num.pcs = 20, error = 0, and min.cells set to 5% of cells before PCA/UMAP and Monocle 3. Compare inferred topologies across feature sets: acceptance requires the DUBStepR-selected set to yield the three-branch lymphoid/myeloid/megakaryocyte-erythroid topology while all-peaks and HVF sets fail. Rationale: DUBStepR exploits gene-gene (or peak-peak) correlation structure to select cell-type-specific markers, and 'is scalable to over a million cells, and can be straightforwardly applied to other data types such as single-cell ATAC-seq'.

## Related tools

- **Monocle 3** (Trajectory inference on UMAP embeddings to derive differentiation topology (three-branch hematopoietic hierarchy))
- **DUBStepR** (Correlation-based feature (gene/peak) selection whose output set is evaluated by the Monocle 3 trajectory; called with optimise.features = TRUE, k = 10, num.pcs = 20, error = 0) — github.com/prabhakarlab/DUBStepR
- **Seurat** (Recommended container for loading data (hdf5r) and building PCA/UMAP embeddings that feed Monocle 3)
- **R** (Analysis environment; DUBStepR requires R >= 3.5.0)

## Examples

```
DUBStepR(input.data, min.cells, species, optimise.features = TRUE, k = 10, num.pcs = 20, error = 0)
```

## Evaluation signals

- DUBStepR-selected feature set's Monocle 3 trajectory recovers the three-branch lymphoid/myeloid/megakaryocyte-erythroid topology matching Fig 7
- All-peaks and HVF comparator sets fail to recover the known hematopoietic differentiation hierarchy on the same Buenrostro et al. bone marrow data
- DUBStepR parameters match the paper's call: optimise.features = TRUE, k = 10, num.pcs = 20, error = 0, min.cells = 5% of cells
- Peaks are log-normalized before PCA, and the same PCA + UMAP + Monocle 3 pipeline is applied uniformly across all three feature sets
- Scaling sanity: DUBStepR feature selection completes 'without sharp increases in time or memory consumption' (supplementary Supp. Fig. S4 claim)

## Limitations

- DUBStepR's own elbow-point computation and key quantitative results are deferred to supplementary notes/figures (Supp. Note 1B, Supp. Fig. S3) not described in the main text, so thresholds may need supplementary retrieval
- Existing feature selection methods (and presumably naive feature sets) 'perform inconsistently across datasets, occasionally even resulting in poorer clustering accuracy than without feature selection' — trajectory output is only as good as the upstream feature set
- Cell-type label assignment can be confounded by reference panel coverage — e.g., SingleR misclassified cells as T/NK 'due to the absence of platelet and RBC transcriptomes in the reference panel'
- Benchmark dataset for the paper's clustering evaluation is only identified by reference number 28, not named or accessioned, limiting direct reuse

## Evidence

- [other] Run Monocle 3 trajectory inference on each UMAP embedding to derive the differentiation topology.: [evidence span withheld — anchor status: not_found]
- [other] the DUBStepR-selected peak set's Monocle 3 trajectory recovers the three-branch lymphoid/myeloid/megakaryocyte-erythroid topology while the all-peaks and HVF sets fail to do so, matching Fig 7.: [evidence span withheld — anchor status: not_found]
- [abstract] DUBStepR is scalable to over a million cells, and can be straightforwardly applied to other data types such as single-cell ATAC-seq.: "DUBStepR is scalable to over a million cells, and can be straightforwardly applied to other data types such as single-cell ATAC-seq."
- [intro] typically reduced to a small number of dimensions using principal component analysis (PCA): "which are then typically reduced to a small number of dimensions using principal component analysis (PCA)5"
- [methods] DUBStepR(input.data, min.cells, species, optimise.features, k, num.pcs, error): "`DUBStepR(input.data, min.cells, species, optimise.features, k, num.pcs, error)`"
- [readme] DUBStepR is based on the intuition that cell-type-specific marker genes tend to be well correlated with each other: "DUBStepR is based on the intuition that cell-type-specific marker genes tend to be well correlated with each other, i.e. they typically have strong positive and negative correlations"
