---
name: knn-cell-graph-construction
description: 'Use when preparing scRNA-seq count data (e.g. simulated scenario4 matrices,
  HRA000084 HSPCs, GSE84133 pancreas) as input to scGAE: a cell adjacency matrix must
  be built via KNN before training the graph autoencoder.'
when_to_use_negative:
- Input is raw, unnormalized scRNA-seq counts — scGAE requires preprocessed data as
  input (preprocess with scTransform/Seurat or scanpy first)
- The downstream method ignores cell–cell relationships (e.g. methods that 'only embedded
  the distinct cell features while ignoring the cell–cell relationships') — the graph
  is built specifically for graph-autoencoder-based embedding
- A 2D visualization-only pipeline without graph structure is desired (the paper notes
  the graph autoencoder cannot directly embed into 2D; an intermediate dimension plus
  tSNE is used instead)
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scGAE
  role: The pipeline whose cell graph is built via KNN (get_adj) and consumed by the
    multitask-oriented graph autoencoder implemented with Spektral
  repo: github.com/ZixiangLuo1161/scGAE
- name: t-SNE
  role: Used downstream of the KNN-graph-based embedding to visualize the embedded
    data in two dimensions
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_008
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:082858f977bf75c3da1a5957c07d602793bdbe1af1bb1ecd3da556a31f60291c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scGAE
  - t-SNE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# knn-cell-graph-construction

## Summary

Constructs a cell-to-cell KNN graph (default K=35, adjusted per dataset) from a preprocessed scRNA-seq expression matrix, producing the adjacency matrix that the scGAE multitask-oriented graph autoencoder consumes to preserve topological and feature information.

## When to use

Use when preparing scRNA-seq count data (e.g. simulated scenario4 matrices, HRA000084 HSPCs, GSE84133 pancreas) as input to scGAE: a cell adjacency matrix must be built via KNN before training the graph autoencoder.

## When NOT to use

- Input is raw, unnormalized scRNA-seq counts — scGAE requires preprocessed data as input (preprocess with scTransform/Seurat or scanpy first)
- The downstream method ignores cell–cell relationships (e.g. methods that 'only embedded the distinct cell features while ignoring the cell–cell relationships') — the graph is built specifically for graph-autoencoder-based embedding
- A 2D visualization-only pipeline without graph structure is desired (the paper notes the graph autoencoder cannot directly embed into 2D; an intermediate dimension plus tSNE is used instead)

## Inputs

- Preprocessed scRNA-seq expression/count matrix (post-scTransform, e.g. from Seurat or scanpy)
- KNN neighbor count K (default 35, adjusted per dataset)

## Outputs

- Cell-to-cell adjacency matrix (from get_adj) fed to the SCGAE model

## How to apply

Preprocess the raw count matrix first with scTransform in the Seurat package (or Seurat/scanpy as the README allows), since scGAE takes preprocessed data as input. Build the cell graph with the KNN algorithm using the study's default K=35, adjusting K for the dataset at hand — the paper notes the optimal K varies among datasets and different parts of a dataset. Compute the adjacency matrix using the get_adj function from the scGAE repository (preprocessing.py, scgae.py, layers.py, losses.py, utils.py) and pass it to the SCGAE model. The resulting cell graph encodes cell–cell relationships so the subsequent multitask-oriented graph autoencoder preserves topological structure alongside feature information, unlike deep learning methods that embed cells while ignoring these relationships.

## Related tools

- **scGAE** (The pipeline whose cell graph is built via KNN (get_adj) and consumed by the multitask-oriented graph autoencoder implemented with Spektral) — github.com/ZixiangLuo1161/scGAE
- **t-SNE** (Used downstream of the KNN-graph-based embedding to visualize the embedded data in two dimensions)

## Examples

```
from preprocessing import get_adj; A = get_adj(preprocessed_data)  # KNN cell graph, default K=35, feeds the SCGAE model
```

## Evaluation signals

- Adjacency matrix computed by get_adj connects each cell to its K nearest neighbors, with K=35 as the default (or a dataset-adjusted K, as the authors did)
- Downstream scGAE run on the graph reproduces scenario4 robustness NMIs near 0.62, 0.62, 0.65, 0.61 across dropout rates 12±0.3%, 17±0.4%, 23±0.5%, 30±0.6% (each within ~0.55–0.70)
- On GSE84133 pancreas data, centroid distances in the 20-dimensional embedding show activated–quiescent stellate and ductal-subtype pairs closer than unrelated population pairs, confirming the graph preserved inter-population topology
- On HRA000084 HSPC data, Louvain clustering on the scGAE embedding resolves at least 2 subclusters within the Ba/Eo/MaP population (Fig. 4a,b structure)

## Limitations

- The optimal K for KNN graph construction remains unresolved across datasets — 'the optimal K varies among different datasets and different parts of a dataset', so K=35 may need adjustment
- No random seed or stochastic-variation control is reported for the pipeline, so graph-derived embeddings may vary between runs
- No software/package versions are specified for the tools in the paper (the README lists specific versions, e.g. tensorflow 2.3.0, spektral 0.6.1)

## Evidence

- [abstract] abstract: "scGAE builds a cell graph and uses a multitask-oriented graph autoencoder to preserve topological structure information and feature information"
- [discussion] discussion: "The default K is predened as 35 in this study and adjusted according to the datasets in our experiments."
- [readme] readme: "Compute the adjacency matrix using get_adj function and build the SCGAE model."
