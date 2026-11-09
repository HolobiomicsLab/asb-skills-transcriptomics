---
name: random-surfing-transition-matrix-computation
description: 'Use when constructing a cell-cell relationship graph from a normalized
  scRNA-seq expression matrix: after building the unweighted KNN graph, to compute
  transition probabilities and the PPMI-weighted adjacency.'
when_to_use_negative:
- Input is a raw count matrix or has not been normalized and filtered to highly variable
  genes — the paper's pipeline normalizes the scRNA-seq matrix and selects the top
  d highly variable genes before graph construction
- A weighted cell-cell relationship graph is already available — random surfing is
  designed to enrich an unweighted KNN graph, not to re-weight existing edge weights
- The downstream model does not use graph convolution over the cell-PPMI graph (e.g.,
  non-graph clustering like DEC or DCA) — the PPMI graph is specific to scMGCA-style
  aggregation
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMGCA
  role: Method whose cell-PPMI graph construction this skill implements; random surfing
    plus PPMI feeds the GCN encoder
  repo: https://pypi.org/project/scMGCA
- name: scipy
  role: Sparse-matrix operations for running random surfing on the KNN graph and storing
    the PPMI adjacency
- name: scikit-learn
  role: Nearest-neighbor search to build the unweighted KNN graph input to random
    surfing
- name: numpy
  role: Distance computation and PPMI matrix element-wise conversion/clipping
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-36134-7
    title: Topological identification and interpretation for single-cell gene regulation
      elucidation across multiple platforms using scMGCA
  source_content_hash: sha256:ca4485e10193529eb290f0d7a372b5083acef2e6453b3cbac7f93a72cb7997ea
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - anndata
  - numpy
  - scMGCA
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# random-surfing-transition-matrix-computation

## Summary

Runs a random surfing algorithm over the unweighted k-NN cell graph to obtain a transition matrix capturing vertex-to-vertex transition probabilities, which is then converted into the positive pointwise mutual information (PPMI) cell-PPMI graph used by scMGCA to aggregate neighboring cells.

## When to use

Use when constructing a cell-cell relationship graph from a normalized scRNA-seq expression matrix: after building the unweighted KNN graph, to compute transition probabilities and the PPMI-weighted adjacency.

## When NOT to use

- Input is a raw count matrix or has not been normalized and filtered to highly variable genes — the paper's pipeline normalizes the scRNA-seq matrix and selects the top d highly variable genes before graph construction
- A weighted cell-cell relationship graph is already available — random surfing is designed to enrich an unweighted KNN graph, not to re-weight existing edge weights
- The downstream model does not use graph convolution over the cell-PPMI graph (e.g., non-graph clustering like DEC or DCA) — the PPMI graph is specific to scMGCA-style aggregation

## Inputs

- Normalized scRNA-seq gene expression matrix (genes not expressed in cells filtered out; top d highly variable genes retained)
- Unweighted KNN cell graph adjacency matrix (Euclidean-distance k nearest neighbors, self-connections included)
- Random-surfing parameters: restart probability (1 − α) and maximum number of steps s = 3

## Outputs

- Transition matrix U of vertex-to-vertex transition probabilities from random surfing
- Cell-PPMI graph adjacency A with clipped nonnegative PPMI weights (sparse .npz, e.g., cell_ppmi_graph.npz)

## How to apply

First build the unweighted k-nearest-neighbor cell graph by computing Euclidean distances between nodes and connecting each node to its k shortest-distance neighbors including itself. Run the random-surfing algorithm on this KNN graph with the probability of returning to the initial vertex (restarting the process) set to (1 − α) and at most s = 3 steps, using SciPy sparse operations, to obtain the transition matrix U. Convert U into the positive pointwise mutual information (PPMI) matrix, clipping negative entries to zero, to form the weighted cell-PPMI graph adjacency A per Eqs. (5)-(7); the PPMI acts as a co-occurrence attention mechanism describing similar cells. Save A as a sparse matrix (e.g., cell_ppmi_graph.npz) for downstream scMGCA graph convolution. Note that the paper does not report numeric values for k or α, so parameter values must be taken from the released scMGCA code.

## Related tools

- **scMGCA** (Method whose cell-PPMI graph construction this skill implements; random surfing plus PPMI feeds the GCN encoder) — https://pypi.org/project/scMGCA
- **scipy** (Sparse-matrix operations for running random surfing on the KNN graph and storing the PPMI adjacency)
- **scikit-learn** (Nearest-neighbor search to build the unweighted KNN graph input to random surfing)
- **numpy** (Distance computation and PPMI matrix element-wise conversion/clipping)

## Examples

```
from sklearn.neighbors import kneighbors_graph; from scipy.sparse import save_npz; A_knn = kneighbors_graph(X, k, mode='connectivity', include_self=True); U = random_surf(A_knn, alpha, s=3); save_npz('cell_ppmi_graph.npz', ppmi_clip(U))
```

## Evaluation signals

- All entries of the resulting adjacency A are nonnegative PPMI weights (negative entries clipped to zero)
- Transition matrix U rows are consistent with a random walk on the KNN graph with restart probability (1 − α) and at most s = 3 steps
- Script runs end-to-end on the demo dataset without errors
- Output A is saved as a valid sparse matrix file (cell_ppmi_graph.npz) loadable with SciPy
- No numeric validation threshold is given in the paper for this intermediate graph — treat nonnegativity and structural consistency as the primary checks

## Limitations

- The paper does not specify numeric values for the number of KNN neighbors k or the restart probability parameter α; values must be recovered from the scMGCA implementation
- No numeric threshold is reported in the paper for validating the intermediate cell-PPMI graph
- Clipping negative PPMI entries to zero discards dissimilarity information between cells

## Evidence

- [results] Random surfing on KNN graph: "we used a random surfing algorithm based on KNN graph to aggregate neighboring cells"
- [discussion] Restart probability definition: "The probability of returning to the initial vertex and restarting the process is set to (1 − α)"
- [intro] Cell-PPMI graph objective: "we propose the cell-PPMI graph with a positive pointwise mutual information (PPMI) matrix and random surfing to aggregate adjacent cells"
- [discussion] KNN graph construction detail: "Through computing the Euclidean distance between nodes, and setting the number of neighbors k, each node is connected to the node within k shortest distance including itself"
