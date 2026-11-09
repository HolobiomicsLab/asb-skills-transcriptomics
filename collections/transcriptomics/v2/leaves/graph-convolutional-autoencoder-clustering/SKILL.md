---
name: graph-convolutional-autoencoder-clustering
description: Use when clustering raw scRNA-seq count matrices (AnnData) across platforms,
  especially with rare cell types/small clusters, or when comparing against deep embedding
  methods (scDeepCluster, DESC, scVI) with ARI/NMI/ASW.
when_to_use_negative:
- The goal is denoising/imputation only rather than clustering — a ZINB-based autoencoder
  such as DCA/scDeepCluster is the appropriate choice instead
- The input is already a reduced embedding or feature table with no raw gene count
  matrix, since the cell-PPMI graph and multinomial decoder require the preprocessed
  expression matrix
- The analysis targets marker gene discovery or pathway interpretation rather than
  cell clustering — downstream annotation steps (p_val_adj ≤ 0.05, avg_logFC ≥ 0.5
  marker filtering) extend beyond this skill
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3316
tools:
- name: scMGCA
  role: 'Core implementation: the graph-embedding autoencoder with cell-PPMI graph,
    two GCN layers, multinomial decoder, and KL self-optimization, installed from
    PyPi'
  repo: https://pypi.org/project/scMGCA
- name: SCANPY
  role: 'Preprocessing: filter genes not expressed in cells, normalize the expression
    matrix, select top d highly variable genes'
- name: TensorFlow
  role: Framework the scMGCA core model is built on for training the graph convolutional
    autoencoder
- name: scikit-learn
  role: K-means/spectral clustering initialization and computing NMI/ARI against ground-truth
    labels
- name: scDeepCluster
  role: Baseline ZINB decoder used for the ablation variant replacing the multinomial-based
    decoder
  repo: https://github.com/ttgump/scDeepCluster
- name: DESC
  role: Baseline deep embedding method for comparison of clustering and batch-effect
    correction (ASW) performance
  repo: https://github.com/eleozzr/desc
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36134-7
    title: Topological identification and interpretation for single-cell gene regulation
      elucidation across multiple platforms using scMGCA
  source_content_hash: sha256:4ae8d567bef694f7e1b7e3b76319e3435c8ee9b0b0b573c2e06ffdd6366702a3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DCA
  - DESC
  - SCANPY
  - Seurat
  - TensorFlow
  - anndata
  - numpy
  - pandas
  - scCAEs
  - scDeepCluster
  - scMGCA
  - scVI
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.65
n_claims: 20
---

# graph-convolutional-autoencoder-clustering

## Summary

scMGCA-style graph convolutional autoencoder clustering: builds a cell-PPMI graph via random surfing on a KNN graph, encodes it with a two-layer GCN autoencoder with a multinomial-based decoder, and self-optimizes clustering with KL divergence on the latent representation.

## When to use

Use when clustering raw scRNA-seq count matrices (AnnData) across platforms, especially with rare cell types/small clusters, or when comparing against deep embedding methods (scDeepCluster, DESC, scVI) with ARI/NMI/ASW.

## When NOT to use

- The goal is denoising/imputation only rather than clustering — a ZINB-based autoencoder such as DCA/scDeepCluster is the appropriate choice instead
- The input is already a reduced embedding or feature table with no raw gene count matrix, since the cell-PPMI graph and multinomial decoder require the preprocessed expression matrix
- The analysis targets marker gene discovery or pathway interpretation rather than cell clustering — downstream annotation steps (p_val_adj ≤ 0.05, avg_logFC ≥ 0.5 marker filtering) extend beyond this skill

## Inputs

- Raw scRNA-seq count matrix loaded as an AnnData object (e.g., Muraro pancreas, QS heart, Qx Limb Muscle, Tabula Muris)
- Ground-truth cell type labels for each cell (used for ARI/NMI evaluation)
- Requirements: Python 3.6.8 environment with numpy, scipy, tensorflow, scikit-learn, pandas, scanpy, anndata

## Outputs

- Low-dimensional latent cell embeddings from the graph convolutional autoencoder (used for ASW and UMAP visualization)
- Cluster assignments per cell after KL self-optimization
- Per-dataset NMI, ARI, and ASW metrics (e.g., assembled into clustering_benchmark_results.csv)

## How to apply

