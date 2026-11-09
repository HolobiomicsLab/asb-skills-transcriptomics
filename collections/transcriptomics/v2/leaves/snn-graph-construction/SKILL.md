---
name: snn-graph-construction
description: Use when a scRNA-seq Seurat object must be prepared with an SNN graph
  and UMAP embedding as the spatial substrate for Census's cluster-and-contour label
  stabilization before hierarchical annotation.
when_to_use_negative:
- The input object has not yet been normalized, scaled, and PCA-reduced — RunUMAP/FindNeighbors
  on unprocessed counts will not yield a meaningful SNN graph
- The environment has Seurat v5 installed and the workflow requires Census, since
  Census explicitly requires Seurat version 4 loaded first
- The task is plain clustering/visualization with no downstream Census label-stabilization
  — the contour/propagation loop is not needed
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: Seurat 4
  role: Constructs the shared nearest-neighbor graph and UMAP embedding (FindNeighbors/RunUMAP
    on dims=1:50) that provide the cluster structure Census uses for label propagation;
    must be loaded before the Census library
  repo: https://github.com/sjdlabgroup/Census
- name: Census
  role: Consumes the Seurat object's UMAP reduction and SNN clusters in its custom
    label-stabilizing algorithm (cluster propagation plus prediction contours)
  repo: https://github.com/sjdlabgroup/Census
- name: MASS
  role: Computes prediction contours for each candidate label in the UMAP space built
    from the SNN graph
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad714
    title: Hierarchical and automated cell-type annotation and inference of cancer
      cell of origin with Census
  source_content_hash: sha256:998a507b146b4b26ec4c08309e787b83af587f7c01a3e2be0e8fd9b06697aae1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sjdlabgroup/Census
  tools:
  - Census
  - MASS
  - Seurat 4
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# snn-graph-construction

## Summary

Constructs a shared nearest-neighbor (SNN) graph and UMAP embedding from a processed Seurat object so that Census's label-stabilizing algorithm can propagate confident binary predictions within UMAP SNN clusters and compute MASS-based prediction contours.

## When to use

Use when a scRNA-seq Seurat object must be prepared with an SNN graph and UMAP embedding as the spatial substrate for Census's cluster-and-contour label stabilization before hierarchical annotation.

## When NOT to use

- The input object has not yet been normalized, scaled, and PCA-reduced — RunUMAP/FindNeighbors on unprocessed counts will not yield a meaningful SNN graph
- The environment has Seurat v5 installed and the workflow requires Census, since Census explicitly requires Seurat version 4 loaded first
- The task is plain clustering/visualization with no downstream Census label-stabilization — the contour/propagation loop is not needed

## Inputs

- Seurat object (Seurat v4 compatible) with single-cell RNA-seq counts
- Normalized, scaled expression matrix with variable features identified
- PCA reduction computed on the first 50 dimensions

## Outputs

- SNN neighbor graph stored in the Seurat object (used for UMAP SNN cluster detection)
- UMAP embedding (umap1, umap2 coordinates) supplied to census_main and used for MASS prediction contours

## How to apply

First process the raw data through the standard Seurat pipeline: CreateSeuratObject, NormalizeData, ScaleData, FindVariableFeatures, RunPCA, then RunUMAP(dims=1:50) — Census requires the input Seurat object to already contain a UMAP dimensionality reduction. Critically, load Seurat version 4 (library(Seurat)) before loading the Census library, since Census requires Seurat v4 to run. Build the shared nearest-neighbor graph on the PCA embedding (Seurat's FindNeighbors on dims=1:50) and the corresponding UMAP SNN clusters; these clusters are the units across which Census propagates the most common (average) label. The UMAP coordinates (umap1, umap2) also serve as the 2-D space in which MASS-based prediction contours are computed per candidate label. Rationale: the stabilizing algorithm uses 'clusters and prediction contours in UMAP space to propagate higher confidence binary prediction outcomes', so graph quality directly controls how stable the propagated labels are.

## Related tools

- **Seurat 4** (Constructs the shared nearest-neighbor graph and UMAP embedding (FindNeighbors/RunUMAP on dims=1:50) that provide the cluster structure Census uses for label propagation; must be loaded before the Census library) — https://github.com/sjdlabgroup/Census
- **Census** (Consumes the Seurat object's UMAP reduction and SNN clusters in its custom label-stabilizing algorithm (cluster propagation plus prediction contours)) — https://github.com/sjdlabgroup/Census
- **MASS** (Computes prediction contours for each candidate label in the UMAP space built from the SNN graph)

## Examples

```
obj = obj %>% CreateSeuratObject() %>% NormalizeData() %>% ScaleData() %>% FindVariableFeatures() %>% RunPCA() %>% RunUMAP(dims=1:50); res = census_main(obj, organ = 'Lung')
```

## Evaluation signals

- Seurat v4 is loaded before library(Census) — a mismatch here causes Census to fail to run
- The Seurat object passed to census_main contains a UMAP dimensionality reduction (the README states the input 'is a Seurat object that contains a UMAP dimensionality reduction')
- RunUMAP was called with dims=1:50 on the PCA reduction, matching the documented pipeline
- The propagated/stabilized labels vary smoothly within UMAP SNN clusters, and the propagate-contour-dispute loop converges with no further label changes between iterations
- Final res$pred contains non-NA celltype assignments mapped to umap1/umap2 coordinates

## Limitations

- Census requires Seurat version 4; newer Seurat versions must not be loaded before the Census library
- The graph/UMAP is built per dataset and the cancer model only uses the first organ specified in the organ parameter, so multi-organ inputs get contours stabilized in a single embedding
- Available organ choices are limited to the 24 Tabula Sapiens organs (e.g., Lung, Pancreas, Kidney); tissues outside this set are not covered by the pretrained model

## Evidence

- [readme] Seurat pipeline for UMAP substrate: "obj %>% CreateSeuratObject() %>% NormalizeData() %>% ScaleData() %>%
  FindVariableFeatures() %>% RunPCA() %>% RunUMAP(dims=1:50)"
- [readme] Seurat v4 requirement: "Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R."
- [readme] Input must contain UMAP reduction: "The input data is a Seurat object that contains a UMAP dimensionality reduction."
- [intro] Purpose of clusters and contours: "This involves using clusters and prediction contours in UMAP space to propagate higher confidence binary prediction outcomes."
- [readme] Label-stabilizing algorithm scope: "New datasets are annotated using the pretrained models followed by a custom-developed label-stabilizing algorithm (Fig. 1b)."
