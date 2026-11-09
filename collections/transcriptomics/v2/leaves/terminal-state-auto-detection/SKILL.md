---
name: terminal-state-auto-detection
description: Use when scRNA-seq (or CyTOF/scATAC) data requires automated identification
  of end-state cell fates and branch probabilities without manually assigning terminal
  states, especially for rare lineages in multifurcating or disconnected trajectories.
when_to_use_negative:
- Terminal states are already known and must be imposed exactly — supply user-defined
  terminal states instead of relying on auto-detection
- Data is raw counts without a PCA representation or any graph-ready low-dimensional
  embedding
- The analysis goal is purely visualization of an existing trajectory rather than
  fate/lineage detection
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: VIA
  role: Trajectory inference algorithm that computes pseudotime and reconstructs lineages
    via lazy-teleporting random walks, auto-detecting terminal states
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Clustering algorithm used to build the cluster-graph where each node is a
    cluster of single cells, feeding terminal-state detection
  repo: https://github.com/ShobiStassen/VIA
- name: UMAP
  role: Embedding used for single-cell level visualization of the inferred trajectory
    and terminal states
- name: scanpy
  role: sc.AnnData container holding expression and adata.obsm['X_pca'] input to VIA
- name: scVelo
  role: Distributes the Bastidas-Ponce E15.5 pancreatic endocrine dataset used as
    the exemplar input
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:ce791453e01071ea44b8c20c38f4b3d69e25294ea259dff6e58a7bc166e0cfcb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShobiStassen/VIA
  tools:
  - PARC
  - UMAP
  - VIA
  - scVelo
  - scanpy
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# terminal-state-auto-detection

## Summary

Automatically detect terminal cell fates/lineages in single-cell trajectory inference using VIA's lazy-teleporting random walks on a PARC cluster-graph, without user-defined terminal states. Enables discovery of rare or elusive fates (e.g., the ~3% delta-cell population in E15.5 pancreatic endocrinogenesis) that other TI methods miss.

## When to use

Use when scRNA-seq (or CyTOF/scATAC) data requires automated identification of end-state cell fates and branch probabilities without manually assigning terminal states, especially for rare lineages in multifurcating or disconnected trajectories.

## When NOT to use

- Terminal states are already known and must be imposed exactly — supply user-defined terminal states instead of relying on auto-detection
- Data is raw counts without a PCA representation or any graph-ready low-dimensional embedding
- The analysis goal is purely visualization of an existing trajectory rather than fate/lineage detection

## Inputs

- AnnData object with normalized gene expression and PCA representation in adata.obsm['X_pca']
- 2D embedding (UMAP or PHATE) of single cells for visualization
- Optional marker-gene list for fate annotation (e.g., Sst, Hhex, Cd24a, Ins1, Ins2, Gcg)
- Optional root/start cell index or label

## Outputs

- via_object.terminal_clusters — set of automatically detected terminal-state cluster IDs
- via_object.single_cell_bp — per-cell branch probabilities (lineage likelihoods) toward each terminal state
- Pseudotime values per cell along each detected lineage

## How to apply

Assemble the data as an AnnData object with the PCA representation stored in adata.obsm['X_pca'] and compute a UMAP embedding (e.g., umap-learn with random_state=42, n_neighbors=15) for single-cell visualization. Run VIA via via_wrapper without supplying user-defined terminal states so that the default pseudotime_threshold_TS=30 percentile criterion qualifies potential nodes as Terminal States, relying on PARC clustering with jac_std_global=0.15 and too_big_factor=0.4 (clusters exceeding 40% of the population are re-clustered for granularity). Set an early root cell via root_user (e.g., root=[42]). Extract via_object.terminal_clusters to enumerate auto-detected terminal states and via_object.single_cell_bp for lineage branch probabilities. Annotate each terminal cluster by marker-gene expression (e.g., Sst/Hhex/Cd24a for delta cells, Ins1/Ins2 for beta subtypes) and validate in pseudotime ordering where applicable.

## Related tools

- **VIA** (Trajectory inference algorithm that computes pseudotime and reconstructs lineages via lazy-teleporting random walks, auto-detecting terminal states) — https://github.com/ShobiStassen/VIA
- **PARC** (Clustering algorithm used to build the cluster-graph where each node is a cluster of single cells, feeding terminal-state detection) — https://github.com/ShobiStassen/VIA
- **UMAP** (Embedding used for single-cell level visualization of the inferred trajectory and terminal states)
- **scanpy** (sc.AnnData container holding expression and adata.obsm['X_pca'] input to VIA)
- **scVelo** (Distributes the Bastidas-Ponce E15.5 pancreatic endocrine dataset used as the exemplar input)

## Examples

```
via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42])
```

## Evaluation signals

- Run recovers the expected number of terminal clusters (e.g., exactly five for the E15.5 pancreatic endocrine data: Alpha, Beta-1, Beta-2, Delta, Epsilon) with no user-supplied terminal states
- Rare-fate check: the detected rare terminal state matches the ~3% delta-cell population with exclusively elevated Hhex, Sst, and Cd24a
- Branch probabilities (via_object.single_cell_bp) assign each cell predominantly to a single terminal fate near the branch point
- Pseudotime ordering is consistent with known biology (e.g., Beta-2, Ins2+Ins1low immature, precedes Beta-1, Ins1+Ins2+ mature)
- Cluster granularity is sane: no cluster exceeds too_big_factor=0.4 of the population and clusters with <10 cells (small_pop) are merged

## Limitations

- Terminal-state detection depends on the pseudotime_threshold_TS percentile (default 30) and cluster granularity parameters (jac_std_global, too_big_factor); mis-tuning can merge or split fates
- Methods requiring UMAP (or t-SNE) embeddings before parsing can be highly susceptible to user-defined input parameters
- An early (suitable) start cell must be identifiable; root_user can be set arbitrarily but this affects pseudotime orientation
- Other automated cell-fate detection methods (SlingShot, Palantir, STREAM, Monocle3) exhibit low sensitivity to cell fates — results should be compared against such baselines

## Evidence

- [abstract] discovering elusive lineages and less populous cell fates missed by other methods across a variety of data types: "discovering elusive lineages and less populous cell fates missed by other methods across a variety of data types"
- [methods] pseudotime_threshold_TS default 30 percentile: "(optional, default = 30) Percentile threshold for potential node to qualify as Terminal State"
- [methods] too_big_factor granularity: "(optional, default = 0.4) if a cluster exceeds this share of the entire cell population, then PARC will be run on the large cluster to increase granularity."
- [methods] root setting: "We identify an early (suitable) start cell root = [42]. Can also set an arbitrary value"
- [readme] automated terminal state prediction in StaVia: "to infer pseudotimes, automated terminal state prediction and automated plotting of temporal gene dynamics along lineages"
