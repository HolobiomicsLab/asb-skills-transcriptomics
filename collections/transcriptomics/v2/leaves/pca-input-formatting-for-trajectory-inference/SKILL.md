---
name: pca-input-formatting-for-trajectory-inference
description: Use when preparing PC-reduced single-cell omics matrices (scRNA-seq,
  scATAC-seq chromVAR k-mer or TF-motif z-scores) as input to VIA's via_wrapper, or
  when testing trajectory-inference robustness across PC counts.
when_to_use_negative:
- When the input is already a trajectory with known pseudotime/lineage labels and
  no dimensionality sensitivity study is needed
- When the analysis target is a raw feature matrix that still requires PCA computation
  or a different reduction (this skill assumes PC projections are already available
  to place in obsm['X_pca'])
- For purely topological visualization tasks without trajectory inference (e.g., plain
  UMAP/t-SNE embedding), where VIA's cluster-graph machinery is unnecessary
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: VIA
  role: Trajectory inference method consuming the obsm['X_pca'] projections via via.via_wrapper
    to compute pseudotime, terminal states, and branch probabilities
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Clustering algorithm underlying VIA's cluster-graph construction on the PCA-formatted
    input
  repo: https://github.com/ShobiStassen/VIA
- name: scanpy
  role: Used to assemble the AnnData object holding data and PC projections
- name: UMAP
  role: Embedding computed for single-cell-level visualization of the VIA trajectory
    graph
- name: pandas
  role: Loads the scATAC-seq motif and k-mer z-score matrices from the VIA Datasets
    folder
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:d0e210000ddf68304d1a0390ade6d63ec5e56845ab6a78dc35e68923ac308c6a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShobiStassen/VIA
  tools:
  - PARC
  - UMAP
  - VIA
  - pandas
  - scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# pca-input-formatting-for-trajectory-inference

## Summary

Formats single-cell data (e.g., scATAC-seq motif/k-mer z-score matrices from Buenrostro hematopoiesis) into an AnnData object with PC projections stored in adata.obsm['X_pca'] so VIA's cluster-graph trajectory inference can consume them, and enables robustness sweeps over PC counts.

## When to use

Use when preparing PC-reduced single-cell omics matrices (scRNA-seq, scATAC-seq chromVAR k-mer or TF-motif z-scores) as input to VIA's via_wrapper, or when testing trajectory-inference robustness across PC counts.

## When NOT to use

- When the input is already a trajectory with known pseudotime/lineage labels and no dimensionality sensitivity study is needed
- When the analysis target is a raw feature matrix that still requires PCA computation or a different reduction (this skill assumes PC projections are already available to place in obsm['X_pca'])
- For purely topological visualization tasks without trajectory inference (e.g., plain UMAP/t-SNE embedding), where VIA's cluster-graph machinery is unnecessary

## Inputs

- TF-motif z-score matrix (Buenrostro scATAC-seq hematopoiesis)
- ChromVAR k-mer accessibility z-score matrix (Buenrostro scATAC-seq hematopoiesis)
- AnnData object with adata.obsm['X_pca'] populated with PC projections
- UMAP or PHATE embedding for visualization
- Optional root/start cell index (root_user, e.g., root=[42])

## Outputs

- adata.obsm['X_pca'] PC projection matrix ready for via.via_wrapper
- Per-cell pseudotime (single_cell_pt_markov)
- Automated terminal state predictions
- Single-cell branch probabilities (single_cell_bp)
- Cluster-level trajectory graph overlaid on UMAP embedding (e.g., scATAC_BuenrostroPCs_MainFig.png)
- F1-score table of terminal state detection vs PC count (e.g., scatac_buenrostro_f1_table.csv)

## How to apply

Load the count- or z-score matrices (e.g., Buenrostro scATAC-seq TF-motif z-scores and ChromVAR k-mer accessibility z-scores from the VIA Datasets folder) with pandas, assemble a scanpy AnnData object, and place the principal component projections in adata.obsm['X_pca'] as required by via.via_wrapper. Compute a UMAP (or PHATE) embedding for single-cell-level visualization alongside the PCA. Then run VIA (import pyVIA.core as via) with PARC-based clustering parameters such as knn=30, edgepruning_clustering_resolution=0.15, cluster_graph_pruning=0.15, too_big_factor=0.4, and pseudotime_threshold_TS=30, repeating the run across increasing PC counts (e.g., 10–100 PCs) to probe sensitivity. The rationale is that VIA's F1-scores for terminal state detection remain robust beyond ~50 PCs on k-mer Z Score-preprocessed scATAC-seq data, where SlingShot, Palantir, STREAM, and Monocle3 degrade.

## Related tools

- **VIA** (Trajectory inference method consuming the obsm['X_pca'] projections via via.via_wrapper to compute pseudotime, terminal states, and branch probabilities) — https://github.com/ShobiStassen/VIA
- **PARC** (Clustering algorithm underlying VIA's cluster-graph construction on the PCA-formatted input) — https://github.com/ShobiStassen/VIA
- **scanpy** (Used to assemble the AnnData object holding data and PC projections)
- **UMAP** (Embedding computed for single-cell-level visualization of the VIA trajectory graph)
- **pandas** (Loads the scATAC-seq motif and k-mer z-score matrices from the VIA Datasets folder)

## Examples

```
adata = sc.AnnData(data_genes); adata.obsm['X_pca'] = data.values; v = via.via_wrapper(adata, true_label, embedding=umap_emb, knn=30, ncomps=50, root=[42], edgepruning_clustering_resolution=0.15, cluster_graph_pruning=0.15, too_big_factor=0.4, pseudotime_threshold_TS=30)
```

## Evaluation signals

- adata.obsm['X_pca'] is populated and its shape matches (n_cells, n_pcs) for the requested PC count
- Reproduced scATAC_BuenrostroPCs_MainFig.png exists and matches the reference MPP/LMPP/CMP to CLP/MEP/GMP lineage hierarchy
- scatac_buenrostro_f1_table.csv exists and VIA F1-scores remain stable at PC counts above 50 while SlingShot/Palantir/STREAM/Monocle3 degrade
- Terminal states detected automatically by VIA correspond to expected hematopoietic fates without manual annotation of fate labels
- PARC clustering completes with default tuning (edgepruning_clustering_resolution=0.15, too_big_factor=0.4) without oversized clusters exceeding 40% of cells

## Limitations

- VIA and compared TI methods (SlingShot, Palantir, STREAM, Monocle3) are sensitive to the number of input PCs beyond ~50 on k-mer Z Score-preprocessed scATAC-seq data, though VIA is the most robust
- No software version numbers for VIA or compared methods are stated in the article's discussion, complicating exact reproduction
- No accession identifiers are given for the Buenrostro hematopoiesis dataset in the discussion, so input provenance relies on the VIA Datasets folder

## Evidence

- [methods] Load data as AnnData with PCA stored in adata.obsm['X_pca']: "adata = sc.AnnData(data_genes)
	adata.obsm['X_pca'] = data.values"
- [methods] VIA wrapper call with PCA input parameters: "via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42], dataset='',"
- [methods] Pseudotime threshold default for terminal state detection: [evidence span withheld — anchor status: not_found]
- [results] PC-count robustness finding on scATAC-seq hematopoiesis: [evidence span withheld — anchor status: not_found]
- [readme] README pointer to scATAC-seq hematopoiesis dataset and figure: "Datasets are available in the [Datasets folder](https://github.com/ShobiStassen/VIA/tree/master/Datasets)"
