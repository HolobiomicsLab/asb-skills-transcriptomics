---
name: trajectory-inference-terminal-state-detection
description: Use when a bifurcating or multi-fate single-cell trajectory (scRNA-seq,
  scATAC-seq, integrated, or cytometry) must have its terminal fates and branch probabilities
  detected automatically without prior fate labels.
when_to_use_negative:
- The goal is benchmarking TI methods on synthetic tree/loop topologies where a dedicated
  simulation framework (e.g., dyntoy/dyngen toy data) is the appropriate input rather
  than real annotated progenitor data
- Only a global low-resolution trajectory is needed and rare or less populous terminal
  fates are of no interest, so simpler methods like Slingshot may suffice
- The data modality lacks a meaningful low-dimensional embedding (10s-100s of PCs/features);
  VIA relies on adata.obsm['X_pca'] or equivalent reduced space
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: VIA
  role: 'Core trajectory inference: lazy-teleporting random walks on a PARC cluster-graph
    to compute pseudotime and detect terminal fates and branch probabilities'
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Clusters single cells so each node of the graph underlying VIA's trajectory
    computation represents a cell cluster
  repo: https://github.com/ShobiStassen/VIA
- name: Seurat
  role: Anchor-based integration of the scRNA-seq and scATAC-seq cardiac progenitor
    data to produce the integrated PCA embedding
- name: UMAP
  role: Provides the embedding used for single-cell level visualization of the VIA
    trajectory
- name: SlingShot
  role: Baseline fate-detection method that fails to resolve cardiomyocyte or endothelial
    fates in the comparison
- name: Palantir
  role: Baseline fate-detection method that fails to resolve either lineage in the
    comparison
- name: STREAM
  role: Baseline TI method that detects multiple spurious branches fragmenting the
    cardiac progenitor structure
- name: Monocle3
  role: Baseline TI method that fails to resolve either cell fate in the comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:f32f71d52f49d55a19537b0fe17f4d8833a0d10602975788f4e6819d5ed47876
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
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# trajectory-inference-terminal-state-detection

## Summary

Detect terminal cell fates and lineages in single-cell trajectories using VIA's lazy-teleporting random walks on a PARC cluster-graph, recovering less populous fates (e.g., Isl1+ cardiac progenitor cardiomyocyte and endothelial branches) that SlingShot, Palantir, STREAM, and Monocle3 merge or fragment.

## When to use

Use when a bifurcating or multi-fate single-cell trajectory (scRNA-seq, scATAC-seq, integrated, or cytometry) must have its terminal fates and branch probabilities detected automatically without prior fate labels.

## When NOT to use

- The goal is benchmarking TI methods on synthetic tree/loop topologies where a dedicated simulation framework (e.g., dyntoy/dyngen toy data) is the appropriate input rather than real annotated progenitor data
- Only a global low-resolution trajectory is needed and rare or less populous terminal fates are of no interest, so simpler methods like Slingshot may suffice
- The data modality lacks a meaningful low-dimensional embedding (10s-100s of PCs/features); VIA relies on adata.obsm['X_pca'] or equivalent reduced space

## Inputs

- GEO-deposited Jia et al. scRNA-seq count matrix (197 Isl1+ cardiac progenitor cells) with time-stage/fate annotations
- Jia et al. scATAC-seq count matrix (695 cells) with fate annotations
- Seurat anchor-based integrated scRNA-seq/scATAC-seq object with PCA embeddings
- AnnData object with adata.obsm['X_pca'] as VIA input
- User-specified early-stage root cell index (root_user)

## Outputs

- terminal_clusters (VIA-predicted terminal state cluster identities)
- single_cell_pt_markov (per-cell Markov-chain pseudotime)
- single_cell_bp (n_cells x n_terminal_states branch-probability matrix)
- Fig. 5g-style grouped accuracy comparison plot across modalities and methods (fig5g_fate_detection_accuracy.png)

## How to apply

Load the count matrices as a scanpy AnnData with PCA stored in adata.obsm['X_pca'] (for the cardiac progenitor task: Jia et al. scRNA-seq 197 cells, scATAC-seq 695 cells, plus a Seurat anchor-based integration, each with its own PCA embedding). VIA first builds a fast PARC cluster-graph of cells, then computes pseudotime and reconstructs lineages via lazy-teleporting random walks with MCMC refinement. Run via.via_wrapper with knn=10-30, ncomps matching retained PCs, jac_std_global=0.15, too_big_factor=0.4, pseudotime_threshold_TS=30, small_pop=10, and an early-stage root via root_user; when a modality yields separate lineage components, use via_wrapper_disconnected with root=[r1, r2] and preserve_disconnected=True. Extract per-run outputs: terminal_clusters, single_cell_pt_markov (per-cell Markov pseudotime), and single_cell_bp (n_cells x n_terminal_states branch probabilities). Score cardiomyocyte and endothelial terminal-state detection against published fate annotations and compare with SlingShot, Palantir, STREAM, and Monocle3 baselines that merge or fragment the lineages.

