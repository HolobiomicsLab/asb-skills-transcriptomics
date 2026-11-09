---
name: single-cell-branch-probability-extraction
description: Use when a VIA trajectory has been run on single-cell omics data and
  you need per-cell fate assignments or branch probabilities toward detected terminal
  states to score lineage/fate resolution against annotations.
when_to_use_negative:
- Input is raw counts without a PCA embedding in adata.obsm['X_pca'] — VIA requires
  a low-dimensional representation as input
- The trajectory topology is known to be a simple linear path with a single fate —
  branch probabilities toward multiple terminal states are undefined/unnecessary
- Terminal fates are already known and provided — VIA's automated terminal-state detection
  is not needed
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: VIA
  role: Core trajectory inference algorithm producing pseudotime, terminal states
    and branch probabilities via lazy-teleporting random walks and MCMC
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Clustering step underlying VIA's cluster-graph, whose nodes are PARC clusters
  repo: https://github.com/ShobiStassen/VIA
- name: UMAP
  role: Embedding used for single-cell visualization of VIA results
- name: Seurat
  role: Anchor-based integration of scRNA-seq and scATAC-seq modalities to produce
    the integrated PCA embedding fed to VIA
- name: SlingShot
  role: Baseline fate-detection method compared against VIA's branch probability outcomes
- name: Palantir
  role: Baseline fate-detection method compared against VIA's branch probability outcomes
- name: STREAM
  role: Baseline fate-detection method compared against VIA's branch probability outcomes
- name: Monocle3
  role: Baseline fate-detection method compared against VIA's branch probability outcomes
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:23a17581bef3a0c7b86029197cb0d3fb31719a53e8a51e4840e23c87c56654f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShobiStassen/VIA
  tools:
  - Monocle3
  - PARC
  - Palantir
  - STREAM
  - Seurat
  - SlingShot
  - UMAP
  - VIA
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# Single-cell branch probability extraction

## Summary

Extracts per-cell pseudotime (Markov-chain based) and per-cell terminal-fate branch probabilities from VIA trajectory inference runs, enabling quantitative scoring of whether individual cell fates (e.g., cardiomyocyte vs endothelial from Isl1+ progenitors) are resolved or merged/fragmented.

## When to use

Use when a VIA trajectory has been run on single-cell omics data and you need per-cell fate assignments or branch probabilities toward detected terminal states to score lineage/fate resolution against annotations.

## When NOT to use

- Input is raw counts without a PCA embedding in adata.obsm['X_pca'] — VIA requires a low-dimensional representation as input
- The trajectory topology is known to be a simple linear path with a single fate — branch probabilities toward multiple terminal states are undefined/unnecessary
- Terminal fates are already known and provided — VIA's automated terminal-state detection is not needed

## Inputs

- Scanpy AnnData object with adata.obsm['X_pca'] populated (from individual scRNA-seq, scATAC-seq, or Seurat-integrated embeddings)
- VIA run object from via_wrapper or via_wrapper_disconnected
- Per-cell fate/time-stage annotations (e.g., published cardiomyocyte vs endothelial labels from Jia et al.)

## Outputs

- terminal_clusters: detected terminal-state cluster identifiers
- single_cell_pt_markov: per-cell Markov pseudotime values
- single_cell_bp: n_cells x n_terminal_states branch probability matrix
- Fate-detection accuracy scores per terminal state per input condition
- Fig. 5g-style grouped accuracy comparison plot (e.g., fig5g_fate_detection_accuracy.png)

## How to apply

After running VIA via via_wrapper (knn=10-30, ncomps matching retained PCs, jac_std_global=0.15, too_big_factor=0.4, pseudotime_threshold_TS=30, small_pop=10, with an early-stage root cell), collect three per-run outputs: terminal_clusters (detected terminal states), single_cell_pt_markov (Markov-pseudotime per cell), and single_cell_bp (an n_cells x n_terminal_states matrix of branch probabilities). For disconnected lineages (e.g., when a modality yields separate lineage components), use via_wrapper_disconnected with multiple root cells and preserve_disconnected=True so terminal states of separate components are not merged. Score fate-detection accuracy by mapping the highest-branch-probability terminal state per cell back to published fate annotations and comparing against baselines (SlingShot, Palantir, STREAM, Monocle3). VIA's lazy-teleporting random walks are specifically designed to detect less populous cell fates that these baselines merge or fragment.

