---
name: terminal-state-and-lineage-reconstruction
description: Use when a single-cell omics dataset (AnnData with PCA in obsm['X_pca'])
  needs pseudotime, automated terminal state prediction, and lineage/branch probabilities,
  especially for tree-like, cyclic, or disconnected topologies.
when_to_use_negative:
- The goal is only clustering or dimensionality reduction without ordering of cells
  along trajectories or fate detection
- The dataset has no meaningful differentiation dynamics or trajectory structure (e.g.
  a static cell atlas of unrelated cell types)
- Cells require time/trajectory labels along a known schedule — VIA's disconnected/time-aware
  variant needs t_diff_step-compatible labeled times, which are not available here
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: VIA
  role: 'Core trajectory inference algorithm: PARC cluster-graph construction, pseudotime
    via lazy-teleporting random walks, terminal state detection, and lineage branch
    probabilities'
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Clustering algorithm underlying VIA's cluster-graph, tuned via edgepruning_clustering_resolution,
    cluster_graph_pruning, too_big_factor, and small_pop
  repo: https://github.com/ShobiStassen/VIA
- name: UMAP
  role: Embedding on which the VIA cluster-level trajectory graph and single-cell
    pseudotime are visualized
- name: scanpy
  role: Constructs the AnnData object and stores PC projections in adata.obsm['X_pca']
    required by via.via_wrapper
- name: SlingShot
  role: Baseline TI method whose cell-fate detection F1-scores are compared against
    VIA across increasing PC counts
- name: Palantir
  role: Baseline TI method compared for fate-detection sensitivity and robustness
    to number of PCs
- name: STREAM
  role: Baseline tree-like TI method used for F1-score comparison on scATAC-seq hematopoiesis
- name: Monocle3
  role: Baseline TI method used for cell-fate detection comparison
- name: dyntoy
  role: Source of synthetic multifurcating/disconnected toy datasets used in the starter
    tutorials to validate VIA topology inference
  repo: dynverse/dyntoy
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:36d5bec0d4b98154941f4cea79bf18bdf2605fdf6af74d67f587510891a44e35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShobiStassen/VIA
  tools:
  - Monocle3
  - PARC
  - Palantir
  - STREAM
  - SlingShot
  - UMAP
  - VIA
  - dyntoy
  - pandas
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# terminal-state-and-lineage-reconstruction

## Summary

Reconstruct single-cell trajectories, pseudotime, and cell fates using VIA's PARC-based cluster-graph and lazy-teleporting random walks, which automate terminal state detection and lineage (branch probability) assignment across scRNA-seq, scATAC-seq, cytometry, and imaging modalities.

## When to use

Use when a single-cell omics dataset (AnnData with PCA in obsm['X_pca']) needs pseudotime, automated terminal state prediction, and lineage/branch probabilities, especially for tree-like, cyclic, or disconnected topologies.

## When NOT to use

- The goal is only clustering or dimensionality reduction without ordering of cells along trajectories or fate detection
- The dataset has no meaningful differentiation dynamics or trajectory structure (e.g. a static cell atlas of unrelated cell types)
- Cells require time/trajectory labels along a known schedule — VIA's disconnected/time-aware variant needs t_diff_step-compatible labeled times, which are not available here

## Inputs

- AnnData object with feature matrix (e.g. scRNA-seq counts, TF-motif z-scores, ChromVAR k-mer accessibility z-scores)
- PCA projections placed in adata.obsm['X_pca']
- UMAP or PHATE embedding for single-cell visualization
- Optional true cell-type labels and root/start cell identifier

## Outputs

- Per-cell pseudotime (single_cell_pt_markov)
- Automatically detected terminal states
- Single-cell branch probabilities (single_cell_bp) defining lineage assignments
- Cluster-level trajectory graph overlaid on UMAP embedding
- Gene/motif trend curves along pseudotime (e.g. GATA1, CEBPD, IRF8)

## How to apply

Load the data as an AnnData object with PCA projections stored in adata.obsm['X_pca'], and compute a UMAP (or PHATE) embedding for single-cell visualization. Run VIA via via.via_wrapper with a root/start cell (root_user, e.g. root=[42]) and PARC-based clustering parameters such as knn=30 (or knn=10 in the toy tutorial), jac_std_global/edgepruning_clustering_resolution=0.15, cluster_graph_pruning=0.15, too_big_factor=0.4, and pseudotime_threshold_TS=30 (percentile threshold for terminal state qualification). Extract pseudotime, terminal states, single_cell_pt_markov, and single_cell_bp (branch probabilities) from the VIA object to establish lineage hierarchies such as MPP/LMPP/CMP to CLP/MEP/GMP in the scATAC-seq hematopoiesis data. Optionally compute gene/motif trend curves along pseudotime (e.g. GATA1, CEBPD, IRF8) and render the cluster-level trajectory graph overlaid on the UMAP embedding. For disconnected or cyclic topologies use via.via_wrapper_disconnected with preserve_disconnected=True and multiple roots; robustness of fate detection (e.g. F1-scores beyond 50 PCs) can be tabulated against baselines like SlingShot, Palantir, STREAM, and Monocle3.

