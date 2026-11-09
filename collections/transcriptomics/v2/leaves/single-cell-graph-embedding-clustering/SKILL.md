---
name: single-cell-graph-embedding-clustering
description: Use when clustering raw or preprocessed scRNA-seq count matrices where
  preserving cell-cell topological relationships matters, e.g. datasets with rare
  cell types, small clusters, or >25,000 cells needing batched learning.
when_to_use_negative:
- Input is already a cell-feature embedding or cluster-label table — scMGCA requires
  the gene expression count matrix to build the cell-PPMI graph
- The goal is gene-level pathway enrichment without first obtaining a latent representation
  — use a standard GO/KEGG over-representation workflow instead
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scMGCA
  role: Core graph convolutional autoencoder that learns cell-cell topology representation
    and cluster assignments simultaneously
  repo: https://pypi.org/project/scMGCA
- name: SCANPY
  role: 'Preprocessing: filtering all-zero genes, normalizing the expression matrix,
    selecting highly variable genes'
- name: TensorFlow
  role: Framework on which the scMGCA core model is built and trained
- name: scikit-learn
  role: Provides K-means initialization of clustering centers, compared against spectral
    clustering
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-36134-7
    title: Topological identification and interpretation for single-cell gene regulation
      elucidation across multiple platforms using scMGCA
  source_content_hash: sha256:912c3e204cd4f2f0383f19d3c4adaa779fff0c593029989de5543f4ddf622810
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCANPY
  - TensorFlow
  - scMGCA
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# single-cell-graph-embedding-clustering

## Summary

Cluster scRNA-seq cells by training scMGCA, a graph-embedding autoencoder that simultaneously learns cell-cell topology via a cell-PPMI graph and cluster assignments through a two-layer GCN encoder with a multinomial-based decoder and KL-divergence self-optimized clustering.

## When to use

Use when clustering raw or preprocessed scRNA-seq count matrices where preserving cell-cell topological relationships matters, e.g. datasets with rare cell types, small clusters, or >25,000 cells needing batched learning.

## When NOT to use

- Input is already a cell-feature embedding or cluster-label table — scMGCA requires the gene expression count matrix to build the cell-PPMI graph
- The goal is gene-level pathway enrichment without first obtaining a latent representation — use a standard GO/KEGG over-representation workflow instead

## Inputs

- Raw scRNA-seq count matrix (e.g. 'Qx Limb Muscle', 'QS heart', 'Muraro', 'Adam' datasets from public repositories)
- Number of expected clusters / number of neighbors k for the KNN cell graph

## Outputs

- Latent embedding of cells in compressed transcriptomic space
- Cluster assignments per cell
- GCN weight matrices W1/W2 usable for downstream gene-ranking and functional interpretation

## How to apply

Preprocess in SCANPY by filtering out genes not expressed in cells, normalizing the gene expression matrix, and selecting highly variable genes. Build a cell graph using a random surfing algorithm on a KNN graph to compute the cell-PPMI matrix, then train scMGCA (TensorFlow) with two GCN layers, a multinomial-based decoder, and an inner product decoder. Initialize clustering centers with the better strategy from K-means (scikit-learn) and spectral clustering, and jointly optimize autoencoder losses with KL divergence on latent representations to self-optimize clustering. For datasets with >25,000 cells, split into multiple batches for learning; evaluate clustering against ground-truth labels where available.

## Related tools

- **scMGCA** (Core graph convolutional autoencoder that learns cell-cell topology representation and cluster assignments simultaneously) — https://pypi.org/project/scMGCA
- **SCANPY** (Preprocessing: filtering all-zero genes, normalizing the expression matrix, selecting highly variable genes)
- **TensorFlow** (Framework on which the scMGCA core model is built and trained)
- **scikit-learn** (Provides K-means initialization of clustering centers, compared against spectral clustering)

## Examples

```
pip install scMGCA; python -c "import scMGCA; scMGCA.run(data, n_clusters=k)"
```

## Evaluation signals

- Cluster assignments recovered on benchmark datasets with rare cell types and small clusters ('QS heart', 'Muraro', 'Qx limb Muscle', 'Adam') match published clustering performance
- Silhouette (ASW) score on 'QS Trachea' and 'QS Heart' datasets should be the highest among compared deep embedding methods
- Batched learning is automatically triggered for datasets with >25,000 cells (e.g. 'Tabula Muris' with 70,118 cells)
- Latent embedding must jointly preserve cell-cell topology (from the cell-PPMI graph) and enable downstream interpretation of the compressed transcription space

## Limitations

- No software version numbers reported (Python, TensorFlow, SCANPY all unstated versions), so exact reproducibility of the environment is not guaranteed
- Number of KNN neighbors k and restart probability α for the random surfing algorithm are not specified in the article
- Maximum iteration count for the stopping condition is only specified as 'the maximum number of iterations is achieved' with no numeric value
- Loss weight coefficients γ1, γ2, γ3 for the total objective function are not provided numerically

## Evidence

- [abstract] we develop a deep graph learning method, scMGCA, for single-cell data analysis: "we develop a deep graph learning method, scMGCA, for single-cell data analysis"
- [intro] we propose a graph convolutional autoencoder framework, called scMGCA, to analyze scRNA-seq datasets: "we propose a graph convolutional autoencoder framework, called scMGCA, to analyze scRNA-seq datasets"
- [results] we used a random surfing algorithm based on KNN graph to aggregate neighboring cells: "we used a random surfing algorithm based on KNN graph to aggregate neighboring cells"
- [results] we perform the Kullback-Leibler (KL) divergence on the latent representations to self-optimize the clustering task: "we perform the Kullback-Leibler (KL) divergence on the latent representations to self-optimize the clustering task"
- [discussion] the core model is built on the Tensorflow framework that is publicly available at https://pypi.org/project/scMGCA: "the core model is built on the Tensorflow framework that is publicly available at https://pypi.org/project/scMGCA"
- [readme] scMGCA is a Python package containing tools for clustering single-cell data based on a graph-embedding autoencoder: "`scMGCA` is a Python package containing tools for clustering single-cell data based on a graph-embedding autoencoder"
