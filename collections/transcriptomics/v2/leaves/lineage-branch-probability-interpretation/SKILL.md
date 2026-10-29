---
name: lineage-branch-probability-interpretation
description: Use when a scRNA-seq trajectory analysis must recover rare or small terminal
  populations (e.g., ~3% delta cells) and assign lineage labels from computed branch
  probabilities rather than manual annotation.
when_to_use_negative:
- When the topology is disconnected and requires multiple roots — use via_wrapper_disconnected
  with preserve_disconnected=True instead of the standard wrapper
- When terminal states must be user-specified rather than auto-detected (VIA's percentile
  criterion pseudotime_threshold_TS=30 would override that intent)
- When clusters smaller than the small_pop default (10 cells) are the target, since
  VIA attempts to merge clusters with population < 10 cells into larger ones
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: VIA
  role: Computes pseudotime, auto-detects terminal states, and produces single-cell
    branch probabilities via lazy-teleporting random walks
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Clusters the single-cell graph so each node is a cluster, forming the cluster-graph
    on which lineage probabilities are computed
- name: UMAP
  role: Provides the single-cell-level embedding used for trajectory visualization
- name: scanpy
  role: Assembles and manipulates the AnnData object holding the expression data and
    PCA representation
- name: scVelo
  role: Distributes the Bastidas-Ponce E15.5 pancreatic endocrine dataset loaded via
    scv.datasets
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:fd6983ce09cd5d1109eb46f507ce50af291807a4253a21737c11cac8a03ebaf9
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# lineage-branch-probability-interpretation

## Summary

Interprets VIA's automated terminal-state detection and single-cell branch probabilities (lineage likelihoods) to assign fate labels — e.g., the rare Delta cells and Beta-1/Beta-2 subtypes in E15.5 pancreatic endocrinogenesis — without manual cell-fate assignment.

## When to use

Use when a scRNA-seq trajectory analysis must recover rare or small terminal populations (e.g., ~3% delta cells) and assign lineage labels from computed branch probabilities rather than manual annotation.

## When NOT to use

- When the topology is disconnected and requires multiple roots — use via_wrapper_disconnected with preserve_disconnected=True instead of the standard wrapper
- When terminal states must be user-specified rather than auto-detected (VIA's percentile criterion pseudotime_threshold_TS=30 would override that intent)
- When clusters smaller than the small_pop default (10 cells) are the target, since VIA attempts to merge clusters with population < 10 cells into larger ones

## Inputs

- AnnData object with PCA stored in adata.obsm['X_pca'] (e.g., scVelo-distributed Bastidas-Ponce E15.5 pancreatic endocrine dataset)
- UMAP embedding (umap-learn, e.g., random_state=42, n_neighbors=15) for single-cell visualization
- Marker gene list for fate annotation (e.g., Sst, Hhex, Cd24a, Ins1, Ins2, Gcg)

## Outputs

- via_object.terminal_clusters: automatically detected terminal state cluster ids
- via_object.single_cell_bp: per-cell branch probabilities (lineage likelihoods)
- Fate assignment of terminal clusters (Alpha, Beta-1, Beta-2, Delta, Epsilon) with pseudotime ordering

## How to apply

Run VIA via via_wrapper (knn=10, ncomps=20, jac_std_global=0.15, root=[42]) on AnnData with PCA in adata.obsm['X_pca'] and a UMAP embedding, leaving terminal states auto-detected by the default pseudotime_threshold_TS=30 percentile and too_big_factor=0.4 granularity. Extract via_object.terminal_clusters to enumerate detected terminal states and via_object.single_cell_bp for per-cell lineage likelihoods. Assign each terminal cluster to a fate (Alpha, Beta-1, Beta-2, Delta, Epsilon) by marker expression (Sst, Hhex, Cd24a, Ins1, Ins2, Gcg). Confirm lineage ordering via pseudotime, e.g., that Beta-2 (Ins2+Ins1low immature) precedes Beta-1 (Ins1+Ins2+ mature). The lazy-teleporting random walk strategy underpins branch-probability computation and is what enables detection of less populous fates missed by other TI methods.

## Related tools

- **VIA** (Computes pseudotime, auto-detects terminal states, and produces single-cell branch probabilities via lazy-teleporting random walks) — https://github.com/ShobiStassen/VIA
- **PARC** (Clusters the single-cell graph so each node is a cluster, forming the cluster-graph on which lineage probabilities are computed)
- **UMAP** (Provides the single-cell-level embedding used for trajectory visualization)
- **scanpy** (Assembles and manipulates the AnnData object holding the expression data and PCA representation)
- **scVelo** (Distributes the Bastidas-Ponce E15.5 pancreatic endocrine dataset loaded via scv.datasets)

## Examples

```
v = via.via_wrapper(adata, true_label, adata.obsm['X_umap'], knn=10, ncomps=20, jac_std_global=0.15, root=[42]); terminals = v.terminal_clusters; bp = v.single_cell_bp
```

## Evaluation signals

- The run recovers exactly five auto-detected terminal clusters corresponding to the five endocrine lineages
- The Delta terminal state captures the rare (~3%) population with exclusively elevated Hhex, Sst, and Cd24a
- Beta-2 (Ins2+Ins1low immature) precedes Beta-1 (Ins1+Ins2+ mature) in pseudotime
- Terminal states were found without supplying user-defined terminal states (default pseudotime_threshold_TS=30 percentile criterion)
- single_cell_bp yields coherent lineage likelihoods consistent with the detected terminal_clusters

## Limitations

- VIA attempts to merge clusters with a population < 10 cells (small_pop default), which can threaten detection of very rare fates
- Methods requiring UMAP or t-SNE before parsing large atlases are highly susceptible to user-defined input parameters, so embedding choice affects interpretation
- Other TI algorithms (SlingShot, Palantir, STREAM, Monocle3) exhibit low sensitivity to cell fates, so cross-tool comparison of the same rare fates may fail

## Evidence

- [results] VIA automatically detects all hormone-producing pancreatic lineages, including the rare delta cells with exclusively elevated Hhex, Sst, and Cd24a, without manual assignment.: [evidence span withheld — anchor status: not_found]
- [methods] via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42]: "via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42]"
- [methods] (optional, default = 30) Percentile threshold for potential node to qualify as Terminal State: "(optional, default = 30) Percentile threshold for potential node to qualify as Terminal State"
- [abstract] discovering elusive lineages and less populous cell fates missed by other methods across a variety of data types: "discovering elusive lineages and less populous cell fates missed by other methods across a variety of data types"
- [methods] (optional, default = 10) Via *attempts* to merge Clusters with a population < 10 cells with larger clusters.: "(optional, default = 10) Via *attempts* to merge Clusters with a population < 10 cells with larger clusters."