Preprocess the count matrix in SCANPY: filter genes not expressed in cells, normalize the expression matrix, and retain the top d=500 highly variable genes. Build the cell graph by computing a KNN adjacency graph from Euclidean distances between cells, then apply random surfing to compute the cell-PPMI matrix so adjacent cells are aggregated topologically. Train the graph convolutional autoencoder with two GCN layers and a multinomial-based decoder (not ZINB), initializing cluster centers with the best strategy from K-means and spectral clustering. Perform self-optimized clustering by applying Kullback-Leibler (KL) divergence on the latent representations, balancing the autoencoder and clustering losses with weight coefficients γ1, γ2, γ3. For datasets with more than 25,000 cells (e.g., Tabula Muris with 70,118 cells), split the data into multiple learning batches. Evaluate predicted labels against ground-truth cell types with scikit-learn ARI/NMI and compute ASW (average silhouette width) on the learned embeddings.

## Related tools

- **scMGCA** (Core implementation: the graph-embedding autoencoder with cell-PPMI graph, two GCN layers, multinomial decoder, and KL self-optimization, installed from PyPi) — https://pypi.org/project/scMGCA
- **SCANPY** (Preprocessing: filter genes not expressed in cells, normalize the expression matrix, select top d highly variable genes)
- **TensorFlow** (Framework the scMGCA core model is built on for training the graph convolutional autoencoder)
- **scikit-learn** (K-means/spectral clustering initialization and computing NMI/ARI against ground-truth labels)
- **scDeepCluster** (Baseline ZINB decoder used for the ablation variant replacing the multinomial-based decoder) — https://github.com/ttgump/scDeepCluster
- **DESC** (Baseline deep embedding method for comparison of clustering and batch-effect correction (ASW) performance) — https://github.com/eleozzr/desc

## Examples

```
conda create -n scMGCA python=3.6.8; pip install scMGCA
```

## Evaluation signals

- scMGCA achieves the highest mean NMI 0.8304, ARI 0.8278, and ASW 0.5827 across the 20-dataset benchmark, outperforming all 12 baseline methods
- Both ablations (plain KNN graph instead of cell-PPMI; ZINB decoder instead of multinomial) must underperform the full scMGCA on ARI/NMI, per the Source Data ablation table
- On the four-protocol human pancreas data (CEL-seq, CEL-seq2, Fluidigm C1, Smart-seq2), the batch-mixing ASW on scMGCA embeddings should match or exceed DESC, Harmony, MNN, scVI and Scanorama
- Predicted cluster labels compared to true labels with scikit-learn ARI/NMI should be highest on datasets with rare cell types and small clusters (QS heart, Muraro, Qx limb Muscle, Adam)
- Schema check: output CSV contains NMI, ARI, and ASW rows for scMGCA and all 12 baselines across all 20 datasets

## Limitations

- Key hyperparameters are not specified in the paper: number of KNN neighbors k, random-surfing restart probability (1 − α), loss weights γ1/γ2/γ3, maximum iteration count, and the numeric value of d highly variable genes beyond the d=500 used in the benchmark workflow
- No software version numbers reported for Python, TensorFlow, SCANPY or R packages, which complicates exact reproduction
- Datasets with >25,000 cells must be divided into multiple batches for learning, adding runtime/memory complexity on large data such as Tabula Muris (70,118 cells)
- No specific dataset accessions are provided; benchmark data are only described as coming from public repositories, with reproduction relying on the paper's Source Data file

## Evidence

- [results] GCN layers and KL self-optimization: "scMGCA uses two layers of GCN to effectively extract the gene expression information and cell-cell topological structure"
- [results] KL divergence on latent representations: "we perform the Kullback-Leibler (KL) divergence on the latent representations to self-optimize the clustering task"
- [results] Random surfing on KNN graph: "we used a random surfing algorithm based on KNN graph to aggregate neighboring cells"
- [results] Batching large datasets: "scMGCA divides the data into multiple batches for learning when clustering datasets with >25,000 cells"
- [discussion] Highly variable gene selection: "The first step is to filter out genes that are not expressed in cells, reducing the impact of the high dropout rates in scRNA-seq data."
- [readme] Package description: "`scMGCA` is a Python package containing tools for clustering single-cell data based on a graph-embedding autoencoder that simultaneously learns cell–cell topology representation and"
