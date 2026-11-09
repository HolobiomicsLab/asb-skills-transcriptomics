---
name: parc-cluster-graph-construction
description: Use when a single-cell dataset (e.g. mESC CyTOF, scRNA-seq) must be reduced
  to a cluster-level graph before trajectory inference, especially at atlas scale
  (10^2–10^6 cells).
when_to_use_negative:
- When cells are already grouped into clusters or a cluster-level graph exists and
  only downstream pseudotime/lineage computation is needed
- When a separate dimensionality-reduction embedding (UMAP/t-SNE) is being relied
  on as the analysis substrate — the article warns such methods are 'highly susceptible
  to user defined input parameters' and compromise global information at scale
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: PARC
  role: Clustering algorithm that builds the single-cell-level graph in which each
    node represents a cluster of cells, feeding VIA's trajectory stage
  repo: https://github.com/ShobiStassen/VIA
- name: VIA
  role: Downstream consumer of the PARC cluster graph; runs lazy-teleporting random
    walks on it to compute pseudotime and lineages
  repo: https://github.com/ShobiStassen/VIA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:b5b7a660365abb66ce848b58e088e8d4480318c1e943e94a3e03819516dbf4da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShobiStassen/VIA
  tools:
  - PARC
  - VIA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# parc-cluster-graph-construction

## Summary

Constructs the PARC cluster graph — a fast, scalable graph whose nodes are clusters of single cells — that serves as the substrate for VIA's pseudotime and lineage inference on single-cell omics data.

## When to use

Use when a single-cell dataset (e.g. mESC CyTOF, scRNA-seq) must be reduced to a cluster-level graph before trajectory inference, especially at atlas scale (10^2–10^6 cells).

## When NOT to use

- When cells are already grouped into clusters or a cluster-level graph exists and only downstream pseudotime/lineage computation is needed
- When a separate dimensionality-reduction embedding (UMAP/t-SNE) is being relied on as the analysis substrate — the article warns such methods are 'highly susceptible to user defined input parameters' and compromise global information at scale

## Inputs

- Single-cell feature matrix (e.g. CyTOF antibody intensities or scRNA-seq counts) as AnnData with PCA stored in adata.obsm['X_pca']
- Pruning/clustering parameters: edgepruning_clustering_resolution, too_big_factor, small_pop, Leiden resolution_parameter, random_seed

## Outputs

- PARC cluster graph whose nodes are cell clusters with jaccard-pruned weighted edges
- Cluster-level graph underlying VIA's trajectory and pseudotime computations

## How to apply

Cluster single cells into graph nodes using PARC with jaccard-weighted edge pruning: set edgepruning_clustering_resolution=0.15 (the key global tuning parameter), too_big_factor=0.4 so any cluster exceeding 0.4 of the population is re-clustered for granularity, and small_pop=10 so clusters under 10 cells are merged into larger ones. Run Leiden clustering with resolution_parameter=1 and random_seed=42 for reproducibility. Optionally apply a second, local pruning threshold (edgepruning_clustering_resolution_local, in standard deviations above the mean minkowski distance between neighbors) and set cluster_graph_pruning, often equal to jac_std_global, to control connectivity in the cluster graph underlying trajectory computations; increase it to retain more connectivity. The resulting cluster graph feeds VIA's lazy-teleporting random-walk stage, so over-pruning here will fragment trajectories while under-pruning leaves a busy graph (edgebundle_pruning_twice=True can condense visualization edges).

## Related tools

- **PARC** (Clustering algorithm that builds the single-cell-level graph in which each node represents a cluster of cells, feeding VIA's trajectory stage) — https://github.com/ShobiStassen/VIA
- **VIA** (Downstream consumer of the PARC cluster graph; runs lazy-teleporting random walks on it to compute pseudotime and lineages) — https://github.com/ShobiStassen/VIA

## Examples

```
via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, too_big_factor=0.4, small_pop=10, root=[42], random_seed=42)
```

## Evaluation signals

- Cluster graph respects small_pop=10: no cluster with fewer than 10 cells remains unmerged
- No cluster exceeds too_big_factor=0.4 of the total population without being re-clustered for finer granularity
- Cluster graph retains global connectivity sufficient for downstream VIA pseudotime; disconnected or overly busy edges indicate mis-set pruning (increase cluster_graph_pruning or set edgebundle_pruning_twice)
- Downstream standalone pseudotimes from the graph correlate strongly and positively (Spearman) with annotated day labels and full-VIA output on the mESC CyTOF dataset
- Reproducibility: identical graph under Leiden resolution_parameter=1 with random_seed=42

## Limitations

- edgepruning_clustering_resolution is a key tuning parameter; overly aggressive pruning can fragment trajectories and lose fine-grained sub-trajectories
- Cluster-graph edges may appear too busy visually and require a second round of edge bundling (edgebundle_pruning_twice=True)
- Methods depending on pre-reduction embeddings (UMAP/t-SNE) before graph construction are susceptible to user-defined input parameters and may lose global connectivity on million-cell atlases

## Evidence

- [intro] Single-cell level graph is clustered such that each node represents a cluster of single cells (computed by our clustering algorithm PARC14): "Single-cell level graph is clustered such that each node represents a cluster of single cells (computed by our clustering algorithm PARC14)"
- [methods] global level graph pruning for PARC clustering stage. Key tuning parameter.: "global level  graph pruning for PARC clustering stage. Key tuning parameter."
- [methods] too_big_factor ... if a cluster exceeds this share of the entire cell population, then PARC will be run on the large cluster to increase granularity.: "(optional, default = 0.4) if a cluster exceeds this share of the entire cell population, then PARC will be run on the large cluster to increase granularity."
- [methods] Often set to the same value as the PARC clustering level of jac_std_global. To retain more connectivity in the clustergraph underlying the trajectory computations, increase the value: "Often set to the same value as the PARC clustering level of jac_std_global. To retain more connectivity in the clustergraph underlying the trajectory computations, increase the value"
- [discussion] VIA alleviates these challenges by fast and scalable construction of cluster-graph of cells, followed by pseudotime, and reconstructing cell lineages: "fast and scalable construction of cluster-graph of cells, followed by pseudotime, and reconstructing cell lineages"
