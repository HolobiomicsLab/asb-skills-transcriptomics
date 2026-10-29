---
name: parc-clustering-parameter-tuning
description: Use when constructing the PARC cluster-graph stage of a VIA trajectory-inference
  run (e.g., on a .h5ad AnnData with PCA in obsm['X_pca']) and needing to set pruning
  resolution, granularity, and terminal-state thresholds for scalable, globally connected
  TI.
when_to_use_negative:
- The input is raw expression data not yet reduced to principal components — VIA expects
  PCA stored in adata.obsm['X_pca']
- The topology is known to be disconnected and requires the separate via_wrapper_disconnected
  entry point with preserve_disconnected=True rather than standard PARC tuning
- The goal is only a 2-D embedding visualization (UMAP/t-SNE) — the paper shows UMAP-based
  Monocle3 loses fine-grained supergroup connectivity on MOCA, so embedding-based
  clustering is the wrong basis
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: VIA
  role: Trajectory-inference wrapper that invokes PARC clustering and computes pseudotime/lineages
    from the tuned cluster-graph
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Single-cell graph clustering algorithm producing the cluster-nodes of the
    VIA cluster-graph; parameters edgepruning_clustering_resolution and too_big_factor
    tune its pruning and granularity
  repo: https://github.com/ShobiStassen/VIA
- name: dyntoy
  role: Source of toy/simulated trajectory datasets (e.g., 4-leaf multifurcation)
    used in VIA tutorials to validate clustering-parameter behavior
  repo: https://github.com/dynverse/dyntoy
- name: UMAP
  role: Optional embedding for single-cell-level visualization of the PARC cluster-graph;
    not used for clustering itself
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:aa7adbf86c1f2f03819004d70a4b154779fe24e89b5df2167062846d7344fe51
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShobiStassen/VIA
  tools:
  - Monocle3
  - PARC
  - Palantir
  - STREAM
  - UMAP
  - VIA
  - dyntoy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# parc-clustering-parameter-tuning

## Summary

Tunes the PARC graph-clustering parameters that build the cluster-graph underlying VIA trajectory inference, controlling pruning severity, cluster granularity, and terminal-state detection on single-cell atlas data.

## When to use

Use when constructing the PARC cluster-graph stage of a VIA trajectory-inference run (e.g., on a .h5ad AnnData with PCA in obsm['X_pca']) and needing to set pruning resolution, granularity, and terminal-state thresholds for scalable, globally connected TI.

## When NOT to use

- The input is raw expression data not yet reduced to principal components — VIA expects PCA stored in adata.obsm['X_pca']
- The topology is known to be disconnected and requires the separate via_wrapper_disconnected entry point with preserve_disconnected=True rather than standard PARC tuning
- The goal is only a 2-D embedding visualization (UMAP/t-SNE) — the paper shows UMAP-based Monocle3 loses fine-grained supergroup connectivity on MOCA, so embedding-based clustering is the wrong basis

## Inputs

- AnnData object with principal components stored in adata.obsm['X_pca'] (e.g., 1.3-million-cell MOCA mouse organogenesis atlas expression matrix)
- Embedding (UMAP or PHate) for single-cell-level visualization
- Root/start cell identifier (root_user, e.g., root=[42])
- Optional cell-type labels and marker genes for annotation of the cluster graph

## Outputs

- PARC cluster-graph where each node is a cluster of single cells, with adjacency relationships between cluster nodes
- Predicted terminal states (nodes above the pseudotime_threshold_TS percentile)
- Pseudotime values and multi-branch trajectory/lineage assignments
- Runtime measurements for the end-to-end VIA execution

## How to apply

Load the expression data as AnnData with principal components stored in adata.obsm['X_pca'], then invoke VIA's wrapper (via.via_wrapper) which runs PARC to cluster the single-cell graph so each cluster-graph node represents a cluster of cells. Set the key tuning parameter edgepruning_clustering_resolution (mapped to jac_std_global), the global-level graph pruning threshold for the PARC clustering stage, often mirroring it in cluster_graph_pruning to retain more connectivity in the cluster graph. Control granularity with too_big_factor (default 0.4): any cluster exceeding this share of the cell population is re-clustered with PARC to increase granularity, and optionally merge clusters smaller than small_pop (default 10 cells) into larger ones. Detect terminal states via pseudotime_threshold_TS (default 30), the percentile threshold for a potential node to qualify as a terminal state, and increase cluster-graph pruning if the trajectory computations lose connectivity. Rerun across several numbers of principal components (varying ncomps) and verify the globally connected, multi-branch topology is preserved, as demonstrated on the 1.3-million-cell MOCA atlas.

## Related tools

- **VIA** (Trajectory-inference wrapper that invokes PARC clustering and computes pseudotime/lineages from the tuned cluster-graph) — https://github.com/ShobiStassen/VIA
- **PARC** (Single-cell graph clustering algorithm producing the cluster-nodes of the VIA cluster-graph; parameters edgepruning_clustering_resolution and too_big_factor tune its pruning and granularity) — https://github.com/ShobiStassen/VIA
- **dyntoy** (Source of toy/simulated trajectory datasets (e.g., 4-leaf multifurcation) used in VIA tutorials to validate clustering-parameter behavior) — https://github.com/dynverse/dyntoy
- **UMAP** (Optional embedding for single-cell-level visualization of the PARC cluster-graph; not used for clustering itself)

## Examples

```
via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42], too_big_factor=0.4, pseudotime_threshold_TS=30)
```

## Evaluation signals

- The VIA cluster-graph remains a single connected component with concordant branch/adjacency structure across several tested numbers of principal components (ncomps)
- Clusters exceeding too_big_factor = 0.4 of the population are split by re-running PARC to increase granularity
- Terminal states correspond to nodes above the pseudotime_threshold_TS = 30 percentile and match known cell fates
- Fine-grained supergroups stay connected and neighborhood relationships are preserved (unlike UMAP-based Monocle3) on the 1.3M-cell MOCA atlas
- End-to-end runtime stays on the order of tens of minutes (~40 min) at million-cell scale

## Limitations

- edgepruning_clustering_resolution is a key tuning parameter with no fixed universal value; cluster-graph pruning must often be increased to retain connectivity, which the paper reports as sensitive to user-defined choices (cf. UMAP-dependent methods on MOCA)
- The paper does not report software versions, stress-test parameter ranges, or quantitative accuracy metrics for the clustering stage; the MOCA dataset is referenced without an accession identifier
- small_pop attempts to merge clusters with fewer than 10 cells into larger clusters, which may obscure rare cell fates

## Evidence

- [methods] edgepruning_clustering_resolution is a key tuning parameter for the PARC stage: "global level  graph pruning for PARC clustering stage. Key tuning parameter."
- [methods] too_big_factor default and re-clustering behavior: "(optional, default = 0.4) if a cluster exceeds this share of the entire cell population, then PARC will be run on the large cluster to increase granularity."
- [methods] Terminal-state percentile threshold: "(optional, default = 30) Percentile threshold for potential node to qualify as Terminal State"
- [methods] cluster_graph_pruning guidance mirrors jac_std_global: "Often set to the same value as the PARC clustering level of jac_std_global. To retain more connectivity in the clustergraph underlying the trajectory computations, increase the value"
- [readme] Concrete via_wrapper call with jac_std_global=0.15: "via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42], dataset='',"
- [intro] PARC defines the cluster-graph nodes in VIA: "Single-cell level graph is clustered such that each node represents a cluster of single cells (computed by our clustering algorithm PARC14)"
