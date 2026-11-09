---
name: scrna-scatac-multimodal-integration
description: Use when paired or separate scRNA-seq and scATAC-seq count matrices with
  time-stage/fate annotations must be integrated and used to reconstruct a bifurcating
  trajectory and detect terminal cell fates.
when_to_use_negative:
- Data consists of a single modality with a simple tree-like topology where established
  methods (e.g., Slingshot, STREAM) suffice without integration.
- Only normalized expression/feature matrices are available without raw per-modality
  count matrices or fate annotations to score terminal-state recovery.
- The goal is generic Seurat integration quality assessment rather than downstream
  trajectory inference and fate detection.
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_2229
tools:
- name: VIA
  role: Trajectory inference and automated terminal-state/branch-probability detection
    on individual and integrated PCA embeddings via via_wrapper and via_wrapper_disconnected
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Graph clustering underlying VIA's cluster-graph construction of cells
  repo: https://github.com/ShobiStassen/VIA
- name: UMAP
  role: Embedding for single-cell level visualization of inferred trajectories
- name: Seurat
  role: Anchor-based integration of the scRNA-seq and scATAC-seq modalities
- name: SlingShot
  role: Baseline TI method for fate-detection accuracy comparison (fails to resolve
    both lineages)
- name: Palantir
  role: Baseline TI method for fate-detection accuracy comparison (fails to resolve
    either fate)
- name: STREAM
  role: Baseline TI method; detects multiple spurious branches that fragment the cardiac
    progenitor structure
- name: Monocle3
  role: Baseline TI method for fate-detection accuracy comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:2dfd1777ffdace22dc09419424065ca6eb2464d1e0e54ca317c0d5db6bddaf0b
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
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# scrna-scatac-multimodal-integration

## Summary

Normalize and Seurat-anchor-integrate paired scRNA-seq and scATAC-seq cardiac progenitor datasets (Jia et al., 197 RNA cells and 695 ATAC cells), then feed individual-modality and integrated PCA embeddings into VIA to recover the bifurcating Isl1+ cardiomyocyte and endothelial fates that baseline TI methods merge or fragment.

## When to use

Use when paired or separate scRNA-seq and scATAC-seq count matrices with time-stage/fate annotations must be integrated and used to reconstruct a bifurcating trajectory and detect terminal cell fates.

## When NOT to use

- Data consists of a single modality with a simple tree-like topology where established methods (e.g., Slingshot, STREAM) suffice without integration.
- Only normalized expression/feature matrices are available without raw per-modality count matrices or fate annotations to score terminal-state recovery.
- The goal is generic Seurat integration quality assessment rather than downstream trajectory inference and fate detection.

## Inputs

- GEO-deposited Jia et al. scRNA-seq cardiac progenitor count matrix (197 cells) with time-stage/fate annotations
- GEO-deposited Jia et al. scATAC-seq cardiac progenitor count matrix (695 cells) with time-stage/fate annotations
- Seurat anchor-based integrated scRNA-seq/scATAC-seq object
- PCA embeddings stored per modality and for the integrated object in adata.obsm['X_pca']
- Early-stage root cell identifier for the Isl1+ progenitor trajectory

## Outputs

- Per-run VIA terminal_clusters for each input (scRNA-seq, scATAC-seq, integrated)
- Single-cell Markov pseudotime (single_cell_pt_markov)
- Branch probability matrix single_cell_bp (n_cells x n_terminal_states)
- Fig. 5g-style grouped accuracy comparison plot (fig5g_fate_detection_accuracy.png) of VIA vs. baselines across the three input conditions

## How to apply

