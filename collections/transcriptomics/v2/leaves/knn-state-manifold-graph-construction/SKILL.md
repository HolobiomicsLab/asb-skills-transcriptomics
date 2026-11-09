---
name: knn-state-manifold-graph-construction
description: Use when preparing single-cell RNA-Seq data (e.g. a Seurat object) as
  an undirected kNN graph input for EVA/ELVAR community detection and DA testing,
  or when reproducing the paper's lungCD4T or Tabula Muris Senis pipelines.
when_to_use_negative:
- Input is already a cell-cell graph or ELVAR-compatible igraph object — rebuilding
  the kNN graph would discard existing neighborhood structure
- The goal is generic clustering/visualization (FindClusters/RunTSNE) without downstream
  attribute-aware DA testing
- Data is single-cell ATAC-Seq or another modality where a gene-expression PCA state-space
  is not the appropriate manifold
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: Seurat
  role: Creates the Seurat object, runs FindVariableFeatures (vst), ScaleData, RunPCA,
    and FindNeighbors to build the kNN state-manifold graph
  repo: https://github.com/aet21/ELVAR
- name: ELVAR
  role: Consumes the kNN state-manifold graph; installs via devtools::install_github
    and its tutorial data (lungCD4Tinfo) provides the worked example
  repo: https://github.com/aet21/ELVAR
- name: Matrix
  role: Handles the sparse adjacency matrix extracted from the Seurat graph for igraph
    conversion
- name: igraph
  role: Converts the adjacency matrix into an undirected graph (graph.adjacency) and
    stores cell attributes as vertex attributes
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-39017-z
    title: Cell-attribute aware community detection improves differential abundance
      testing from single-cell RNA-Seq data
  source_content_hash: sha256:a479a178d419db5aa0a0f4a963880c0118bb282127c0d0b16e42e1fe41e93246
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
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# knn-state-manifold-graph-construction

## Summary

Builds a cell-cell kNN graph (the 'state-manifold') from PCA-reduced scRNA-Seq data using Seurat's FindNeighbors, which ELVAR then consumes for cell-attribute aware community detection and differential abundance testing.

## When to use

Use when preparing single-cell RNA-Seq data (e.g. a Seurat object) as an undirected kNN graph input for EVA/ELVAR community detection and DA testing, or when reproducing the paper's lungCD4T or Tabula Muris Senis pipelines.

## When NOT to use

- Input is already a cell-cell graph or ELVAR-compatible igraph object — rebuilding the kNN graph would discard existing neighborhood structure
- The goal is generic clustering/visualization (FindClusters/RunTSNE) without downstream attribute-aware DA testing
- Data is single-cell ATAC-Seq or another modality where a gene-expression PCA state-space is not the appropriate manifold

## Inputs

- scRNA-Seq count matrix (e.g. from data(lungCD4Tinfo) or Tabula Muris Senis)
- Seurat object with cell-level metadata/phenotype attributes
- PCA embedding (first 8 PCs)

## Outputs

- Cell-cell nearest-neighbor adjacency matrix (state-manifold)
- Undirected igraph graph with cell attributes as vertex_attr (e.g. 'age' or condition)
- Preprocessed Seurat object for downstream ProcessEVA/DoDA

## How to apply

After creating a Seurat object and assigning cell attributes (e.g. age or condition) in meta.data, run FindVariableFeatures (selection.method='vst'), ScaleData, and RunPCA, then build the graph with FindNeighbors(dims=1:8, k.param=20). Convert the adjacency matrix to an igraph undirected graph via graph.adjacency(adj.m, mode='undirected') and attach the biological condition as a vertex attribute (e.g. vertex_attr(gr.o, name='age') <- seu.o@meta.data$age). The k.param=20 neighborhood and dims=1:8 PCA settings are the paper's defaults; the vertex attribute is essential because ELVAR's EVA clustering weights edges by attribute similarity (alpha balances state-space vs attribute similarity). Validation: the resulting graph should reproduce Louvain clustering consistent with FindClusters and support the tutorial's naive-to-mature Cd4+ T-cell DA result.

## Related tools

- **Seurat** (Creates the Seurat object, runs FindVariableFeatures (vst), ScaleData, RunPCA, and FindNeighbors to build the kNN state-manifold graph) — https://github.com/aet21/ELVAR
- **ELVAR** (Consumes the kNN state-manifold graph; installs via devtools::install_github and its tutorial data (lungCD4Tinfo) provides the worked example) — https://github.com/aet21/ELVAR
- **Matrix** (Handles the sparse adjacency matrix extracted from the Seurat graph for igraph conversion)
- **igraph** (Converts the adjacency matrix into an undirected graph (graph.adjacency) and stores cell attributes as vertex attributes)

## Examples

```
seu.o <- FindVariableFeatures(seu.o, selection.method="vst"); seu.o <- ScaleData(seu.o); seu.o <- RunPCA(seu.o); seu.o <- FindNeighbors(seu.o, dims=1:8, k.param=20); gr.o <- graph.adjacency(adj.m, mode="undirected"); vertex_attr(gr.o, name="age") <- seu.o@meta.data$age;
```

## Evaluation signals

- Graph built with FindNeighbors dims=1:8 and k.param=20 matches the paper's stated parameters
- Adjacency matrix converts to an undirected igraph with number of vertices equal to the cell count
- Vertex attribute (e.g. 'age' or condition) is populated for every vertex from seu.o@meta.data with no missing values
- Downstream Eva_partitions on this graph at resolution=1, threshold=1e-7 reproduces clustering consistent with Seurat FindClusters at resolution 0.1
- No-correction run on lungCD4T data reproduces the tutorial's highly significant naive-to-mature Cd4+ T-cell DA result (Wald and LRT statistics from DoDA)

## Limitations

- Graph quality depends on PCA dimensionality (dims=1:8) and k.param=20 choices; the paper does not explore sensitivity to these parameters
- ELVAR is sensitive to false positives in the clustering attribute (sample/phenotype label), which corrupts the vertex attributes attached to the graph
- Louvain-based clustering on this graph is non-deterministic; ~100 EVA repeats per alpha (0.1–0.9) are recommended for robustness, adding computational cost
- Seurat/igraph versions and non-deterministic aspects of some upstream steps may affect exact reproducibility

## Evidence

- [methods] Build cell-cell nearest neighbor graph (state-manifold): "seu.o <- FindNeighbors(seu.o, dims = 1:8, k.param=20,verbose=FALSE);"
- [methods] Extract state-manifold as graph with cell attribute: "gr.o <- graph.adjacency(adj.m,mode="undirected"); vertexN.v <- names(V(gr.o)); vertex_attr(gr.o,name="age") <- seu.o@meta.data$age;"
- [methods] Create Seurat object and preprocess: "seu.o <- FindVariableFeatures(seu.o, selection.method = "vst",verbose=FALSE);"
- [readme] README installation and scope: "ELVAR is an R-package for differential abundance (DA) testing of cell-types in single-cell RNA-Seq data. It implements an Extended Louvain clustering Algorithm (EVA) that takes cell attribute"
- [intro] Clustering rationale from intro: "This novel concept allows clustering of cells, not only by similarity in the high-dimensional state-space, but also by how similar their attributes (e.g. age, disease stage) are"
