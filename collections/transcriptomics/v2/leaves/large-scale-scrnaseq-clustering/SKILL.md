---
name: large-scale-scrnaseq-clustering
description: Use when an scRNA-seq expression matrix exceeds the memory/runtime limits
  of standard deep-learning clustering (tens of thousands to >1 million cells, e.g.,
  mouse brain 1,306,127 cells) and graph-neural-network competitors crash or exceed
  a 24-hour budget.
when_to_use_negative:
- Datasets under the 25,000-cell batching threshold where standard scMGCA (or scDeepCluster/DESC)
  runs without the batching regime — the large-scale batching pipeline is unnecessary
  overhead
- Input is spatial transcriptomics, bulk RNA-seq, or an already-annotated cell-type
  table rather than a raw/dimensioned scRNA-seq count expression matrix
- A supervised or marker-based cell-type annotation task is required (e.g., SingleR-style
  reference annotation) rather than unsupervised clustering
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMGCA
  role: 'Primary method: graph convolutional autoencoder with cell-PPMI graph construction,
    >25,000-cell batching, and KL self-optimized clustering; installed from PyPI'
  repo: https://pypi.org/project/scMGCA
- name: TensorFlow
  role: Backend framework on which the scMGCA core model is built
- name: SCANPY
  role: Scalable toolkit used with anndata to load and preprocess the single-cell
    gene expression data
- name: anndata
  role: Data container for the large scRNA-seq expression matrix
- name: scikit-learn
  role: Provides K-means initialization strategy and clustering utilities
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-36134-7
    title: Topological identification and interpretation for single-cell gene regulation
      elucidation across multiple platforms using scMGCA
  source_content_hash: sha256:bac4da98b552e8851f6310cc50756ccbd37dd4f4c3e5bdbc641957386d01a1ea
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCANPY
  - TensorFlow
  - anndata
  - pandas
  - scMGCA
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# large-scale-scrnaseq-clustering

## Summary

Scalable single-cell RNA-seq clustering with scMGCA, a graph convolutional autoencoder that batches datasets larger than 25,000 cells and can cluster >1 million cells (e.g., the 1,306,127-cell mouse brain dataset) on a standard in-memory computer, where competing graph neural networks fail or do not finish within 24 hours.

## When to use

Use when an scRNA-seq expression matrix exceeds the memory/runtime limits of standard deep-learning clustering (tens of thousands to >1 million cells, e.g., mouse brain 1,306,127 cells) and graph-neural-network competitors crash or exceed a 24-hour budget.

## When NOT to use

- Datasets under the 25,000-cell batching threshold where standard scMGCA (or scDeepCluster/DESC) runs without the batching regime — the large-scale batching pipeline is unnecessary overhead
- Input is spatial transcriptomics, bulk RNA-seq, or an already-annotated cell-type table rather than a raw/dimensioned scRNA-seq count expression matrix
- A supervised or marker-based cell-type annotation task is required (e.g., SingleR-style reference annotation) rather than unsupervised clustering

## Inputs

- scRNA-seq gene expression matrix (mouse brain, 1,306,127 cells) loadable via scanpy/anndata
- pip-installable scMGCA package with requirements: numpy, scipy, tensorflow, scikit-learn, pandas, scanpy, anndata
- Python 3.6.8 conda environment with TensorFlow

## Outputs

- scale_clustering_labels.csv containing one predicted cluster label for each of the 1,306,127 cells
- Wall-clock time and peak-memory profile of the large-scale run
- Latent graph-embedded representations from the graph convolutional autoencoder

## How to apply

Set up a conda environment with Python 3.6.8 and install scMGCA from PyPI with its requirements (numpy, scipy, tensorflow, scikit-learn, pandas, scanpy, anndata). Load the mouse brain scRNA-seq expression matrix following demo_scale.py, filter out genes not expressed in cells, and keep the top d highly variable genes. Because the dataset exceeds 25,000 cells, divide the data into multiple batches for learning as scMGCA prescribes for large datasets. Construct the cell graph with the random surfing algorithm on a KNN graph to obtain the cell-PPMI matrix, initialize clustering centers with the best of K-means vs spectral clustering, then train the two-GCN-layer autoencoder with a multinomial-based decoder while self-optimizing clustering via Kullback-Leibler divergence on the latent representations. Write one predicted cluster label per cell to scale_clustering_labels.csv and confirm the run completes within the in-memory RAM of a standard computer.

## Related tools

- **scMGCA** (Primary method: graph convolutional autoencoder with cell-PPMI graph construction, >25,000-cell batching, and KL self-optimized clustering; installed from PyPI) — https://pypi.org/project/scMGCA
- **TensorFlow** (Backend framework on which the scMGCA core model is built)
- **SCANPY** (Scalable toolkit used with anndata to load and preprocess the single-cell gene expression data)
- **anndata** (Data container for the large scRNA-seq expression matrix)
- **scikit-learn** (Provides K-means initialization strategy and clustering utilities)

## Examples

```
pip install scMGCA && python demo_scale.py  # Python 3.6.8 env; batch >25,000 cells; writes scale_clustering_labels.csv
```

## Evaluation signals

- Every one of the 1,306,127 cells receives a predicted cluster label in scale_clustering_labels.csv (no dropped cells)
- The run completes to termination rather than crashing, and finishes within the in-memory RAM of a standard computer as stated in the package requirements
- Among graph neural networks compared, scMGCA is the only one that runs successfully on the 1.3-million-cell dataset (competitors fail or do not complete within 24 hours)
- Time and memory usage are comparable to other deep learning methods apart from cell-graph computation overhead
- Pipeline follows demo_scale.py steps: genes not expressed in cells are filtered out and the top d highly variable genes are retained before graph construction

## Limitations

- Datasets exceeding 25,000 cells must be divided into multiple batches for learning, adding pipeline complexity and batch bookkeeping
- Cell-graph computation (random surfing on the KNN graph to build the cell-PPMI matrix) introduces overhead beyond typical deep-learning time/memory
- Most competing clustering algorithms fail or do not complete within 24 hours at this scale, so direct runtime comparisons against all GNN baselines are not obtainable
- Key hyperparameters (number of KNN neighbors k, restart probability α, number of highly variable genes d, loss weights γ1–γ3, maximum iterations) are not given numeric values in the paper

## Evidence

- [results] batching for >25k cells: "scMGCA divides the data into multiple batches for learning when clustering datasets with >25,000 cells"
- [discussion] gene filtering and HVG selection: "The first step is to filter out genes that are not expressed in cells, reducing the impact of the high dropout rates in scRNA-seq data."
- [discussion] HVG retention: "the top d highly variable genes with high-level information are used in the study"
- [results] cell graph construction: "we used a random surfing algorithm based on KNN graph to aggregate neighboring cells"
- [readme] package description from README: "scMGCA is a Python package containing tools for clustering single-cell data based on a graph-embedding autoencoder"
- [discussion] TensorFlow backend: "the core model is built on the Tensorflow framework that is publicly available at https://pypi.org/project/scMGCA"