Load the GEO-deposited Jia et al. cardiac progenitor scRNA-seq (197 cells) and scATAC-seq (695 cells) count matrices with their time-stage/fate annotations, normalize each modality, and integrate them with Seurat anchor-based integration. Compute PCA embeddings for each individual modality and for the integrated object, placing them in adata.obsm['X_pca'] for scanpy AnnData input to VIA. Run VIA via via_wrapper on all three inputs (knn=10-30, ncomps matching retained PCs, jac_std_global=0.15, too_big_factor=0.4, pseudotime_threshold_TS=30, small_pop=10) with an early-stage root cell; where a modality yields separate lineage components, use via_wrapper_disconnected with root=[r1, r2] and preserve_disconnected=True. Extract terminal_clusters, single_cell_pt_markov pseudotime, and single_cell_bp branch probabilities (n_cells x n_terminal_states), then score cardiomyocyte and endothelial terminal-state detection against the published fate annotations and compare with SlingShot, Palantir, STREAM, and Monocle3, which fail to resolve both lineages.

## Related tools

- **VIA** (Trajectory inference and automated terminal-state/branch-probability detection on individual and integrated PCA embeddings via via_wrapper and via_wrapper_disconnected) — https://github.com/ShobiStassen/VIA
- **PARC** (Graph clustering underlying VIA's cluster-graph construction of cells) — https://github.com/ShobiStassen/VIA
- **UMAP** (Embedding for single-cell level visualization of inferred trajectories)
- **Seurat** (Anchor-based integration of the scRNA-seq and scATAC-seq modalities)
- **SlingShot** (Baseline TI method for fate-detection accuracy comparison (fails to resolve both lineages))
- **Palantir** (Baseline TI method for fate-detection accuracy comparison (fails to resolve either fate))
- **STREAM** (Baseline TI method; detects multiple spurious branches that fragment the cardiac progenitor structure)
- **Monocle3** (Baseline TI method for fate-detection accuracy comparison)

## Examples

```
via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42], dataset='')
```

## Evaluation signals

- Reproduced figure must show VIA recovering both the cardiomyocyte and the endothelial terminal fates that baselines merge or fragment, matching the reported Fig. 5g pattern of VIA detecting less populous cell fates missed by other methods.
- fig5g_fate_detection_accuracy.png exists and contains accuracy bars for all three input conditions (scRNA-seq, scATAC-seq, integrated).
- VIA terminal_clusters from each run map to both published cardiomyocyte and endothelial fate annotations.
- single_cell_bp matrix has shape n_cells x n_terminal_states with sensible probability mass (rows sum toward 1) for each terminal state.
- Where a modality yields separate lineage components, via_wrapper_disconnected with root=[r1, r2] and preserve_disconnected=True retains both components.

## Limitations

- ScATAC-seq input has fewer informative dimensions (695 cells), so modality-specific embeddings may require via_wrapper_disconnected with multiple roots when components are disconnected.
- Baseline methods (Monocle3, Slingshot, Palantir, STREAM) exhibit low sensitivity to cell fates, so cross-method comparisons on small populations can be dominated by method-specific clustering granularity.
- VIA behaves predictably across parameter settings but results depend on tuning knobs such as jac_std_global, too_big_factor, pseudotime_threshold_TS, and small_pop.

## Evidence

- [abstract] VIA discovers elusive lineages and less populous cell fates missed by other methods across multiple data types including proteomic, epigenomic, multi-omics, and morphological datasets: "discovering elusive lineages and less populous cell fates missed by other methods across a variety of data types"
- [intro] even the few algorithms which automate cell fate detection exhibit low sensitivity to cell fates: "even the few algorithms which automate cell fate detection (e.g., SlingShot1, Palantir2, STREAM3, and Monocle34) exhibit low sensitivity to cell fates"
- [methods] Load data as AnnData with PCA stored in adata.obsm['X_pca']: "adata = sc.AnnData(data_genes)
	adata.obsm['X_pca'] = data.values"
- [methods] Run VIA for disconnected trajectories with multiple roots: "via.via_wrapper_disconnected(adata_counts, true_label, embedding=adata_counts.obsm['X_pca'][:, 0:2], root=[1,1], preserve_disconnected=True"
- [readme] StaVia is generalizable and data modality agnostic across scRNA-seq, imaging and flow cytometry, scATAC-seq: [evidence span withheld — anchor status: external_doc]
