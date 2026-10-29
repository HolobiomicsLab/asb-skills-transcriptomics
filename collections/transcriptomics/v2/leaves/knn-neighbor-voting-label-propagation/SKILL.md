---
name: knn-neighbor-voting-label-propagation
description: Use when annotating individual cells directly from marker GEPs (no clustering)
  and you need to smooth/consolidate per-cell GEP assignments using a precomputed
  KNN neighbor index.
when_to_use_negative:
- The target dataset contains cell types absent from the reference GEP set (e.g.,
  TMS cells annotated with MCA GEPs can only be labeled 'unknown')
- High-resolution neuronal subtypes are needed but the reference (MCA) lacks them
  — MCA GEPs would compromise Mouse Brain neuron annotation precision
- You only need GEP-averaged expression for dimension reduction, not per-cell categorical
  annotation
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: CellGEP
  role: R implementation containing the cellgep function that performs highest-GEP
    assignment and 20-neighbor voting with dynamic.cutoff
  repo: https://github.com/MaShisongLab/cellgep
- name: Seurat
  role: Builds the Seurat object, runs PCA/UMAP, and computes the neighbor index via
    FindNeighbors(return.neighbor = T) consumed by cellgep
  repo: https://satijalab.org/seurat/
- name: edgeR
  role: R dependency loaded when computing GEP-averaged expression with average_by_gep
    prior to annotation
  repo: https://bioconductor.org/packages/release/bioc/html/edgeR.html
- name: graph
  role: R dependency required before sourcing cellgep.R and calling cellgep
  repo: https://www.bioconductor.org/packages/release/bioc/html/graph.html
- name: RBGL
  role: R dependency required before sourcing cellgep.R and calling cellgep
  repo: https://www.bioconductor.org/packages/release/bioc/html/RBGL.html
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:9a57e806982d41c3ef9af67b988b8d74807f339dc59c82ed63c5d35a6aeaf8fe
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/MaShisongLab/CellGEP
  repo_url: https://github.com/MaShisongLab/cellgep
  tools:
  - CellGEP
  - RBGL
  - Seurat
  - SingleCellGGM
  - edgeR
  - graph
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# knn-neighbor-voting-label-propagation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A step inside CellGEP in which each cell's marker-GEP annotation is consolidated by k-nearest-neighbor voting: after the highest-expressed GEP is assigned per cell, the labels of the cell's stored Seurat neighbors are used to propagate and stabilize cell-type labels without clustering.

## When to use

Use when annotating individual cells directly from marker GEPs (no clustering) and you need to smooth/consolidate per-cell GEP assignments using a precomputed KNN neighbor index.

## When NOT to use

- The target dataset contains cell types absent from the reference GEP set (e.g., TMS cells annotated with MCA GEPs can only be labeled 'unknown')
- High-resolution neuronal subtypes are needed but the reference (MCA) lacks them — MCA GEPs would compromise Mouse Brain neuron annotation precision
- You only need GEP-averaged expression for dimension reduction, not per-cell categorical annotation

## Inputs

- GEP-averaged expression matrix (genes x cells averaged per GEP) from average_by_gep
- KNN neighbor index matrix (nn.idx from Seurat FindNeighbors, 20 neighbors per cell)
- Marker GEP table, e.g. 'data/mca.marker.geps.txt' (Ensembl ID) or 'mca.marker.symbl.geps.txt' (gene symbol)
- Integer array geps_to_use specifying which GEPs to use (e.g., 1:gep_num)

## Outputs

- Per-cell annotation table with columns GEP, GEP_id, is_original, Expression, GEP_mean, GEP_sd, GEPs_expressed, GEP_Ori, and neighbor vote columns N1..N20
- Assignment purity/accuracy metrics per GEP (e.g., M1 oligodendrocyte purity 99.6%, M3 T-cell purity 99.2%) and ARI/NMI against cluster labels
- meta.data column 'marker_gep' for Seurat DimPlot visualization

## How to apply

First build a Seurat object for the dataset, run PCA/UMAP, and compute the neighbor index with FindNeighbors(return.neighbor = T), extracting sc_object@neighbors$RNA.nn@nn.idx (a 20-neighbor index, as shown in the README). Compute GEP-averaged expression with average_by_gep over the marker GEPs (e.g., data/mca.marker.geps.txt), then call cellgep(geps_expression$expression, sc_neighbor, 1:gep_num, dynamic.cutoff=T): a GEP is called expressed when its averaged value exceeds a dynamically determined fraction of its maximum, the highest-expressed GEP is assigned, and KNN voting over the 20 stored neighbors propagates/consolidates each cell's label. The output annotation table stores the winning GEP (GEP, GEP_id, Expression) plus per-neighbor vote columns (N1, N2, ..., N20), where 'na' indicates a neighbor that did not vote for the same GEP. Rationale: neighbor voting suppresses spurious single-cell GEP calls by requiring consistency with transcriptionally similar cells, enabling cluster-free label transfer as validated on MCA (e.g., M1 cells 99.6% oligodendrocytes, M3 cells 99.2% T cells).

## Related tools

- **CellGEP** (R implementation containing the cellgep function that performs highest-GEP assignment and 20-neighbor voting with dynamic.cutoff) — https://github.com/MaShisongLab/cellgep
- **Seurat** (Builds the Seurat object, runs PCA/UMAP, and computes the neighbor index via FindNeighbors(return.neighbor = T) consumed by cellgep) — https://satijalab.org/seurat/
- **edgeR** (R dependency loaded when computing GEP-averaged expression with average_by_gep prior to annotation) — https://bioconductor.org/packages/release/bioc/html/edgeR.html
- **graph** (R dependency required before sourcing cellgep.R and calling cellgep) — https://www.bioconductor.org/packages/release/bioc/html/graph.html
- **RBGL** (R dependency required before sourcing cellgep.R and calling cellgep) — https://www.bioconductor.org/packages/release/bioc/html/RBGL.html

## Examples

```
require(graph); require(RBGL); source('cellgep.R'); gep_num = dim(geps_expression$expression)[1]; anno = cellgep(geps_expression$expression, sc_neighbor, 1:gep_num, dynamic.cutoff=T)
```

## Evaluation signals

- Annotation purity for known GEP-cell-type pairs: M1-assigned cells should be ~99.6% oligodendrocytes (1,120/1,125) and M3-assigned cells ~99.2% T cells (1,886 cells)
- Neighbor vote columns N1..N20 should be populated with GEP ids or 'na' (not blanks), and the majority of neighbors should agree with the cell's own GEP call
- Expression values in the annotation table should satisfy the GEP's dynamic cutoff, i.e. be at or above the fraction of the GEP's maximal average expression stored as GEP_mean/GEP_sd context
- ARI and NMI computed against independent cluster annotations (scDHA, scMGCA) should be non-trivially positive; results written to cellgep_annotation_metrics.csv
- Neighbor index must come from FindNeighbors(return.neighbor = T) on the same PCA space used for UMAP; rows of the neighbor matrix should match the expression matrix cell order

## Limitations

- Label transfer is constrained by the cell-type coverage of the source dataset — target cells whose types are absent become 'unknown'
- Annotating datasets with finer cell subtypes than the reference (e.g., Mouse Brain neurons with MCA GEPs) compromises annotation precision
- SingleCellGGM/CellGEP relies exclusively on transcriptome data and does not integrate other single-cell modalities (ATAC-seq, proteomics)

## Evidence