## Related tools

- **VIA** (Core trajectory inference algorithm: PARC cluster-graph construction, pseudotime via lazy-teleporting random walks, terminal state detection, and lineage branch probabilities) — https://github.com/ShobiStassen/VIA
- **PARC** (Clustering algorithm underlying VIA's cluster-graph, tuned via edgepruning_clustering_resolution, cluster_graph_pruning, too_big_factor, and small_pop) — https://github.com/ShobiStassen/VIA
- **UMAP** (Embedding on which the VIA cluster-level trajectory graph and single-cell pseudotime are visualized)
- **scanpy** (Constructs the AnnData object and stores PC projections in adata.obsm['X_pca'] required by via.via_wrapper)
- **SlingShot** (Baseline TI method whose cell-fate detection F1-scores are compared against VIA across increasing PC counts)
- **Palantir** (Baseline TI method compared for fate-detection sensitivity and robustness to number of PCs)
- **STREAM** (Baseline tree-like TI method used for F1-score comparison on scATAC-seq hematopoiesis)
- **Monocle3** (Baseline TI method used for cell-fate detection comparison)
- **dyntoy** (Source of synthetic multifurcating/disconnected toy datasets used in the starter tutorials to validate VIA topology inference) — dynverse/dyntoy

## Examples

```
via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42], dataset='')
```

## Evaluation signals

- Reproduced trajectory graph recovers the expected lineage hierarchy (MPP/LMPP/CMP to CLP/MEP/GMP) matching the reference scATAC_BuenrostroPCs_MainFig.png
- Terminal state detection F1-scores remain stable for VIA at PC counts above 50 where baseline TI methods (SlingShot, Palantir, STREAM, Monocle3) degrade
- Pseudotime increases monotonically from the user-set root cell toward detected terminal states
- Gene/motif trend curves (GATA1, CEBPD, IRF8) show coherent dynamics along pseudotime consistent with known hematopoietic regulators
- Validation artifacts (scATAC_BuenrostroPCs_MainFig.png and scatac_buenrostro_f1_table.csv) are produced and consistent with reference outputs

## Limitations

- Performance of compared TI methods (and VIA parameters) depends on user-tunable settings; key tuning parameters include edgepruning_clustering_resolution and cluster_graph_pruning, and VIA's behavior under varying parameter ranges is only characterized via stress tests deferred to supplementary figures
- If the visualized cluster graph edges appear too busy, a second edge-bundling iteration (edgebundle_pruning_twice=True) is needed, and small clusters (< small_pop, default 10 cells) are merged into larger clusters, which can obscure rare fates
- Baselines like SlingShot, Palantir, STREAM, and Monocle3 exhibit low sensitivity to cell fates and can miss less populous cell fates; methods requiring UMAP/t-SNE before parsing large atlases (e.g. MOCA) are highly susceptible to user-defined input parameters

## Evidence

- [intro] we present VIA, a graph-based TI algorithm that uses a new strategy to compute pseudotime, and reconstruct cell lineages based on lazy-teleporting random walks: "we present VIA, a graph-based TI algorithm that uses a new strategy to compute pseudotime, and reconstruct cell lineages based on lazy-teleporting random walks"
- [methods] adata = sc.AnnData(data_genes)
	adata.obsm['X_pca'] = data.values: "adata = sc.AnnData(data_genes)
	adata.obsm['X_pca'] = data.values"
- [methods] via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42], dataset='',: "via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42], dataset='',"
- [methods] pseudotime_threshold_TS  (optional, default = 30) Percentile threshold for potential node to qualify as Terminal State: "pseudotime_threshold_TS  (optional, default = 30) Percentile threshold for potential node to qualify as Terminal State"
- [discussion] reconstructing cell lineages based on lazy-teleporting random walks and MCMC simulations: "reconstructing cell lineages based on lazy-teleporting random walks and MCMC simulations"
- [readme] Generalizable and data modality agnostic StaVia still offers all the functionality of Via 1.0 across single-cell data modalities (scRNA-seq, imaging and flow cyometry, scATAC-seq): [evidence span withheld — anchor status: not_found]
