---
name: pc-count-robustness-benchmarking
description: Use when assessing sensitivity of trajectory inference / terminal-state
  detection to the dimensionality of the input PC representation, e.g. scATAC-seq
  (ChromVAR k-mer or TF-motif z-scores) swept across ~10–100 PCs to compare fate-detection
  F1 stability.
when_to_use_negative:
- Input is raw scATAC-seq fragment/count data rather than preprocessed TF-motif or
  k-mer z-score matrices with precomputed PC projections
- The goal is topology benchmarking on simulated toy data (e.g. dynverse/dyntoy 4-leaf
  or disconnected simulations) rather than PC-count sensitivity on real scATAC-seq
  data
- The comparison focuses on scalability/runtime (e.g. the 1.3-million-cell MOCA atlas)
  rather than fate-detection accuracy as a function of dimensionality
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_2229
tools:
- name: VIA
  role: Primary trajectory inference method whose terminal-state detection F1 robustness
    across PC counts is benchmarked; run via via.via_wrapper on adata.obsm['X_pca']
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Clustering algorithm underlying VIA's cluster-graph construction; controlled
    via edgepruning_clustering_resolution and cluster_graph_pruning
  repo: https://github.com/ShobiStassen/VIA
- name: UMAP
  role: Embedding used for single-cell visualization of the VIA cluster-level trajectory
    graph
- name: scanpy
  role: Used to assemble the AnnData object holding PC projections in adata.obsm['X_pca']
- name: pandas
  role: Loads the Buenrostro TF-motif and k-mer z-score matrices from the VIA Datasets
    folder
- name: SlingShot
  role: Baseline TI method compared against VIA in F1-vs-PC-count fate detection
- name: Palantir
  role: Baseline TI method compared against VIA in F1-vs-PC-count fate detection
- name: STREAM
  role: Baseline TI method compared against VIA in F1-vs-PC-count fate detection
- name: Monocle3
  role: Baseline TI method compared against VIA in F1-vs-PC-count fate detection
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:7bcc312404910ab408bf32cb8a897fe6087fdff5c6ac991820592694494272de
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
  - pandas
  - scanpy
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# pc-count-robustness-benchmarking

## Summary

Benchmarks how terminal-state and lineage detection accuracy of trajectory inference methods (VIA vs SlingShot, Palantir, STREAM, Monocle3) degrades as the number of input principal components increases, using scATAC-seq hematopoiesis data preprocessed with TF-motif and k-mer z-score protocols. It demonstrates VIA's F1-score robustness beyond ~50 PCs where baselines degrade.

## When to use

Use when assessing sensitivity of trajectory inference / terminal-state detection to the dimensionality of the input PC representation, e.g. scATAC-seq (ChromVAR k-mer or TF-motif z-scores) swept across ~10–100 PCs to compare fate-detection F1 stability.

## When NOT to use

- Input is raw scATAC-seq fragment/count data rather than preprocessed TF-motif or k-mer z-score matrices with precomputed PC projections
- The goal is topology benchmarking on simulated toy data (e.g. dynverse/dyntoy 4-leaf or disconnected simulations) rather than PC-count sensitivity on real scATAC-seq data
- The comparison focuses on scalability/runtime (e.g. the 1.3-million-cell MOCA atlas) rather than fate-detection accuracy as a function of dimensionality

## Inputs

- Buenrostro scATAC-seq hematopoiesis TF-motif z-score matrix (from VIA Datasets folder)
- ChromVAR k-mer accessibility z-score matrix (scATAC_hematopoiesis loader)
- PC projection matrices across PC counts (10–100), stored as adata.obsm['X_pca']
- Cell type labels for computing F1 scores of terminal-state detection

## Outputs

- F1-score table (scatac_buenrostro_f1_table.csv) of fate detection vs number of PCs for VIA and baseline TI methods
- Reproduced trajectory figure (scATAC_BuenrostroPCs_MainFig.png) showing cluster-level trajectory graph overlaid on UMAP
- Pseudotime, terminal states, and single-cell branch probabilities (single_cell_pt_markov, single_cell_bp)
- Gene/motif trend curves along pseudotime for GATA1, CEBPD, IRF8

## How to apply

