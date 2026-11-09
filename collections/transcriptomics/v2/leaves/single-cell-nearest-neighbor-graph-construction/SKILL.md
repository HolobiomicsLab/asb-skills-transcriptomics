---
name: single-cell-nearest-neighbor-graph-construction
description: 'Use when preparing single-cell RNA-Seq count data for ELVAR differential
  abundance testing: a Seurat-processed count matrix with PCA is available and an
  igraph state-manifold with condition attributes is needed for cell-attribute aware
  community detection.'
when_to_use_negative:
- Input is already a clustered community structure or an existing graph, rather than
  a raw count matrix needing PCA and neighbor construction
- The analysis goal is a Louvain baseline without cell-attribute information, where
  attribute-aware neighbor graphs are unnecessary (though the same graph object is
  still the substrate)
- Cell attributes are known to contain false positives in the sample/phenotype label,
  since ELVAR was shown to be more sensitive to FPs in the clustering attribute than
  in cell-type annotation
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: FindVariableFeatures (vst), ScaleData, RunPCA, and FindNeighbors (dims=1:8,
    k.param=20) to compute the kNN adjacency
- name: ELVAR
  role: R package whose Eva_partitions consumes the attributed igraph graph; provides
    data(lungCD4Tinfo) for the tutorial workflow
  repo: https://github.com/aet21/ELVAR
- name: igraph
  role: graph.adjacency conversion to undirected graph, vertex_attr attachment of
    condition labels, connectivity check
- name: Matrix
  role: Sparse adjacency-matrix handling when extracting the kNN graph from the Seurat
    object
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-39017-z
    title: Cell-attribute aware community detection improves differential abundance
      testing from single-cell RNA-Seq data
  source_content_hash: sha256:7aa2b109e9fc20109c016d79c410b8220fd585ced7eb8ef167dc1373596a24d0
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
  - ggplot2
  - igraph
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# single-cell-nearest-neighbor-graph-construction

## Summary

Constructs a cell-cell nearest-neighbor state-manifold graph from scRNA-Seq PCA embeddings (Seurat FindNeighbors, dims 1:8, k.param=20), converts it to an undirected igraph object with self-loops removed, and attaches the biological condition (e.g. age or disease phenotype) as a vertex attribute for downstream ELVAR/EVA clustering.

## When to use

Use when preparing single-cell RNA-Seq count data for ELVAR differential abundance testing: a Seurat-processed count matrix with PCA is available and an igraph state-manifold with condition attributes is needed for cell-attribute aware community detection.

## When NOT to use

- Input is already a clustered community structure or an existing graph, rather than a raw count matrix needing PCA and neighbor construction
- The analysis goal is a Louvain baseline without cell-attribute information, where attribute-aware neighbor graphs are unnecessary (though the same graph object is still the substrate)
- Cell attributes are known to contain false positives in the sample/phenotype label, since ELVAR was shown to be more sensitive to FPs in the clustering attribute than in cell-type annotation

## Inputs

- scRNA-Seq / snRNA-Seq raw count matrix (e.g. 537-cell lung Cd4+ T-cell matrix from lungCD4Tinfo in ELVAR)
- Seurat object with PCA embedding (RunPCA on vst variable features)
- Per-cell metadata: condition/attribute labels (age group, disease stage, phenotype) and replicate ID (e.g. mouse)

## Outputs

- Undirected cell-cell nearest-neighbor igraph graph (self-loops removed, verified connected)
- Vertex-attributed graph with condition attribute (e.g. 'age') attached for Eva_partitions

## How to apply

First run the standard Seurat preprocessing on the raw count matrix (CreateSeuratObject, FindVariableFeatures with selection.method='vst', ScaleData, RunPCA). Then call FindNeighbors on the first 8 principal components with k.param=20 to build the k-nearest-neighbor state-manifold. Extract the adjacency as an undirected igraph graph (graph.adjacency, mode='undirected', zeroing the diagonal to remove self-loops) and attach the condition variable — e.g. age coded 0/1/2/3/5 for 1m/3m/18m/21m/30m months, or a smell-loss phenotype label — via vertex_attr. Verify the graph is connected before passing it to Eva_partitions, since EVA community detection requires a single connected state-manifold; cell attributes (age, disease stage) guide the purity-vs-modularity trade-off during subsequent attribute-aware clustering.

## Related tools

- **Seurat** (FindVariableFeatures (vst), ScaleData, RunPCA, and FindNeighbors (dims=1:8, k.param=20) to compute the kNN adjacency)
- **ELVAR** (R package whose Eva_partitions consumes the attributed igraph graph; provides data(lungCD4Tinfo) for the tutorial workflow) — https://github.com/aet21/ELVAR
- **igraph** (graph.adjacency conversion to undirected graph, vertex_attr attachment of condition labels, connectivity check)
- **Matrix** (Sparse adjacency-matrix handling when extracting the kNN graph from the Seurat object)

## Examples

```
gr.o <- graph.adjacency(adj.m, mode="undirected"); vertex_attr(gr.o, name="age") <- seu.o@meta.data$age; eva.o <- Eva_partitions(gr.o, resolution=1, threshold=0.0000001, alpha=0.8, Vattr.name="age");
```

## Evaluation signals

- Graph has no self-loops (diagonal of adjacency zeroed) and is reported as connected before Eva_partitions
- Edge structure derives from FindNeighbors with dims=1:8 and k.param=20, matching the paper's state-manifold parameters
- Vertex attribute (e.g. age) length matches number of vertices and values align with cell names in V(gr.o)
- Subsequent Eva_partitions run's purity and modularity fall within the purmodInfo boxplot distribution at the chosen alpha (e.g. alpha=0.8), confirming a typical graph/partition
- Louvain analog (LVnonseq) on the same graph yields weaker DA significance than EVA (e.g. Fig. 3g comparison), indicating attribute-aware clustering is functioning

## Limitations

- ELVAR's attribute-aware community detection incurs increased computational complexity and runtimes compared to Milo or the Louvain-analog, though the authors deemed this not practically prohibitive
- Finding the optimal alpha parameter may require 100 runs for each of nine alpha values (0.1–0.9), i.e. ~900 runs, potentially parallelized over a server
- False positives in the clustering attribute (sample/phenotype label) can degrade the quality of the inferred communities

## Evidence

- [methods] Build cell-cell nearest neighbor graph (state-manifold): "seu.o <- FindNeighbors(seu.o, dims = 1:8, k.param=20,verbose=FALSE);"
- [methods] Extract state-manifold as graph with cell attribute: "gr.o <- graph.adjacency(adj.m,mode="undirected"); vertexN.v <- names(V(gr.o)); vertex_attr(gr.o,name="age") <- seu.o@meta.data$age;"
- [methods] PCA preprocessing precedes neighbor construction: "seu.o <- FindVariableFeatures(seu.o, selection.method = "vst",verbose=FALSE); seu.o <- ScaleData(seu.o,verbose=FALSE); seu.o <- RunPCA(seu.o,verbose=FALSE);"
- [intro] ELVAR clusters cells by state-space and attribute similarity: "This novel concept allows clustering of cells, not only by similarity in the high-dimensional state-space, but also by how similar their attributes (e.g. age, disease stage) are"
- [discussion] Alpha selection requires many runs: "to identify an optimal a parameter value may require a total of 100 runs for each of nine a parameter values, so a total of 900 runs, which on a 100-node server would require 9 instances"