## Related tools

- **VIA** (Core trajectory inference algorithm producing pseudotime, terminal states and branch probabilities via lazy-teleporting random walks and MCMC) — https://github.com/ShobiStassen/VIA
- **PARC** (Clustering step underlying VIA's cluster-graph, whose nodes are PARC clusters) — https://github.com/ShobiStassen/VIA
- **UMAP** (Embedding used for single-cell visualization of VIA results)
- **Seurat** (Anchor-based integration of scRNA-seq and scATAC-seq modalities to produce the integrated PCA embedding fed to VIA)
- **SlingShot** (Baseline fate-detection method compared against VIA's branch probability outcomes)
- **Palantir** (Baseline fate-detection method compared against VIA's branch probability outcomes)
- **STREAM** (Baseline fate-detection method compared against VIA's branch probability outcomes)
- **Monocle3** (Baseline fate-detection method compared against VIA's branch probability outcomes)

## Examples

```
v = via.via_wrapper(adata, true_label, embedding=adata.obsm['X_pca'], knn=20, ncomps=30, jac_std_global=0.15, too_big_factor=0.4, pseudotime_threshold_TS=30, small_pop=10, root=[early_stage_cell]); bp = v.single_cell_bp; pt = v.single_cell_pt_markov
```

## Evaluation signals

- VIA must recover terminal states corresponding to both cardiomyocyte and endothelial fates of the bifurcating Isl1+ progenitor trajectory on scRNA-seq, scATAC-seq, and Seurat-integrated inputs
- Branch-probability-derived fate assignments must match published fate annotations at higher accuracy than SlingShot, Palantir, STREAM, and Monocle3, which merge or fragment these lineages
- single_cell_bp must be a well-formed n_cells x n_terminal_states matrix with rows summing to a valid probability distribution over terminal states
- On modalities yielding separate lineage components, via_wrapper_disconnected with root=[r1, r2] and preserve_disconnected=True must return terminal states in both components
- The reproduced grouped accuracy plot must exist as fig5g_fate_detection_accuracy.png and contain accuracy bars for all three input conditions (scRNA-seq, scATAC-seq, integrated)

## Limitations

- Compared methods exhibit low sensitivity to cell fates; VIA's advantage is demonstrated for less populous fates, so on very small fate populations terminal-state detection may still be sensitive to small_pop (default 10 cells) and pseudotime_threshold_TS (default 30 percentile) settings
- A cluster exceeding too_big_factor (default 0.4) of the cell population triggers PARC re-clustering to increase granularity, which can affect terminal-state granularity
- Pseudotime requires a biologically meaningful root cell (root_user); an arbitrary or incorrect root skews all downstream branch probabilities

## Evidence

- [abstract] Terminal-state and branch probability outputs are core VIA features; use its documentation quote: "We present VIA, a scalable trajectory inference algorithm that overcomes these limitations by using lazy-teleporting random walks"
- [intro] Baseline fate sensitivity: "even the few algorithms which automate cell fate detection (e.g., SlingShot1, Palantir2, STREAM3, and Monocle34) exhibit low sensitivity to cell fates"
- [methods] VIA wrapper invocation from the paper's methods: "via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42], dataset='',"
- [methods] Disconnected wrapper invocation: "via.via_wrapper_disconnected(adata_counts, true_label, embedding=adata_counts.obsm['X_pca'][:, 0:2], root=[1,1], preserve_disconnected=True"
- [methods] Filter parameter rationale for terminal-state detection: [evidence span withheld — anchor status: not_found]
- [abstract] VIA's fate-detection advantage claimed in the abstract: "discovering elusive lineages and less populous cell fates missed by other methods across a variety of data types"
