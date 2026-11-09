---
name: tf-motif-pseudotime-trend-visualization
description: Use when you have VIA pseudotime, terminal states, and single-cell branch
  probabilities on scATAC-seq (or analogous single-cell) data and need to visualize
  TF-motif activity dynamics along lineages and validate the hierarchy.
when_to_use_negative:
- Input is raw scRNA-seq counts without pseudotime or branch-probability outputs from
  VIA — trend curves along pseudotime require TI outputs first
- The trajectory topology is disconnected or cyclic and no root/terminal-state strategy
  has been established; use via_wrapper_disconnected instead of the standard wrapper
  for lineage trend rendering
- You only need fate-detection accuracy benchmarking (F1 scores vs. SlingShot/Palantir/STREAM/Monocle3)
  without visualization of gene dynamics
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: VIA (pyVIA)
  role: Computes pseudotime, terminal states, and single-cell branch probabilities
    (single_cell_pt_markov, single_cell_bp) via lazy-teleporting random walks, enabling
    trend curves and lineage graph rendering
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Underlying clustering algorithm that builds the cluster-graph whose nodes
    and edges are rendered as the trajectory graph
  repo: https://github.com/ShobiStassen/VIA
- name: UMAP
  role: Provides the single-cell embedding onto which the cluster-level trajectory
    graph is overlaid
- name: scanpy
  role: Used to assemble the AnnData object with adata.obsm['X_pca'] expected by via.via_wrapper
- name: pandas
  role: Loads the Buenrostro scATAC-seq TF-motif and k-mer z-score matrices from the
    VIA Datasets folder
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:795499816a1701b19f338bbe947045677437f32cb87a35dac8f4154a725a7ab8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShobiStassen/VIA
  tools:
  - PARC
  - UMAP
  - VIA
  - VIA (pyVIA)
  - pandas
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# tf-motif-pseudotime-trend-visualization

## Summary

Compute transcription-factor/motif trend curves along VIA-inferred pseudotime (e.g., GATA1, CEBPD, IRF8 in scATAC-seq hematopoiesis) and render the cluster-level trajectory graph overlaid on a single-cell embedding such as UMAP. This visualizes temporal gene-regulatory dynamics along inferred lineages rather than just computing abstract pseudotime.

## When to use

Use when you have VIA pseudotime, terminal states, and single-cell branch probabilities on scATAC-seq (or analogous single-cell) data and need to visualize TF-motif activity dynamics along lineages and validate the hierarchy.

## When NOT to use

- Input is raw scRNA-seq counts without pseudotime or branch-probability outputs from VIA — trend curves along pseudotime require TI outputs first
- The trajectory topology is disconnected or cyclic and no root/terminal-state strategy has been established; use via_wrapper_disconnected instead of the standard wrapper for lineage trend rendering
- You only need fate-detection accuracy benchmarking (F1 scores vs. SlingShot/Palantir/STREAM/Monocle3) without visualization of gene dynamics

## Inputs

- TF-motif z-score matrix (Buenrostro scATAC-seq hematopoiesis) or ChromVAR k-mer accessibility z-score matrix loaded via pandas
- AnnData object with PC projections stored in adata.obsm['X_pca']
- UMAP or PHate single-cell embedding for visualization
- VIA outputs: pseudotime, terminal states, single_cell_pt_markov, single_cell_bp
- Marker genes/motifs of interest (e.g., GATA1, CEBPD, IRF8)

## Outputs

- Gene/motif trend curves along pseudotime for each regulator (e.g., GATA1, CEBPD, IRF8)
- Cluster-level trajectory graph rendered on the UMAP embedding
- Validation figure (e.g., scATAC_BuenrostroPCs_MainFig.png) and per-run fate-detection tables (e.g., scatac_buenrostro_f1_table.csv)

## How to apply

