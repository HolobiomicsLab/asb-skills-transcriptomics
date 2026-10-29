---
name: dimensionality-reduction-baseline-comparison
description: Use when a deep embedding for scRNA-seq cells must be compared to classical
  dimensionality-reduction baselines (PCA, t-SNE, UMAP, full SCANPY/Seurat pipelines)
  using ground-truth cell labels and a quantitative embedding-quality metric such
  as ASW.
when_to_use_negative:
- No ground-truth cell-type labels are available for the dataset — silhouette width
  of true labels cannot be computed
- The goal is comparing clustering accuracy metrics (e.g. ARI on predicted vs true
  labels on 'QS Limb Muscle', 'QS Diaphram', 'Qx Bladder') rather than embedding/dimensionality-reduction
  quality
- The evaluation target is batch effect correction or gene imputation rather than
  dimensionality reduction and visualization performance
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scMGCA
  role: Graph convolutional autoencoder whose latent embedding is the method under
    test; installed from PyPI and built on TensorFlow
  repo: https://pypi.org/project/scMGCA
- name: SCANPY
  role: Computes the PCA, t-SNE and UMAP baseline embeddings on the normalized expression
    matrices and hosts the anndata objects
- name: Seurat
  role: Full R processing pipeline providing one of the paper's baseline embeddings
    for ASW comparison
  repo: https://github.com/satijalab/seurat
- name: scikit-learn
  role: silhouette_score computes per-cell silhouette widths on each embedding using
    the cluster labels, which are then averaged per method and dataset
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-36134-7
    title: Topological identification and interpretation for single-cell gene regulation
      elucidation across multiple platforms using scMGCA
  source_content_hash: sha256:e7391eb0b8047c0319e96d3cca5822e4edfc2e69d2cb0799389d18c2f7bbe9d5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCANPY
  - Seurat
  - anndata
  - numpy
  - pandas
  - scMGCA
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# dimensionality-reduction-baseline-comparison

## Summary

Benchmark the quality of a learned low-dimensional cell embedding (e.g. the scMGCA graph-convolutional autoencoder latent space) against PCA, t-SNE, and UMAP baselines by computing the average silhouette width (ASW) of true cell-type labels, as done for the 'QS Trachea' and 'QS Heart' scRNA-seq datasets in the scMGCA paper.

## When to use

Use when a deep embedding for scRNA-seq cells must be compared to classical dimensionality-reduction baselines (PCA, t-SNE, UMAP, full SCANPY/Seurat pipelines) using ground-truth cell labels and a quantitative embedding-quality metric such as ASW.

## When NOT to use

- No ground-truth cell-type labels are available for the dataset — silhouette width of true labels cannot be computed
- The goal is comparing clustering accuracy metrics (e.g. ARI on predicted vs true labels on 'QS Limb Muscle', 'QS Diaphram', 'Qx Bladder') rather than embedding/dimensionality-reduction quality
- The evaluation target is batch effect correction or gene imputation rather than dimensionality reduction and visualization performance

## Inputs

- Normalized scRNA-seq gene expression matrices for benchmark datasets (e.g. 'QS Trachea', 'QS Heart'), loaded as anndata objects
- Ground-truth cell-type label vectors per dataset
- Latent embedding matrices from scMGCA and baseline embeddings from PCA, t-SNE, UMAP, SCANPY and Seurat pipelines

## Outputs

- ASW comparison table (rows = methods, columns = datasets), saved as asw_comparison.csv
- ASW benchmark bar-chart figure mirroring the paper's figure, saved as asw_comparison.png

## How to apply

Load the normalized scRNA-seq expression matrices and ground-truth cell-type labels with SCANPY/anndata, then run the scMGCA graph convolutional autoencoder (TensorFlow backend) on each dataset to obtain latent embedded representations and cluster assignments. Compute baseline embeddings per dataset with PCA, t-SNE, and UMAP via SCANPY plus the full SCANPY and Seurat processing pipelines used in the paper. For every embedding, compute each cell's silhouette width with scikit-learn's silhouette_score using the cluster (cell-type) labels, then average per method and per dataset. Assemble the ASW values into a comparison table (rows = methods, columns = datasets) and a bar-chart figure mirroring the paper's ASW benchmark. The paper's own acceptance criterion is that scMGCA attains the highest ASW on 'QS Trachea' and 'QS Heart'; if it does not, flag the reproduction as failed.

## Related tools

- **scMGCA** (Graph convolutional autoencoder whose latent embedding is the method under test; installed from PyPI and built on TensorFlow) — https://pypi.org/project/scMGCA
- **SCANPY** (Computes the PCA, t-SNE and UMAP baseline embeddings on the normalized expression matrices and hosts the anndata objects)
- **Seurat** (Full R processing pipeline providing one of the paper's baseline embeddings for ASW comparison) — https://github.com/satijalab/seurat
- **scikit-learn** (silhouette_score computes per-cell silhouette widths on each embedding using the cluster labels, which are then averaged per method and dataset)

## Examples

```
from sklearn.metrics import silhouette_score; asw = silhouette_score(embedding, cell_type_labels)
```

## Evaluation signals

- scMGCA attains the highest ASW on the 'QS Trachea' and 'QS Heart' datasets; otherwise flag the reproduction as failed
- Comparison table has rows = methods (scMGCA, PCA, t-SNE, UMAP, SCANPY, Seurat) and columns = datasets, with one averaged ASW per cell
- Silhouette widths are computed per cell with scikit-learn's silhouette_score against ground-truth cell labels, then averaged per method and per dataset
- Bar-chart figure mirrors the layout of the paper's ASW benchmark figure
- Outputs exist at the expected paths asw_comparison.csv and asw_comparison.png

## Limitations

- No software version numbers reported (Python, TensorFlow, SCANPY, Seurat all unstated), so exact numerical reproduction is not guaranteed
- No specific dataset accessions or repository identifiers are provided in the paper; benchmark data are only described as coming from public repositories, and reproduction relies on the Source Data file
- ASW depends on ground-truth label quality and, for methods that also cluster (like scMGCA), reflects the self-optimized clustering initialized from K-means/spectral clustering

## Evidence

- [results] ASW validation criterion: [evidence span withheld — anchor status: not_found]
- [results] Preprocessing input for embeddings: "We first normalized the scRNA-seq gene expression matrix and selected the highly variable genes"
- [discussion] Reproduction source: "To reproduce the results, please find the Source Data file we provided."
- [readme] scMGCA package description: "scMGCA is a Python package containing tools for clustering single-cell data based on a graph-embedding autoencoder"
