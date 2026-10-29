---
name: dimensionality-reduction-visualization
description: Use when a processed scRNA-seq object (e.g. SingCellaR Human_HSPC) has
  clustering and reduced-dimension results ready and you need publication-style 2D
  plots colored by cluster, gene expression, cell type annotation, or pseudotime.
when_to_use_negative:
- The SingCellaR object has not yet been normalized (normalize_UMIs) or clustered
  (identifyClusters) — no embedding or labels exist to visualize
- The goal is batch integration itself (runScanorama) rather than visualization of
  an existing embedding
- You need automatic interoperability with Seurat objects — SingCellaR 'still lacks
  automatic object transformation' for other packages, so embedding transfer must
  be done manually
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SingCellaR
  role: Stores the reduced-dimension space and cluster assignments; provides get_umap.result,
    get_clusters, and runFA2_ForceDirectedGraph for 2D layouts
  repo: github.com/supatt-lab/SingCellaR
- name: Monocle3
  role: Consumes the transferred SingCellaR UMAP embedding and cluster labels in a
    cell_data_set to draw learn_graph trajectories and pseudotime-colored UMAP plots
- name: fa2
  role: Python force-atlas2 layout backend required by SingCellaR's runFA2_ForceDirectedGraph
- name: R
  role: Environment in which SingCellaR, its ggplot-based plotting, and the Python-backed
    FA2 module are invoked
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:95e31d9cb6dcb4c40559ca08e94746bdf53c941ece6c9946df1b1c6492d70dc1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Monocle3
  - R
  - SingCellaR
  - fa2
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# dimensionality-reduction-visualization

## Summary

Generate 2D embeddings and plots (UMAP, FA2 force-directed graphs, and Monocle3 trajectory UMAPs) from a processed SingCellaR single-cell RNA-seq object so cell populations, cluster structure, and pseudotime trajectories on human hematopoietic stem and progenitor cells can be inspected visually.

## When to use

Use when a processed scRNA-seq object (e.g. SingCellaR Human_HSPC) has clustering and reduced-dimension results ready and you need publication-style 2D plots colored by cluster, gene expression, cell type annotation, or pseudotime.

## When NOT to use

- The SingCellaR object has not yet been normalized (normalize_UMIs) or clustered (identifyClusters) — no embedding or labels exist to visualize
- The goal is batch integration itself (runScanorama) rather than visualization of an existing embedding
- You need automatic interoperability with Seurat objects — SingCellaR 'still lacks automatic object transformation' for other packages, so embedding transfer must be done manually

## Inputs

- Processed SingCellaR object (SingleCellExperiment-based) with UMAP results, Louvain cluster assignments, and normalized UMI matrix
- Supervised Harmony embeddings from get_umap.result(object)
- Cluster labels from get_clusters(object)
- Gene-per-pseudotime expression values for a trajectory gene set (e.g. erythroid-path genes)

## Outputs

- 2D UMAP scatter plots colored by cluster, pseudotime, or trajectory branch (ggplot-based Monocle3 / SingCellaR plots)
- FA2 force-directed graph layout
- Trajectory principal graph plots rooted at the HSC/MPP cluster (Figure 8E-F style panels)
- Gene-per-pseudotime heatmaps for the erythroid branch (Figure 8G-I style panels)

## How to apply

Extract the reduced-dimension coordinates from the SingCellaR object with get_umap.result(object) and pair them with cluster labels from get_clusters(object) before plotting, so the visualization reflects the stored embedding rather than recomputing one. For network-style layouts, run runFA2_ForceDirectedGraph, which depends on the fa2 Python module being reachable from R via the R environmental path. For trajectory figures, transfer the SingCellaR Harmony UMAP embedding and Louvain cluster assignments into a Monocle3 cell_data_set built from get_normalized_umi(object), run learn_graph on the transferred embeddings, call order_cells with the cl1 HSC/MPP cluster as the root node, and render UMAP plots colored by pseudotime and branch (erythroid path cl1-cl7-cl12-cl3). Set the seed before plotting because plot orientation and kBET-style outputs vary with software version and seed number.

## Related tools

- **SingCellaR** (Stores the reduced-dimension space and cluster assignments; provides get_umap.result, get_clusters, and runFA2_ForceDirectedGraph for 2D layouts) — github.com/supatt-lab/SingCellaR
- **Monocle3** (Consumes the transferred SingCellaR UMAP embedding and cluster labels in a cell_data_set to draw learn_graph trajectories and pseudotime-colored UMAP plots)
- **fa2** (Python force-atlas2 layout backend required by SingCellaR's runFA2_ForceDirectedGraph)
- **R** (Environment in which SingCellaR, its ggplot-based plotting, and the Python-backed FA2 module are invoked)

## Examples

```
Rscript -e 'umap <- get_umap.result(object); clusters <- get_clusters(object); plot_embeddings_umap(umap, clusters)'
```

## Evaluation signals

- The Monocle3 trajectory graph is a single connected principal graph rooted in the HSC/MPP (cl1) cluster
- Pseudotime increases monotonically from the HSC/MPP root toward the erythroid terminal clusters (cl1-cl7-cl12-cl3)
- Plot layout qualitatively matches Figure 8E-I panels (trajectory branches and gene-per-pseudotime heatmaps)
- runFA2_ForceDirectedGraph completes without the 'fa2 python module is not installed' or missing R environmental path errors
- Plot orientation is reproducible after fixing the seed number; rotations differing from the protocol are attributable to software version/seed, not data

## Limitations

- UMAP/FDG plot rotations can differ from the protocol output due to different software versions and seed number settings
- runFA2_ForceDirectedGraph fails if the fa2 Python module is not installed (pip install fa2) or the R environmental path for the FA2 Python module is not found by R
- SingCellaR lacks automatic object transformation to interoperate with packages such as Seurat, so embedding/cluster transfer into Monocle3 must be done manually
- Only 75 pre-curated hematopoiesis signature gene sets are provided; other systems require user-compiled gene sets for annotation overlays

## Evidence

- [abstract] a comprehensive analytical protocol to interrogate multiple datasets on SingCellaR, an analysis package in R: "a comprehensive analytical protocol to interrogate multiple datasets on SingCellaR, an analysis package in R"
- [readme] The tutorial of how to use the software can be found at https://supatt-lab.github.io/SingCellaR.Doc/: "The tutorial of how to use the software can be found at https://supatt-lab.github.io/SingCellaR.Doc/"
- [discussion] UMAP/FDG plot rotations differ from protocol due to software version and seed number differences: "This is caused by different software versions for generating plots and the seed number setting."
- [discussion] fa2 module error for force-directed graphs: "The fa2 python module is not installed!. Please install using pip (pip install fa2)"
- [discussion] SingCellaR interoperability limitation: "SingCellaR still lacks 'automatic object transformation' to interact with other existing packages, such as Seurat."