Assemble the scATAC-seq data (TF-motif z-scores or ChromVAR k-mer accessibility z-scores from Buenrostro hematopoiesis) as an AnnData object with PC projections in adata.obsm['X_pca'], then run VIA (import pyVIA.core as via) via via.via_wrapper with PARC-based clustering defaults such as knn=30, edgepruning_clustering_resolution=0.15, cluster_graph_pruning=0.15, too_big_factor=0.4, and pseudotime_threshold_TS=30. Extract pseudotime, terminal states, single_cell_pt_markov, and single_cell_bp outputs, then compute gene/motif trend curves along pseudotime for regulators such as GATA1, CEBPD, and IRF8. Render the cluster-level trajectory graph overlaid on a UMAP (or PHate) embedding used for single-cell-level visualization. Correctness is judged by whether the rendered graph reproduces the expected MPP/LMPP/CMP to CLP/MEP/GMP hierarchy and whether trends (e.g., GATA1 toward MEP) match known hematopoietic regulator behavior.

## Related tools

- **VIA (pyVIA)** (Computes pseudotime, terminal states, and single-cell branch probabilities (single_cell_pt_markov, single_cell_bp) via lazy-teleporting random walks, enabling trend curves and lineage graph rendering) — https://github.com/ShobiStassen/VIA
- **PARC** (Underlying clustering algorithm that builds the cluster-graph whose nodes and edges are rendered as the trajectory graph) — https://github.com/ShobiStassen/VIA
- **UMAP** (Provides the single-cell embedding onto which the cluster-level trajectory graph is overlaid)
- **scanpy** (Used to assemble the AnnData object with adata.obsm['X_pca'] expected by via.via_wrapper)
- **pandas** (Loads the Buenrostro scATAC-seq TF-motif and k-mer z-score matrices from the VIA Datasets folder)

## Examples

```
import pyVIA.core as via; v = via.via_wrapper(adata, true_label, embedding=adata.obsm['X_pca'][:, :2], knn=30, root=[42], pseudotime_threshold_TS=30)
```

## Evaluation signals

- The reproduced figure matches the reference scATAC_BuenrostroPCs_MainFig.png lineage hierarchy (MPP/LMPP/CMP to CLP/MEP/GMP)
- Computed output files scATAC_BuenrostroPCs_MainFig.png and scatac_buenrostro_f1_table.csv exist and are non-empty
- TF-motif trends are biologically coherent: GATA1 trends toward the MEP branch, CEBPD/IRF8 toward GMP/dendritic fates
- Terminal states and branch probabilities are stable across repeated runs and PC counts (F1-scores remain stable above 50 PCs)
- Cluster-graph edges overlaid on the UMAP align with the embedding's apparent differentiation flow without spurious disconnected clusters

## Limitations

- VIA's default parameters (e.g., pseudotime_threshold_TS=30, too_big_factor=0.4, small_pop=10) may need tuning; cluster-graph connectivity depends on cluster_graph_pruning, which should often match jac_std_global
- If the visualized cluster graph edges are too busy, edgebundle_pruning_twice=True is needed for a second edge-bundling iteration
- Missing signals in the source article: no software version numbers for VIA or compared methods, no accession identifiers for validation datasets, and no explicit limitations of VIA itself are acknowledged
- Other TI methods show performance drops beyond 50 PCs on scATAC-seq k-mer Z Scores data, so baseline trend/benchmark comparisons degrade with high PC counts

## Evidence

- [readme] VIA discovers lineages with automated temporal gene dynamics plotting: "automated plotting of temporal gene dynamics along lineages"
- [methods] UMAP embedding is used for single-cell-level visualization of the trajectory: "# use UMAP or PHate to obtain embedding that is used for single-cell level visualization"
- [methods] AnnData assembly with PCA in obsm is the required input format for VIA: "adata = sc.AnnData(data_genes)"
- [methods] VIA wrapper call with embedding, knn, and root parameters grounds the example invocation: "via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42], dataset=''"
- [readme] The scATAC-seq hematopoiesis trajectory represented by VIA graphs is featured in the README: [evidence span withheld — anchor status: external_doc]
