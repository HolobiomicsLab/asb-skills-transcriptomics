---
name: single-cell-embedding-silhouette-evaluation
description: Use when comparing dimensionality-reduction/visualization quality of
  scRNA-seq embeddings (e.g., scMGCA latent space vs PCA, t-SNE, UMAP) on datasets
  with ground-truth cell-type labels, quantified by average silhouette width (ASW).
when_to_use_negative:
- The dataset has no ground-truth cell-type labels — silhouette width of true labels
  cannot be computed; use internal clustering metrics (ARI/NMI) instead.
- The goal is clustering accuracy rather than embedding compactness/separability —
  the paper evaluates clustering via other comparisons (e.g., ARI on 'QS Limb Muscle',
  'QS Diaphram', 'Qx Bladder'), not ASW.
- You need to reproduce batch-effect correction or PDAC cell-type annotation results
  — those are separate analyses in the paper, not silhouette-based embedding evaluation.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: scMGCA
  role: Produces the graph-embedding autoencoder latent representations whose ASW
    is benchmarked; installed from PyPI, built on TensorFlow
  repo: https://pypi.org/project/scMGCA
- name: scMGCA (Zenodo archive)
  role: Archived source code/DOI for reproducing the scMGCA embeddings
  repo: https://doi.org/10.5281/zenodo.7515775
- name: SCANPY
  role: Loads the normalized expression matrices and labels (with anndata) and computes
    PCA, t-SNE, and UMAP baseline embeddings
- name: Seurat
  role: Full Seurat processing pipeline used as a comparison baseline producing embeddings
    for ASW evaluation
  repo: https://github.com/satijalab/seurat
- name: scikit-learn
  role: silhouette_score function used to compute per-cell silhouette widths on cluster
    labels
- name: TensorFlow
  role: Backend framework on which the scMGCA graph convolutional autoencoder is trained
- name: anndata
  role: Data container format for loading normalized scRNA-seq matrices and ground-truth
    labels
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-36134-7
    title: Topological identification and interpretation for single-cell gene regulation
      elucidation across multiple platforms using scMGCA
  source_content_hash: sha256:ee1e955344f43bbd712ea34927aeaa2fbe293a829b22e9513968355d692170b3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCANPY
  - Seurat
  - TensorFlow
  - anndata
  - numpy
  - pandas
  - scMGCA
  - scMGCA (Zenodo archive)
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# single-cell-embedding-silhouette-evaluation

## Summary

Evaluates the quality of low-dimensional single-cell embeddings (scMGCA graph-autoencoder latent space vs PCA, t-SNE, UMAP, and full SCANPY/Seurat pipelines) by computing the average silhouette width (ASW) of true cell-type labels with scikit-learn, benchmarking dimensionality-reduction performance on scRNA-seq datasets.

## When to use

Use when comparing dimensionality-reduction/visualization quality of scRNA-seq embeddings (e.g., scMGCA latent space vs PCA, t-SNE, UMAP) on datasets with ground-truth cell-type labels, quantified by average silhouette width (ASW).

## When NOT to use

- The dataset has no ground-truth cell-type labels — silhouette width of true labels cannot be computed; use internal clustering metrics (ARI/NMI) instead.
- The goal is clustering accuracy rather than embedding compactness/separability — the paper evaluates clustering via other comparisons (e.g., ARI on 'QS Limb Muscle', 'QS Diaphram', 'Qx Bladder'), not ASW.
- You need to reproduce batch-effect correction or PDAC cell-type annotation results — those are separate analyses in the paper, not silhouette-based embedding evaluation.

## Inputs

- Normalized scRNA-seq gene expression matrix (anndata/h5ad) for 'QS Trachea' and 'QS Heart' datasets
- Ground-truth cell-type label vector per cell
- Latent embedded representation from scMGCA graph convolutional autoencoder
- Baseline low-dimensional embeddings from PCA, t-SNE, UMAP (SCANPY) and SCANPY/Seurat pipelines

## Outputs

- asw_comparison.csv — table of average silhouette width per method (rows) and per dataset (columns)
- asw_comparison.png — bar-chart figure mirroring the paper's ASW benchmark figure
- Per-method average silhouette width values on each dataset

## How to apply

