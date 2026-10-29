---
name: graph-embedding-autoencoder-training
description: Use when scRNA-seq count data must be clustered while preserving cell-cell
  topological structure — especially datasets with rare cell types, batch effects,
  or >25,000 cells requiring batched learning.
when_to_use_negative:
- The task is cell-type annotation with known marker genes on pre-clustered data rather
  than unsupervised clustering from raw expression
- Only denoising (not clustering) is needed — DCA with a ZINB loss is the purpose-built
  alternative
- The data are not single-cell expression matrices (the GCN autoencoder models cell-cell
  topology from gene expression, not other feature tables)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: scMGCA
  role: Primary graph convolutional autoencoder package being trained for clustering
  repo: https://pypi.org/project/scMGCA
- name: TensorFlow
  role: Framework on which the scMGCA core model (two GCN layers, multinomial decoder)
    is built
- name: SCANPY
  role: Scalable toolkit used for single-cell preprocessing such as normalization
    and highly variable gene selection
- name: anndata
  role: Data container for loading the scRNA-seq expression matrix
- name: scikit-learn
  role: K-means (and spectral clustering) initialization of clustering centers
- name: scDeepCluster
  role: Baseline deep clustering comparator on the same datasets
  repo: https://github.com/ttgump/scDeepCluster
- name: DESC
  role: Deep embedding baseline comparator for clustering
  repo: https://github.com/eleozzr/desc
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-36134-7
    title: Topological identification and interpretation for single-cell gene regulation
      elucidation across multiple platforms using scMGCA
  source_content_hash: sha256:b1a992dc414a74a449d70373a07b57a9b018dd26b119dd9a56d5f24cce464efa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESC
  - SCANPY
  - TensorFlow
  - anndata
  - pandas
  - scDeepCluster
  - scMGCA
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# graph-embedding-autoencoder-training

## Summary

Train scMGCA's graph convolutional autoencoder (two GCN layers with a multinomial-based decoder) on a normalized scRNA-seq expression matrix plus a cell-PPMI graph, jointly optimizing autoencoder losses with self-optimized KL-divergence clustering to produce latent cell embeddings and cluster assignments.

## When to use

Use when scRNA-seq count data must be clustered while preserving cell-cell topological structure — especially datasets with rare cell types, batch effects, or >25,000 cells requiring batched learning.

## When NOT to use

- The task is cell-type annotation with known marker genes on pre-clustered data rather than unsupervised clustering from raw expression
- Only denoising (not clustering) is needed — DCA with a ZINB loss is the purpose-built alternative
- The data are not single-cell expression matrices (the GCN autoencoder models cell-cell topology from gene expression, not other feature tables)

## Inputs

- scRNA-seq gene expression count matrix (e.g., .h5ad / anndata format)
- Cell graph as a cell-PPMI matrix from random surfing on a KNN graph
- Number of highly variable genes d to retain
- Expected number of clusters for center initialization

## Outputs

- Latent embedded cell representations from the GCN encoder
- Per-cell predicted cluster labels (e.g., scale_clustering_labels.csv)
- Compressed transcriptomic space for downstream functional interpretation
- Clustering metrics (e.g., ASW, ARI) against reference labels

## How to apply

First normalize the gene expression matrix and select the top d highly variable genes, filtering out genes not expressed in cells to mitigate dropout. Build the cell graph with a random surfing algorithm on a KNN graph to obtain the cell-PPMI matrix (positive pointwise mutual information), and initialize clustering centers using the best of K-means or spectral clustering. Train the graph convolutional autoencoder with two GCN layers on TensorFlow, decoding via a multinomial-based decoder, while performing self-optimized clustering by minimizing Kullback-Leibler divergence on the latent representations jointly with the autoencoder losses weighted by coefficients γ1, γ2, γ3. For datasets exceeding 25,000 cells (e.g., the 70,118-cell Tabula Muris or the 1,306,127-cell mouse brain), divide the data into multiple batches for learning. Stop training when the maximum number of iterations is reached, then export latent embeddings and per-cell predicted cluster labels.

## Related tools

- **scMGCA** (Primary graph convolutional autoencoder package being trained for clustering) — https://pypi.org/project/scMGCA
- **TensorFlow** (Framework on which the scMGCA core model (two GCN layers, multinomial decoder) is built)
- **SCANPY** (Scalable toolkit used for single-cell preprocessing such as normalization and highly variable gene selection)
- **anndata** (Data container for loading the scRNA-seq expression matrix)
- **scikit-learn** (K-means (and spectral clustering) initialization of clustering centers)
- **scDeepCluster** (Baseline deep clustering comparator on the same datasets) — https://github.com/ttgump/scDeepCluster
- **DESC** (Deep embedding baseline comparator for clustering) — https://github.com/eleozzr/desc

## Examples

```
pip install scMGCA; python demo_scale.py  # train GCN autoencoder on the 1.3M-cell mouse brain matrix, batching >25,000 cells and writing scale_clustering_labels.csv
```

## Evaluation signals

- Clustering accuracy against reference labels via ARI on benchmark datasets (e.g., the example pancreas data yields ARI ≈ 0.9675 in SHARP's analogous check; scMGCA reports highest ASW on 'QS Trachea' and 'QS Heart')
- Silhouette metric (ASW) on latent embeddings should meet or exceed state-of-the-art deep embedding baselines (DESC, scVI, scCAEs, scDHA, SCA)
- Every cell in the input matrix receives a cluster label (e.g., 1,306,127 rows in scale_clustering_labels.csv for the mouse brain run)
- Rare cell types and small clusters in 'QS heart', 'Muraro', 'Qx limb Muscle', and 'Adam' datasets are recovered rather than merged
- For >25,000-cell datasets, run completes in memory with time comparable to other deep learning methods plus cell-graph computation overhead

## Limitations

- Cell-graph construction with random surfing on the KNN graph adds computational overhead at very large scale (e.g., the 1.3-million-cell mouse brain run)
- The number of KNN neighbors k, restart probability parameter α, highly variable gene count d, loss weights γ1/γ2/γ3, and maximum iteration count are not specified in the article, requiring empirical tuning
- Datasets larger than 25,000 cells must be split into batches, complicating end-to-end reproducibility
- No software version numbers (Python, TensorFlow, SCANPY) or dataset accessions are reported, hindering exact reproduction

## Evidence

- [results] Train graph convolution autoencoder with two GCN layers, multinomial-based decoder, and inner product decoder: "scMGCA uses two layers of GCN to effectively extract the gene expression information and cell-cell topological structure"
- [results] Perform self-optimized clustering via KL divergence jointly with autoencoder losses: "we perform the Kullback-Leibler (KL) divergence on the latent representations to self-optimize the clustering task"
- [results] Divide datasets with >25,000 cells into multiple batches for learning: "scMGCA divides the data into multiple batches for learning when clustering datasets with >25,000 cells"
- [results] Initialize clustering centers with best strategy from K-means and spectral clustering: "we selected the best strategy from K-means and spectral clustering to initialize the clustering centers"
- [discussion] Filter out genes not expressed in cells and keep top d highly variable genes before training: "The first step is to filter out genes that are not expressed in cells, reducing the impact of the high dropout rates in scRNA-seq data."
