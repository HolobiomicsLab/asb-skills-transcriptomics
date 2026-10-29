---
name: minibatch-graph-learning-for-million-cell-data
description: Use when clustering a scRNA-seq expression matrix larger than 25,000
  cells (e.g., the 1.3-million-cell mouse brain dataset) where competing graph neural
  network methods fail or exceed a 24-hour budget.
when_to_use_negative:
- Datasets smaller than the 25,000-cell threshold, where scMGCA's batching regime
  is not required
- When the goal is annotating cell types from a small dataset with rare cell types
  or small clusters — standard scMGCA benchmark runs (QS heart, Muraro, Qx limb Muscle,
  Adam) address that rather than scale
- When input is single-cell data requiring spatial integration with in situ RNA patterns,
  where Seurat-style approaches are used instead
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMGCA
  role: Core graph convolutional autoencoder package installed via pip; provides demo_scale.py
    and the batching logic for large datasets
  repo: https://pypi.org/project/scMGCA
- name: TensorFlow
  role: Framework on which the scMGCA core model is built and trained
- name: SCANPY
  role: Scalable toolkit used jointly with anndata for loading and preprocessing the
    large scRNA-seq expression matrix
- name: DEC
  role: Baseline comparison method that, alongside scMGCA and DESC, was among the
    few to complete on the 1.3M-cell dataset
  repo: https://github.com/XifengGuo/DEC-keras
- name: DESC
  role: Baseline deep embedding clustering method that completed on the 1.3M-cell
    mouse brain run
  repo: https://github.com/eleozzr/desc
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-36134-7
    title: Topological identification and interpretation for single-cell gene regulation
      elucidation across multiple platforms using scMGCA
  source_content_hash: sha256:f29fce54ba3256461fe3a85025edb2d853ceb9825f095d1d958fdd5ed0b09312
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DEC
  - DESC
  - SCANPY
  - TensorFlow
  - anndata
  - pandas
  - scMGCA
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# minibatch-graph-learning-for-million-cell-data

## Summary

Scaling the scMGCA graph-embedding autoencoder to very large scRNA-seq datasets (up to 1,306,127 mouse brain cells) by dividing data into multiple batches for learning and constructing a cell-PPMI graph with random surfing, so clustering completes in-memory where most GNN competitors fail.

## When to use

Use when clustering a scRNA-seq expression matrix larger than 25,000 cells (e.g., the 1.3-million-cell mouse brain dataset) where competing graph neural network methods fail or exceed a 24-hour budget.

## When NOT to use

- Datasets smaller than the 25,000-cell threshold, where scMGCA's batching regime is not required
- When the goal is annotating cell types from a small dataset with rare cell types or small clusters — standard scMGCA benchmark runs (QS heart, Muraro, Qx limb Muscle, Adam) address that rather than scale
- When input is single-cell data requiring spatial integration with in situ RNA patterns, where Seurat-style approaches are used instead

## Inputs

- scRNA-seq gene expression count matrix for a very large sample (>25,000 cells; e.g., 1,306,127 mouse brain cells)
- Demo/reference script demo_scale.py from the scMGCA package
- Conda environment spec (Python 3.6.8) with scMGCA pip dependencies

## Outputs

- Per-cell predicted cluster labels (e.g., scale_clustering_labels.csv)
- Time and memory usage profile at scale
- Trained graph-embedding autoencoder latent representations

## How to apply

Set up a Python environment with scMGCA (numpy, scipy, TensorFlow, scikit-learn, pandas, scanpy, anndata), following the package's demo_scale.py workflow. Filter out genes not expressed in cells and keep the top d highly variable genes; because the dataset exceeds 25,000 cells, divide the data into multiple batches for learning as scMGCA prescribes. Construct the cell graph with the random surfing algorithm on a KNN graph to obtain the cell-PPMI matrix, and initialize clustering centers using the best strategy between K-means and spectral clustering. Train the graph convolutional autoencoder (two GCN layers with a multinomial-based decoder) while performing self-optimized clustering via Kullback-Leibler divergence on the latent representations jointly with the autoencoder losses. Write one predicted cluster label per cell (e.g., scale_clustering_labels.csv) and verify runtime and memory stay within a standard computer's RAM, comparable to other deep learning methods aside from cell-graph computation overhead.

## Related tools

- **scMGCA** (Core graph convolutional autoencoder package installed via pip; provides demo_scale.py and the batching logic for large datasets) — https://pypi.org/project/scMGCA
- **TensorFlow** (Framework on which the scMGCA core model is built and trained)
- **SCANPY** (Scalable toolkit used jointly with anndata for loading and preprocessing the large scRNA-seq expression matrix)
- **DEC** (Baseline comparison method that, alongside scMGCA and DESC, was among the few to complete on the 1.3M-cell dataset) — https://github.com/XifengGuo/DEC-keras
- **DESC** (Baseline deep embedding clustering method that completed on the 1.3M-cell mouse brain run) — https://github.com/eleozzr/desc

## Examples

```
pip install scMGCA; python demo_scale.py  # run the 1.3-million-cell mouse brain scalability demo per the package workflow
```

## Evaluation signals

- Run completes within 24 hours and within the in-memory RAM of a standard computer on the 1,306,127-cell mouse brain dataset
- Among compared graph neural networks, scMGCA is the only one that runs successfully to completion
- Every one of the 1.3 million cells receives exactly one predicted cluster label in the output CSV
- Time and memory usage are comparable to other deep learning methods, apart from cell-graph computation overhead
- Datasets with >25,000 cells are actually divided into multiple batches during learning

## Limitations

- Cell-graph computation (random surfing on the KNN graph) adds overhead relative to other deep learning methods
- Most competing graph neural network algorithms fail or do not complete within 24 hours at ~1.3M cells, so direct GNN comparisons at this scale are infeasible
- Number of KNN neighbors k, restart probability α, and the highly variable gene count d are not given numeric values in the article
- Only scMGCA, DEC, DESC, and traditional methods completed on the 1.3M-cell mouse brain dataset, limiting the comparison baseline set

## Evidence

- [results] Batching threshold for large datasets: "scMGCA divides the data into multiple batches for learning when clustering datasets with >25,000 cells"
- [results] Cell graph construction via random surfing: "we used a random surfing algorithm based on KNN graph to aggregate neighboring cells"
- [results] Self-optimized clustering loss: "we perform the Kullback-Leibler (KL) divergence on the latent representations to self-optimize the clustering task"
- [discussion] First preprocessing step: remove unexpressed genes: "The first step is to filter out genes that are not expressed in cells, reducing the impact of the high dropout rates in scRNA-seq data."
- [discussion] Implementation stack: "scMGCA is implemented in Python, and the core model is built on the Tensorflow framework that is publicly available at https://pypi.org/project/scMGCA"
- [readme] Package purpose from README: "scMGCA is a Python package containing tools for clustering single-cell data based on a graph-embedding autoencoder"
