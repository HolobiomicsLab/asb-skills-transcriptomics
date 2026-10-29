---
name: ppmi-graph-weighting
description: Use when a normalized scRNA-seq gene expression matrix must be converted
  into a weighted cell–cell adjacency for graph-embedding clustering, and an unweighted
  KNN adjacency is insufficient to capture cell–cell transition probabilities.
when_to_use_negative:
- The input is already a cell–cell adjacency or graph-embedded representation rather
  than a gene-expression matrix needing graph construction
- The expression matrix is raw, unnormalized counts — normalization and highly variable
  gene selection must happen first
- The goal is a gene–gene or PPI network rather than a cell–cell relationship graph
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scMGCA
  role: Overall pipeline whose cell-PPMI graph construction this skill performs; the
    weighted graph feeds its graph convolutional autoencoder
  repo: https://pypi.org/project/scMGCA
- name: Scanpy
  role: Loads and normalizes the scRNA-seq expression matrix and selects highly variable
    genes before KNN construction
- name: anndata
  role: Container format for the normalized expression matrix used as input
- name: scikit-learn
  role: Computes Euclidean distances and the k-nearest-neighbor cell graph
- name: scipy
  role: Sparse-matrix operations for random surfing and for saving the PPMI adjacency
    to .npz
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-36134-7
    title: Topological identification and interpretation for single-cell gene regulation
      elucidation across multiple platforms using scMGCA
  source_content_hash: sha256:68eca4ba19ecfd53745c7ee76c2573dc7a8a56d04ef99d234e50fc05b6285c21
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Scanpy
  - anndata
  - numpy
  - scMGCA
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# ppmi-graph-weighting

## Summary

Constructs a weighted cell–cell graph from a normalized scRNA-seq expression matrix by building an unweighted KNN graph and re-weighting its edges via a random-surfing transition matrix converted into a positive pointwise mutual information (PPMI) matrix, as used by scMGCA to feed topology into its graph convolutional autoencoder.

## When to use

Use when a normalized scRNA-seq gene expression matrix must be converted into a weighted cell–cell adjacency for graph-embedding clustering, and an unweighted KNN adjacency is insufficient to capture cell–cell transition probabilities.

## When NOT to use

- The input is already a cell–cell adjacency or graph-embedded representation rather than a gene-expression matrix needing graph construction
- The expression matrix is raw, unnormalized counts — normalization and highly variable gene selection must happen first
- The goal is a gene–gene or PPI network rather than a cell–cell relationship graph

## Inputs

- Normalized scRNA-seq gene expression matrix (genes × cells), loaded via anndata/Scanpy after filtering unexpressed genes and selecting the top d highly variable genes
- Number of KNN neighbors k (value not specified in the paper; must be chosen)
- Random-surfing restart parameter α (restart probability 1 − α, value not specified) and step cap s = 3

## Outputs

- Weighted cell graph adjacency matrix A of nonnegative PPMI weights, saved as a SciPy sparse matrix to cell_ppmi_graph.npz

## How to apply

Start from the normalized expression matrix after filtering out genes not expressed in any cell and retaining the top d highly variable genes (anndata/Scanpy). Compute Euclidean distances between all cells (scikit-learn/NumPy) and connect each node to its k shortest-distance neighbors including itself to form the unweighted KNN graph; note the paper leaves k unspecified so it must be chosen empirically. Run the random-surfing algorithm on this KNN graph with restart probability set to (1 − α) and at most s = 3 steps (SciPy sparse operations) to obtain the transition matrix U. Convert U into the PPMI matrix, clip negative entries to zero per Eq. (5)–(7) to form the weighted adjacency A, and save A as a sparse matrix (cell_ppmi_graph.npz). The rationale is that PPMI co-occurrence weights encode which cells are 'similar' in transition-probability terms, enhancing the graph before the GCN encoder aggregates adjacent cells.

## Related tools

- **scMGCA** (Overall pipeline whose cell-PPMI graph construction this skill performs; the weighted graph feeds its graph convolutional autoencoder) — https://pypi.org/project/scMGCA
- **Scanpy** (Loads and normalizes the scRNA-seq expression matrix and selects highly variable genes before KNN construction)
- **anndata** (Container format for the normalized expression matrix used as input)
- **scikit-learn** (Computes Euclidean distances and the k-nearest-neighbor cell graph)
- **scipy** (Sparse-matrix operations for random surfing and for saving the PPMI adjacency to .npz)

## Examples

```
from sklearn.neighbors import kneighbors_graph; A_knn = kneighbors_graph(X_norm, n_neighbors=k, metric='euclidean', include_self=True)
```

## Evaluation signals

- The script runs end-to-end on the demo dataset without errors
- Every entry of the output adjacency A is nonnegative — no negative PPMI entries survive the clipping step
- A is stored as a sparse matrix in cell_ppmi_graph.npz with dimensions matching the number of cells in the input expression matrix
- The transition matrix U from random surfing uses at most s = 3 steps and restart probability (1 − α) before PPMI conversion
- No numeric accuracy threshold applies to this intermediate graph; correctness is structural (nonnegative weights, valid sparse format), not metric-based

## Limitations

- The paper does not state a numeric value for the number of KNN neighbors k, so graph sparsity must be chosen empirically
- The restart probability parameter α is not assigned a numeric value in the paper
- No numeric validation threshold is reported for the intermediate PPMI graph; verification is limited to end-to-end execution and nonnegativity of entries

## Evidence

- [discussion] knn_self_loops: "Through computing the Euclidean distance between nodes, and setting the number of neighbors k, each node is connected to the node within k shortest distance including itself"
- [discussion] random_surfing_restart: "The probability of returning to the initial vertex and restarting the process is set to (1 − α)"
- [results] knn_random_surfing: "we used a random surfing algorithm based on KNN graph to aggregate neighboring cells"
- [results] hvg_selection: "We first normalized the scRNA-seq gene expression matrix and selected the highly variable genes"
- [readme] readme_purpose: "clustering single-cell data based on a graph-embedding autoencoder that simultaneously learns cell–cell topology representation"