Load the Buenrostro scATAC-seq hematopoiesis matrices (TF-motif z-scores and ChromVAR k-mer accessibility z-scores) from the VIA Datasets folder via pandas, assemble an AnnData object, and place PC projections in adata.obsm['X_pca'] as required by via.via_wrapper. Run VIA (import pyVIA.core as via) with PARC-based clustering using documented defaults — knn=30, edgepruning_clustering_resolution=0.15, cluster_graph_pruning=0.15, too_big_factor=0.4, pseudotime_threshold_TS=30 — repeating the run across increasing PC counts (e.g. 10–100 PCs) for both preprocessing protocols. Extract pseudotime, terminal states, and single-cell branch probabilities (single_cell_pt_markov, single_cell_bp) to confirm the MPP/LMPP/CMP to CLP/MEP/GMP lineage hierarchy, and compute motif trend curves (GATA1, CEBPD, IRF8) along pseudotime. Finally, tabulate fate-detection F1 scores of VIA versus SlingShot, Palantir, STREAM, and Monocle3 as a function of PC count; the rationale is that VIA's lazy-teleporting random walks behave more predictably under varying input parameter settings than the baselines, which 'exhibit low sensitivity to cell fates'.

## Related tools

- **VIA** (Primary trajectory inference method whose terminal-state detection F1 robustness across PC counts is benchmarked; run via via.via_wrapper on adata.obsm['X_pca']) — https://github.com/ShobiStassen/VIA
- **PARC** (Clustering algorithm underlying VIA's cluster-graph construction; controlled via edgepruning_clustering_resolution and cluster_graph_pruning) — https://github.com/ShobiStassen/VIA
- **UMAP** (Embedding used for single-cell visualization of the VIA cluster-level trajectory graph)
- **scanpy** (Used to assemble the AnnData object holding PC projections in adata.obsm['X_pca'])
- **pandas** (Loads the Buenrostro TF-motif and k-mer z-score matrices from the VIA Datasets folder)
- **SlingShot** (Baseline TI method compared against VIA in F1-vs-PC-count fate detection)
- **Palantir** (Baseline TI method compared against VIA in F1-vs-PC-count fate detection)
- **STREAM** (Baseline TI method compared against VIA in F1-vs-PC-count fate detection)
- **Monocle3** (Baseline TI method compared against VIA in F1-vs-PC-count fate detection)

## Examples

```
import via; via.via_wrapper(adata, true_label, embedding, knn=30, edgepruning_clustering_resolution=0.15, cluster_graph_pruning=0.15, too_big_factor=0.4, pseudotime_threshold_TS=30)
```

## Evaluation signals

- scATAC_BuenrostroPCs_MainFig.png and scatac_buenrostro_f1_table.csv exist and the reproduced figure matches the reference lineage hierarchy
- Reconstructed hierarchy reproduces the MPP/LMPP/CMP to CLP/MEP/GMP lineage structure
- VIA F1 scores for terminal state detection remain stable at PC counts above 50 while SlingShot, Palantir, STREAM, and Monocle3 show performance drops beyond 50 PCs
- Pseudotime and single_cell_pt_markov / single_cell_bp outputs are populated and consistent for both TF-motif and k-mer preprocessing protocols
- GATA1, CEBPD, and IRF8 motif trend curves along pseudotime are coherent with the lineage hierarchy

## Limitations

- Other TI methods show a performance drop beyond 50 PCs on k-mer z-score preprocessed scATAC-seq data, so baseline results are protocol-sensitive and may not generalize to other preprocessing choices
- VIA's behavior depends on PARC clustering tuning parameters (edgepruning_clustering_resolution, cluster_graph_pruning, too_big_factor) that require adjustment per dataset
- Specific parameter ranges used in the paper's stress tests are not fully reported in the article text

## Evidence

- [results] On scATAC-seq hematopoiesis data preprocessed with the k-mer Z Scores protocol, other methods show a performance drop beyond 50 PCs: [evidence span withheld — anchor status: not_found]
- [intro] Baselines lack sensitivity to cell fates: "even the few algorithms which automate cell fate detection (e.g., SlingShot1, Palantir2, STREAM3, and Monocle34) exhibit low sensitivity to cell fates"
- [methods] AnnData with PCA in obsm: "adata = sc.AnnData(data_genes)
	adata.obsm['X_pca'] = data.values"
- [readme] Datasets available in VIA repository: "Dataset are available in the [Datasets folder](https://github.com/ShobiStassen/VIA/tree/master/Datasets)"
