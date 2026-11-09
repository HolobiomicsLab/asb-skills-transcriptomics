---
name: kmeans-clustering-ari-evaluation
description: Use when embeddings from a multimodal single-cell omics pipeline (e.
when_to_use_negative:
- No reference cell-type labels are available for the dataset — ARI cannot be computed
  and an internal clustering metric must be used instead
- Input is raw single-cell count matrices rather than generated embeddings — this
  skill evaluates consensus clusterings of SnapCCESS embeddings, not raw-data clustering
- The goal is supervised cell-type classification or label transfer rather than unsupervised
  consensus clustering evaluation
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SnapCCESS
  role: Generates the snapshot variational-autoencoder embeddings that are clustered
    with k-means and scored by ARI
  repo: https://github.com/PYangLab/SnapCCESS
- name: snapccess
  role: Python package installed via pip that produces embeddings from multimodal
    (e.g., TEA-seq, CITE-seq) data loaders
  repo: https://github.com/yulijia/SnapCCESS/tree/main/snapccess-py
- name: Python
  role: Implementation environment for running k-means, computing ARI, and writing
    the comparison CSVs and figures
- name: R
  role: Alternative wrapper environment; SnapCCESS R package calls the Python implementation
    via reticulate
  repo: https://github.com/PYangLab/SnapCCESS
- name: remotes
  role: Used to install the SnapCCESS R wrapper from GitHub
  repo: https://github.com/PYangLab/SnapCCESS
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad382
    title: Ensemble deep learning of embeddings for clustering multimodal single-cell
      omics data
  source_content_hash: sha256:5a9f0714c389970cfad5583b2c50bc60948d4f32f74eb49ba6738a23618a50c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PYangLab/SnapCCESS
  tools:
  - Python
  - R
  - SnapCCESS
  - remotes
  - snapccess
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# kmeans-clustering-ari-evaluation

## Summary

Evaluates consensus cell clustering quality by running k-means on generated embeddings (e.g., SnapCCESS variational-autoencoder snapshots) and scoring the resulting clusters against reference cell-type labels with the adjusted Rand index (ARI). Used in the SnapCCESS paper to quantify sensitivity to hyperparameters and ensemble size across major and minor cell-type partitions.

## When to use

Use when embeddings from a multimodal single-cell omics pipeline (e.g., SnapCCESS snapshot VAE embeddings) must be converted into clusters and quantitatively compared against reference cell-type labels, or when comparing clustering performance across hyperparameter settings such as learning rate or ensemble size.

## When NOT to use

- No reference cell-type labels are available for the dataset — ARI cannot be computed and an internal clustering metric must be used instead
- Input is raw single-cell count matrices rather than generated embeddings — this skill evaluates consensus clusterings of SnapCCESS embeddings, not raw-data clustering
- The goal is supervised cell-type classification or label transfer rather than unsupervised consensus clustering evaluation

## Inputs

- SnapCCESS multimodality embeddings (snapshot variational-autoencoder embeddings from CITE-seq RNA + ADT modalities, or TEA-seq RNA + protein modalities)
- Reference cell-type annotations, split into major and minor cell-type label sets
- Clustering/evaluation configuration: number of k-means clusters, ensemble size (1 or 50), epoch (1), initial learning rate (e.g., 0.02 vs 0.2)

## Outputs

- Consensus clustering assignments per cell from k-means on embeddings
- ARI values per condition against major and minor cell-type labels
- Tabulated comparisons such as ari_comparison_s2a.csv (learning-rate sensitivity) and ari_ensemble_comparison.csv (ensemble 50 vs base 1)

## How to apply

Generate embeddings with SnapCCESS (epoch=1; either ensemble size 1 as base or 50 snapshots for the ensemble configuration), then cluster the embeddings with k-means and compute the adjusted Rand index (ARI) of the consensus clustering against reference cell-type labels. For sensitivity analyses, hold all settings fixed except the factor under test — e.g., run the snapshot variational-autoencoder twice with initial learning rates 0.02 and 0.2 — and tabulate ARI per condition (as in Supplementary Fig. S2a for the Swanson TEA-seq / GSE158013 data). For ensemble evaluation, split cell-type annotations into major and minor label sets and compute ARI against both partitions per dataset (e.g., the Ramaswamy CITE-seq data), writing comparisons to ari_comparison_s2a.csv and ari_ensemble_comparison.csv. SnapCCESS is designed so that its snapshot embeddings 'can be coupled with various clustering algorithms for generating consensus clustering of cells', so k-means here serves as the coupled consensus-clustering step and ARI as the agreement metric.

## Related tools

- **SnapCCESS** (Generates the snapshot variational-autoencoder embeddings that are clustered with k-means and scored by ARI) — https://github.com/PYangLab/SnapCCESS
- **snapccess** (Python package installed via pip that produces embeddings from multimodal (e.g., TEA-seq, CITE-seq) data loaders) — https://github.com/yulijia/SnapCCESS/tree/main/snapccess-py
- **Python** (Implementation environment for running k-means, computing ARI, and writing the comparison CSVs and figures)
- **R** (Alternative wrapper environment; SnapCCESS R package calls the Python implementation via reticulate) — https://github.com/PYangLab/SnapCCESS
- **remotes** (Used to install the SnapCCESS R wrapper from GitHub) — https://github.com/PYangLab/SnapCCESS

## Examples

```
from sklearn.cluster import KMeans; from sklearn.metrics import adjusted_rand_score; ari = adjusted_rand_score(celltype_labels, KMeans(n_clusters=k, random_state=0).fit_predict(embeddings))
```

## Evaluation signals

- ARI comparison across initial learning rates 0.02 and 0.2 on the Swanson TEA-seq data reproduces the sensitivity trend of Supplementary Fig. S2a (default 0.02 high, 0.2 significantly reduced)
- Ensemble size 50 (epoch=1) yields ARI equal to or better than base size 1 against minor cell-type labels across all six datasets, consistent with published values
- ARI table splits per dataset into major and minor cell-type label sets, with MAIT-NKT highlighted in the Ramaswamy CITE-seq embedding as in Figure 3 and Supplementary Fig. S3
- Output CSVs (ari_comparison_s2a.csv, ari_ensemble_comparison.csv) contain one ARI value per condition/dataset with all other settings held fixed

## Limitations

- The SnapCCESS tutorial does not recommend best parameters; hyperparameters must be taken from the paper for the published datasets
- ARI values depend on the coupled clustering algorithm; the paper's numbers assume k-means-based consensus clustering of snapshot embeddings
- GPU acceleration is optional and depends on the local PyTorch/CUDA or ROCm installation

## Evidence

- [other] Cluster each set of embeddings with k-means and compute the adjusted Rand index (ARI) of the resulting consensus clustering against the reference cell-type labels.: [evidence span withheld — anchor status: not_found]
- [intro] SnapCCESS can be coupled with various clustering algorithms for generating consensus clustering of cells.: "SnapCCESS can be coupled with various clustering algorithms for generating consensus clustering of cells."
- [readme] creating snapshots of multimodal variational autoencoder embeddings, which can then be used with downstream clustering methods: [evidence span withheld — anchor status: not_found]
- [other] SnapCCESS (epoch = 1) with an ensemble of 50 achieved higher clustering performance than no ensemble (base of 1): [evidence span withheld — anchor status: not_found]
- [readme] This tutorial only explains how to use this package; it doesn't recommend the best parameters for your datasets.: "This tutorial only explains how to use this package; it doesn't recommend the best parameters for your datasets."