Load the normalized 'QS Trachea' and 'QS Heart' scRNA-seq expression matrices and ground-truth cell-type labels with SCANPY/anndata. Run the scMGCA graph convolutional autoencoder (TensorFlow backend, two GCN layers, multinomial-based decoder, KL-divergence self-optimized clustering) to obtain latent embedded representations and cluster assignments. Generate baseline embeddings per dataset: PCA, t-SNE, and UMAP via SCANPY, plus the full SCANPY and Seurat processing pipelines as the paper's comparison baselines. For every embedding, compute each cell's silhouette width with scikit-learn's silhouette_score using the cluster labels, then average per method and per dataset. Assemble the ASW values into a comparison table (rows = methods, columns = datasets) and a bar-chart figure mirroring the paper's ASW benchmark figure; treat reproduction as failed if scMGCA does not attain the highest ASW on 'QS Trachea' and 'QS Heart'.

## Related tools

- **scMGCA** (Produces the graph-embedding autoencoder latent representations whose ASW is benchmarked; installed from PyPI, built on TensorFlow) — https://pypi.org/project/scMGCA
- **scMGCA (Zenodo archive)** (Archived source code/DOI for reproducing the scMGCA embeddings) — https://doi.org/10.5281/zenodo.7515775
- **SCANPY** (Loads the normalized expression matrices and labels (with anndata) and computes PCA, t-SNE, and UMAP baseline embeddings)
- **Seurat** (Full Seurat processing pipeline used as a comparison baseline producing embeddings for ASW evaluation) — https://github.com/satijalab/seurat
- **scikit-learn** (silhouette_score function used to compute per-cell silhouette widths on cluster labels)
- **TensorFlow** (Backend framework on which the scMGCA graph convolutional autoencoder is trained)
- **anndata** (Data container format for loading normalized scRNA-seq matrices and ground-truth labels)

## Examples

```
from sklearn.metrics import silhouette_score; asw = silhouette_score(embedding, labels)
```

## Evaluation signals

- scMGCA attains the highest ASW among all compared methods (PCA, t-SNE, UMAP, SCANPY, Seurat) on both 'QS Trachea' and 'QS Heart'; otherwise flag the reproduction as failed.
- Every average silhouette width falls in the valid range [-1, 1]; silhouette widths are computed on the embedded representation (not raw expression) against ground-truth labels.
- The asw_comparison.csv table has one row per method and one column per dataset ('QS Trachea', 'QS Heart') with no missing method/dataset cells.
- The bar-chart figure (asw_comparison.png) shows per-method ASW ordering mirroring the paper's ASW benchmark figure.
- Per-cell silhouette widths from scikit-learn silhouette_score are averaged per method per dataset, matching the paper's ASW aggregation.

## Limitations

- The paper does not report software version numbers (Python, TensorFlow, SCANPY, Seurat) or key hyperparameters (number of KNN neighbors k, restart probability α, number of highly variable genes d, loss weights γ1/γ2/γ3, maximum iterations), so exact reproduction of embeddings is under-specified.
- Benchmark samples 'were obtained from public repositories' with no specific dataset accessions provided; the paper points to the Source Data file for reproduction without a DOI/URL.
- scMGCA divides data into multiple batches for learning when datasets exceed 25,000 cells, and silhouette computation is expensive at scale (e.g., 'Tabula Muris' with 70,118 cells), so large datasets require batching/sampling considerations.

## Evidence

- [results] scMGCA has the highest ASW on 'QS Trachea' and 'QS Heart' datasets: [evidence span withheld — anchor status: not_found]
- [results] we perform the Kullback-Leibler (KL) divergence on the latent representations to self-optimize the clustering task: "we perform the Kullback-Leibler (KL) divergence on the latent representations to self-optimize the clustering task"
- [discussion] scMGCA is implemented in Python, and the core model is built on the Tensorflow framework that is publicly available at https://pypi.org/project/scMGCA: "scMGCA is implemented in Python, and the core model is built on the Tensorflow framework that is publicly available at https://pypi.org/project/scMGCA"
- [readme] scMGCA is a Python package containing tools for clustering single-cell data based on a graph-embedding autoencoder: "scMGCA is a Python package containing tools for clustering single-cell data based on a graph-embedding autoencoder"