## Related tools

- **VIA** (Core trajectory inference: lazy-teleporting random walks on a PARC cluster-graph to compute pseudotime and detect terminal fates and branch probabilities) — https://github.com/ShobiStassen/VIA
- **PARC** (Clusters single cells so each node of the graph underlying VIA's trajectory computation represents a cell cluster) — https://github.com/ShobiStassen/VIA
- **Seurat** (Anchor-based integration of the scRNA-seq and scATAC-seq cardiac progenitor data to produce the integrated PCA embedding)
- **UMAP** (Provides the embedding used for single-cell level visualization of the VIA trajectory)
- **SlingShot** (Baseline fate-detection method that fails to resolve cardiomyocyte or endothelial fates in the comparison)
- **Palantir** (Baseline fate-detection method that fails to resolve either lineage in the comparison)
- **STREAM** (Baseline TI method that detects multiple spurious branches fragmenting the cardiac progenitor structure)
- **Monocle3** (Baseline TI method that fails to resolve either cell fate in the comparison)

## Examples

```
via.via_wrapper(adata, true_label, embedding=adata.obsm['X_pca'], knn=30, ncomps=20, jac_std_global=0.15, too_big_factor=0.4, pseudotime_threshold_TS=30, small_pop=10, root=[42]); via.via_wrapper_disconnected(adata, true_label, root=[r1,r2], preserve_disconnected=True)
```

## Evaluation signals

- VIA recovers both the cardiomyocyte and endothelial terminal fates that SlingShot, Palantir, STREAM, and Monocle3 merge or fragment, matching the reported Fig. 5g pattern
- Accuracy of detected terminal states is scored against the published Jia et al. fate annotations for cardiomyocyte and endothelial lineages on each of the three inputs (scRNA-seq, scATAC-seq, integrated)
- The saved fig5g_fate_detection_accuracy.png exists and contains accuracy bars for all three input conditions (scRNA-seq, scATAC-seq, integrated) and all compared methods
- single_cell_bp has shape (n_cells x n_terminal_states) with branch probabilities summing appropriately per cell, and pseudotime increases from the early-stage root cell
- STREAM baseline on the same data produces multiple spurious branches fragmenting the structure, confirming the comparison reproduces the published finding

## Limitations

- VIA's terminal-state detection depends on filter parameters (pseudotime_threshold_TS percentile, small_pop merge size, too_big_factor, jac_std_global pruning) whose settings are not reported in the discussion's stress tests, so behavior may vary with tuning
- Compared baselines exhibit low sensitivity to cell fates even when automated, so accuracy comparisons hinge on the specific dataset; on the cardiac progenitor data Monocle3, Slingshot and Palantir fail to resolve either fate
- No quantitative accuracy metrics or software version numbers are stated in the discussion; results defer to supplementary figures
- Datasets used for validation (hematopoiesis, pancreatic development, cardiac multi-omic) are cited without accession identifiers in the discussion, complicating exact reproduction

## Evidence

- [abstract] VIA discovers elusive lineages and less populous cell fates missed by other methods across multiple data types: "discovering elusive lineages and less populous cell fates missed by other methods across a variety of data types"
- [intro] Baselines exhibit low fate sensitivity: "even the few algorithms which automate cell fate detection (e.g., SlingShot1, Palantir2, STREAM3, and Monocle34) exhibit low sensitivity to cell fates"
- [methods] VIA wrapper invocation with root: "via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42], dataset=''"
- [methods] Disconnected trajectories with multiple roots: "via.via_wrapper_disconnected(adata_counts, true_label, embedding=adata_counts.obsm['X_pca'][:, 0:2], root=[1,1], preserve_disconnected=True"
- [methods] Terminal state percentile threshold: "(optional, default = 30) Percentile threshold for potential node to qualify as Terminal State"
- [readme] VIA reconstructs trajectories via lazy-teleporting random walks: [evidence span withheld — anchor status: external_doc]
