---
name: single-cell-cluster-annotation-propagation
description: Use when a clustered scRNA-seq count matrix (e.g., a Seurat object like
  demo_sc_data) lacks cell-type labels and cell-cell communication analysis with scCrossTalk
  requires a per-cell sc_celltype character vector.
when_to_use_negative:
- The scRNA-seq data are already annotated and a cell-type vector exists to supply
  directly as sc_celltype to create_scCrossTalk
- The input is bulk transcriptomic data or an unclustered count matrix with no cluster
  assignments to propagate labels onto
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDeepSort
  role: Cell-type classifier run in pre-computed-reference mode to predict cell-type
    probabilities for each cell in a cluster
- name: Seurat
  role: Provides the clustered single-cell count matrix / object that the annotation
    is applied to; scCrossTalk is compatible with Seurat 5
- name: scCrossTalk
  role: Downstream consumer — the propagated cell-type labels become the sc_celltype
    argument of create_scCrossTalk
  repo: https://github.com/multitalk/scCrossTalk
- name: R
  role: Implementation environment for the annotation workflow and writing cell_annotations.csv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.eng.2023.12.004
    title: A Single-Cell Landscape of Human Liver Transplantation Reveals a Pathogenic
      Immune Niche Associated with Early Allograft Dysfunction
  source_content_hash: sha256:e2dbe7f66f68e0bbebacc7fc0aa75784044a827f9515156acd27050295e77fea
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/multitalk/scCrossTalk
  repo_url: https://github.com/multitalk/scCrossTalk
  tools:
  - R
  - Seurat
  - scCrossTalk
  - scDeepSort
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# single-cell-cluster-annotation-propagation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns a cell type to each cluster of a single-cell RNA-seq dataset using scDeepSort's pre-computed-reference classifier, propagates the cluster-level label to every member cell, and emits a cell_annotations.csv whose cell-type column feeds directly into scCrossTalk's create_scCrossTalk as the sc_celltype argument.

## When to use

Use when a clustered scRNA-seq count matrix (e.g., a Seurat object like demo_sc_data) lacks cell-type labels and cell-cell communication analysis with scCrossTalk requires a per-cell sc_celltype character vector.

## When NOT to use

- The scRNA-seq data are already annotated and a cell-type vector exists to supply directly as sc_celltype to create_scCrossTalk
- The input is bulk transcriptomic data or an unclustered count matrix with no cluster assignments to propagate labels onto

## Inputs

- Clustered single-cell count matrix with cluster assignments (e.g., Seurat object or demo_sc_data matrix)
- Species designation (e.g., human)
- scDeepSort pre-computed reference-annotated training set

## Outputs

- cell_annotations.csv mapping each cell to a cell type
- Character vector of cell types (sc_celltype) consumable by create_scCrossTalk

## How to apply

Load the clustered single-cell count matrix with cluster assignments (e.g., the demo_sc_data matrix shipped with scCrossTalk) together with the species designation, such as human for the liver transplant samples. Run scDeepSort in pre-computed-reference mode on the clustered object to predict cell-type probabilities for each cell from the reference-annotated training set, then aggregate the per-cell predictions to one cell type per cluster and propagate that cluster-level label back to all member cells as a character vector. Write the resulting cell-to-celltype mapping as cell_annotations.csv, formatted so the cell-type column can be passed directly as the sc_celltype argument of create_scCrossTalk(sc_data, sc_celltype, species). Validate that cell_annotations.csv exists, its row count equals the number of cells in the input object, and every cell carries a non-empty cell-type label; for this paper, labels distinguish lineages such as hepatocytes, endothelial cells, MAIT cells, GZMB+ GZMK+ NK cells, and S100A12+ neutrophils.

## Related tools

- **scDeepSort** (Cell-type classifier run in pre-computed-reference mode to predict cell-type probabilities for each cell in a cluster)
- **Seurat** (Provides the clustered single-cell count matrix / object that the annotation is applied to; scCrossTalk is compatible with Seurat 5)
- **scCrossTalk** (Downstream consumer — the propagated cell-type labels become the sc_celltype argument of create_scCrossTalk) — https://github.com/multitalk/scCrossTalk
- **R** (Implementation environment for the annotation workflow and writing cell_annotations.csv)

## Examples

```
annotations <- read.csv('cell_annotations.csv'); obj <- create_scCrossTalk(sc_data, annotations$celltype, 'human'); obj <- find_lrpairs(object, lrpairs = lrpairs)
```

## Evaluation signals

- cell_annotations.csv exists after the run
- Row count of cell_annotations.csv equals the number of cells in the input object
- Every cell has a non-empty cell-type label
- create_scCrossTalk accepts the cell-type column as sc_celltype and constructs an object of class scCrossTalk without errors
- Cell-type labels are biologically coherent for the tissue (e.g., hepatocytes, endothelial cells, MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils in human liver)

## Limitations

- Annotation quality depends on the pre-computed scDeepSort reference; a mismatched species or tissue reference can mislabel clusters
- All cells in a cluster inherit one label, so heterogeneous clusters will be flattened to a single cell type

## Evidence

- [other] Cluster cell types were annotated by applying scDeepSort, a classifier that uses pre-computed reference data: [evidence span withheld — anchor status: not_found]
