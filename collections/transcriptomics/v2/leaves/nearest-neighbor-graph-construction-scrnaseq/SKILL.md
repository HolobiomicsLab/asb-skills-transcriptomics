---
name: nearest-neighbor-graph-construction-scrnaseq
description: Use when an annotated scRNA-Seq dataset processed through FindVariableFeatures/ScaleData/RunPCA
  needs a k-nearest-neighbor cell graph with vertex attributes (e.g. age, disease
  severity) before community-based differential abundance testing.
when_to_use_negative:
- When the input is already a precomputed graph or adjacency matrix rather than a
  Seurat object with PCA embeddings
- When no cell attribute (condition/phenotype label) is available to attach as vertex
  attributes, since EVA/ELVAR's sensitivity gain depends on attribute-aware clustering
- When the analysis goal is differential expression rather than differential abundance
  testing of cell types
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: Provides FindNeighbors with k.param=20 to build the cell-cell kNN graph from
    PCA space after vst variable-feature selection, scaling and RunPCA
- name: igraph
  role: Converts the Seurat adjacency matrix into an undirected graph and carries
    vertex attributes (e.g. age) needed by Eva_partitions
- name: ELVAR
  role: Consumes the constructed state-manifold graph as input to Eva_partitions for
    attribute-aware community detection
  repo: https://github.com/aet21/ELVAR
- name: Matrix
  role: Handles the sparse adjacency matrix extracted from the Seurat object when
    building the graph
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-39017-z
    title: Cell-attribute aware community detection improves differential abundance
      testing from single-cell RNA-Seq data
  source_content_hash: sha256:1fa83ea591a9e0deb34394e77e7df2a4e01061ae6dba61f21b761ca7643e2736
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/ELVAR
  tools:
  - ELVAR
  - EVA
  - MASS
  - Matrix
  - R
  - Seurat
  - dplyr
  - igraph
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# nearest-neighbor-graph-construction-scrnaseq

## Summary

Builds the cell-cell nearest-neighbor state-manifold graph from a Seurat-processed scRNA-Seq object (post-PCA) that serves as the substrate for ELVAR's attribute-aware EVA community detection and downstream DA testing.

## When to use

Use when an annotated scRNA-Seq dataset processed through FindVariableFeatures/ScaleData/RunPCA needs a k-nearest-neighbor cell graph with vertex attributes (e.g. age, disease severity) before community-based differential abundance testing.

## When NOT to use

- When the input is already a precomputed graph or adjacency matrix rather than a Seurat object with PCA embeddings
- When no cell attribute (condition/phenotype label) is available to attach as vertex attributes, since EVA/ELVAR's sensitivity gain depends on attribute-aware clustering
- When the analysis goal is differential expression rather than differential abundance testing of cell types

## Inputs

- Seurat object with annotated cell types and a biological condition/attribute column in meta.data (e.g. age, Covid-19 severity, patient sample)
- PCA-reduced expression matrix (first several principal components, e.g. dims 1:8)
- Cell-level metadata from benchmark scRNA-Seq datasets (e.g. Tabula Muris Senis lung CD4+ T cells, Chua nasopharyngeal Covid-19 swabs, colon cancer)

## Outputs

- Undirected igraph state-manifold with cells as vertices and kNN edges (k.param=20)
- Vertex attributes attached to the graph (e.g. age, severity) for EVA clustering
- Connectivity check confirming a connected graph

## How to apply

After the standard Seurat preprocessing chain (FindVariableFeatures with selection.method='vst', ScaleData, RunPCA), call FindNeighbors on the Seurat object with a reduced PCA dimensionality (e.g. dims = 1:8) and k.param = 20 to obtain the k-nearest-neighbor adjacency matrix defining the cell state-manifold. Convert the adjacency matrix into an undirected igraph via graph.adjacency(mode='undirected') and attach the biological attribute of interest (e.g. age or Covid-19 severity) as a vertex attribute, since EVA clustering requires both graph structure and cell attributes. Verify that the graph is connected before running Eva_partitions, as the ELVAR pipeline assumes a connected state-manifold. The graph quality (choice of dims and k) directly conditions the sensitivity of the subsequent attribute-aware DA testing.

## Related tools

- **Seurat** (Provides FindNeighbors with k.param=20 to build the cell-cell kNN graph from PCA space after vst variable-feature selection, scaling and RunPCA)
- **igraph** (Converts the Seurat adjacency matrix into an undirected graph and carries vertex attributes (e.g. age) needed by Eva_partitions)
- **ELVAR** (Consumes the constructed state-manifold graph as input to Eva_partitions for attribute-aware community detection) — https://github.com/aet21/ELVAR
- **Matrix** (Handles the sparse adjacency matrix extracted from the Seurat object when building the graph)

## Examples

```
seu.o <- FindNeighbors(seu.o, dims = 1:8, k.param=20, verbose=FALSE); gr.o <- graph.adjacency(seu.o@graphs$RNA_nn@adjacency, mode="undirected"); vertex_attr(gr.o, name="age") <- seu.o@meta.data$age
```

## Evaluation signals

- The graph is connected before Eva_partitions is run
- FindNeighbors parameters match the paper's settings: dims = 1:8 and k.param = 20
- Every cell in the Seurat object appears as exactly one vertex and the attribute (e.g. age) is set on all vertices via vertex_attr(gr.o, name='age')
- Downstream Eva_partitions/SelOptAlpha runs return non-empty communities with modularity values, indicating a usable manifold
- Re-running the graph construction on the same Seurat object yields a deterministic adjacency structure (kNN is deterministic given PCA embeddings)

## Limitations

- Choice of the number of PCA dims (e.g. 1:8) and k (20) conditions graph topology; suboptimal settings propagate to community detection and DA sensitivity
- ELVAR's DA performance depends on the quality of the attribute attached to vertices — false positives in the clustering attribute (sample/phenotype label) degrade community quality
- Batch correction prior to graph construction rarely improved ELVAR sensitivity, so manual integration is usually unnecessary but the graph may still reflect batch structure

## Evidence

- [methods] FindNeighbors with kNN params: "seu.o <- FindNeighbors(seu.o, dims = 1:8, k.param=20,verbose=FALSE);"
- [methods] graph with vertex attribute: [evidence span withheld — anchor status: not_found]
- [methods] PCA preprocessing chain: "seu.o <- FindVariableFeatures(seu.o, selection.method = "vst",verbose=FALSE); seu.o <- ScaleData(seu.o,verbose=FALSE); seu.o <- RunPCA(seu.o,verbose=FALSE);"
- [intro] EVA uses attribute-aware clustering on the manifold: "This novel concept allows clustering of cells, not only by similarity in the high-dimensional state-space, but also by how similar their attributes (e.g. age, disease stage) are"
- [readme] README: ELVAR uses cell-cell nearest neighbour graph: "It implements an Extended Louvain clustering Algorithm (EVA) that takes cell attribute information into acccount when inferring cellular communities from the cell-cell nearest neighbour graph."
