---
name: pancreatic-endocrine-cell-type-annotation
description: Use when scRNA-seq of pancreatic endocrine development (e.g., E15.5 Bastidas-Ponce
  pancreas data) must have rare hormone-producing fates like delta cells and Beta-1/Beta-2
  subtypes detected without manual cell-fate assignment.
when_to_use_negative:
- The input lacks a gene-by-cell expression matrix with an AnnData PCA representation
  (adata.obsm['X_pca']), e.g., it is already a cluster-level feature table.
- The trajectory is disconnected and requires multiple explicit roots — use via_wrapper_disconnected
  with preserve_disconnected=True instead of the standard wrapper.
- Terminal states are already known and manually specified, making VIA's auto-detection
  (pseudotime_threshold_TS=30, too_big_factor=0.4) unnecessary.
tools:
- name: VIA
  role: 'Trajectory inference: cluster-graph construction, pseudotime, auto-detection
    of terminal states, and branch-probability lineage assignment'
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Underlying clustering of single cells into graph nodes used to build VIA's
    cluster-graph
  repo: https://github.com/ShobiStassen/VIA
- name: UMAP
  role: Single-cell level visualization embedding passed to via_wrapper
- name: scVelo
  role: Source of the Bastidas-Ponce E15.5 pancreatic endocrine dataset via scv.datasets
- name: scanpy
  role: Assembly and handling of the AnnData object with PCA in obsm['X_pca']
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:86198d2754692aa27c96695b3edc715374948a68645a62e75c7d051ae6c2718e
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# pancreatic-endocrine-cell-type-annotation

## Summary

Automated detection and marker-based annotation of pancreatic endocrine lineages (Alpha, Beta-1, Beta-2, Delta, Epsilon) in E15.5 murine pancreas scRNA-seq by running VIA's lazy-teleporting random-walk trajectory inference with auto-detected terminal states, then assigning fates via marker genes (Sst, Hhex, Cd24a, Ins1, Ins2, Gcg).

## When to use

Use when scRNA-seq of pancreatic endocrine development (e.g., E15.5 Bastidas-Ponce pancreas data) must have rare hormone-producing fates like delta cells and Beta-1/Beta-2 subtypes detected without manual cell-fate assignment.

## When NOT to use

- The input lacks a gene-by-cell expression matrix with an AnnData PCA representation (adata.obsm['X_pca']), e.g., it is already a cluster-level feature table.
- The trajectory is disconnected and requires multiple explicit roots — use via_wrapper_disconnected with preserve_disconnected=True instead of the standard wrapper.
- Terminal states are already known and manually specified, making VIA's auto-detection (pseudotime_threshold_TS=30, too_big_factor=0.4) unnecessary.

## Inputs

- E15.5 pancreatic endocrine scRNA-seq dataset (scVelo-distributed Bastidas-Ponce AnnData)
- PCA representation stored in adata.obsm['X_pca']
- Endocrine marker gene list (Sst, Hhex, Cd24a, Ins1, Ins2, Gcg)
- UMAP embedding for single-cell visualization (e.g., umap-learn, random_state=42, n_neighbors=15)

## Outputs

- Auto-detected terminal clusters via via_object.terminal_clusters (five endocrine lineages)
- Lineage/branch probabilities per cell via via_object.single_cell_bp
- Pseudotime ordering of fates (Beta-2 before Beta-1)
- Marker-based fate labels: Alpha, Beta-1, Beta-2, Delta, Epsilon
- UMAP-based trajectory visualization colored by fate and pseudotime

## How to apply

Load the E15.5 pancreatic endocrine data distributed with scVelo (scv.datasets Bastidas-Ponce) as an AnnData object with the PCA stored in adata.obsm['X_pca'], and compute a UMAP embedding (e.g., random_state=42, n_neighbors=15) for visualization. Define endocrine marker genes (Sst, Hhex, Cd24a, Ins1, Ins2, Gcg) as the true_label/marker_genes input. Run via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42]) without user-defined terminal states, so the default pseudotime_threshold_TS=30 percentile criterion and too_big_factor=0.4 cluster-granularity rule auto-detect terminal states. Extract via_object.terminal_clusters and via_object.single_cell_bp branch probabilities (lineage likelihoods) to enumerate terminal fates and assign each to Alpha, Beta-1, Beta-2, Delta, or Epsilon by marker expression. Validate that the Delta fate captures the rare (~3%) population with exclusively elevated Hhex, Sst, and Cd24a, and that Beta-2 (Ins2+/Ins1-low immature) precedes Beta-1 (Ins1+/Ins2+ mature) in pseudotime.

## Related tools

- **VIA** (Trajectory inference: cluster-graph construction, pseudotime, auto-detection of terminal states, and branch-probability lineage assignment) — https://github.com/ShobiStassen/VIA
- **PARC** (Underlying clustering of single cells into graph nodes used to build VIA's cluster-graph) — https://github.com/ShobiStassen/VIA
- **UMAP** (Single-cell level visualization embedding passed to via_wrapper)
- **scVelo** (Source of the Bastidas-Ponce E15.5 pancreatic endocrine dataset via scv.datasets)
- **scanpy** (Assembly and handling of the AnnData object with PCA in obsm['X_pca'])

## Examples

```
via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42])
```

## Evaluation signals

- The run recovers exactly five auto-detected terminal clusters corresponding to Alpha, Beta-1, Beta-2, Delta, and Epsilon fates.
- The Delta terminal state matches the rare (~3%) population with exclusively elevated Hhex, Sst, and Cd24a.
- Beta-2 (Ins2+/Ins1-low immature) precedes Beta-1 (Ins1+/Ins2+ mature) along pseudotime.
- via_object.single_cell_bp yields lineage likelihoods that unambiguously assign each terminal cluster to one fate by marker expression.
- AnnData object is correctly formed with adata.obsm['X_pca'] populated before calling via_wrapper.

## Limitations

- Clusters with fewer than 10 cells may be merged into larger clusters (small_pop default = 10), which can threaten detection of very rare fates.
- Cluster granularity depends on too_big_factor (0.4) and jac_std_global (0.15); poor pruning can lose or merge small terminal populations.
- The paper gives no accession identifiers for the pancreatic development validation dataset, complicating direct data retrieval from the article alone.

## Evidence

- [methods] via_wrapper call with key parameters: "via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42], dataset='',"
- [methods] AnnData PCA storage requirement: "adata = sc.AnnData(data_genes)
	adata.obsm['X_pca'] = data.values"
- [methods] Terminal state auto-detection threshold: "(optional, default = 30) Percentile threshold for potential node to qualify as Terminal State"
- [discussion] Paper finding on rare delta cell detection: "VIA distinguished between dendritic subtypes in an scRNA-seq hematopoiesis dataset; identified the rare delta cell islet in pancreatic development"
- [readme] README on automated terminal state prediction: "to infer pseudotimes, automated terminal state prediction and automated plotting of temporal gene dynamics along lineages."
