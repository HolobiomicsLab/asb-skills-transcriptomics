---
name: single-cell-trajectory-embedding-visualization
description: Use when you need a low-dimensional (d=2 or 3) manifold embedding and
  plot of single cells for trajectory visualization, starting from a preprocessed
  N cells × D genes expression matrix and the DTFLOW API.
when_to_use_negative:
- The input is already a reduced feature table (e.g. approximate principal components)
  or an existing embedding rather than the N×D gene expression matrix — BKFD must
  derive its diffusion distributions from the expression data itself.
- You need node-specific restart behavior for a heterogeneous graph — the paper notes
  BKFD uses the same restart probability for all nodes, which may limit the effectiveness
  of the random walk.
- A valid bkfd_embedding.csv already exists and the task only needs downstream pseudotime
  (get_dtflow_times) or branch detection (get_branches(Num)) rather than regenerating
  the embedding.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0203
tools:
- name: DTFLOW
  role: 'Implements this skill''s pipeline: get_transition_matrix(k) for the kNN Markov
    matrix, get_diffusionRWR(p) for the RWR diffusion distributions, get_Y(slover=''svd'',dim)
    for the SVD-based BKFD embedding, and plotY for visualization'
  repo: https://github.com/statway/DTFLOW
- name: Python
  role: Language in which DTFLOW's source is implemented and run (installed via Anaconda;
    example notebook guo2010.ipynb opened in Jupyter)
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2020.08.003
    title: 'DTFLOW: Inference and Visualization of Single-Cell Pseudotime Trajectory
      Using Diffusion Propagation'
  source_content_hash: sha256:1b202b88ff52ea80338078785dd69a01e5019e43f725c853287712a4d194ca97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/statway/DTFLOW
  tools:
  - DTFLOW
  - Python
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# single-cell-trajectory-embedding-visualization

## Summary

Transforms each single cell into a random-walk-with-restart (RWR) diffusion distribution over the k=10 kNN Markov graph, measures cell-cell similarity with the Bhattacharyya kernel (G_ij = sqrt(<s_i, s_j>)), and SVD-decomposes log(G) into a d=2 or 3 dimensional BKFD embedding for pseudotime trajectory visualization in DTFLOW.

## When to use

Use when you need a low-dimensional (d=2 or 3) manifold embedding and plot of single cells for trajectory visualization, starting from a preprocessed N cells × D genes expression matrix and the DTFLOW API.

## When NOT to use

- The input is already a reduced feature table (e.g. approximate principal components) or an existing embedding rather than the N×D gene expression matrix — BKFD must derive its diffusion distributions from the expression data itself.
- You need node-specific restart behavior for a heterogeneous graph — the paper notes BKFD uses the same restart probability for all nodes, which may limit the effectiveness of the random walk.
- A valid bkfd_embedding.csv already exists and the task only needs downstream pseudotime (get_dtflow_times) or branch detection (get_branches(Num)) rather than regenerating the embedding.

## Inputs

- Pre-processed gene expression matrix X (N cells × D genes, raw_df) plus per-cell time_stages labels, as passed to DTFLOW(raw_df, time_stages)
- Number of nearest neighbors k (k=10 in the paper) and RWR parameter p, where 1-p is the restart probability (p=0.9 in the paper)
- Embedding dimension dim for get_Y (d=2 or d=3) and a plot color variable: 'stages', 'dftime', or 'branches'

## Outputs

- N×N stationary diffusion distribution matrix S (every entry s_ij > 0 and each row sums to 1), e.g. saved as diffusion_matrix_S.npy
- N×d BKFD embedding matrix Yd (d=2 or 3), e.g. saved as bkfd_embedding.csv
- Embedding scatter plot from z.plotY(size=(10,10)) colored by stages, dftime, or branches

## How to apply

Create a DTFLOW object from the preprocessed N cells × D genes matrix (z = DTFLOW(raw_df, time_stages)), build the Markov transition matrix M via z.get_transition_matrix(k) over the k=10 nearest-neighbor graph with symmetric Gaussian weights K(xi,xj)=exp(-||xi-xj||^2/(2*sigma_i*sigma_j)) whose local width sigma_i is the distance to the k-th neighbor, and compute the stationary diffusion distributions S=(1-p)(I-pM)^(-1) with z.get_diffusionRWR(0.9), the restart probability used throughout the paper. Form the Bhattacharyya kernel matrix G by taking element-wise square roots of inner products between rows of S (G_ij = sqrt(<s_i, s_j>)), so every diagonal element equals one. Apply the element-wise logarithm to G to obtain logG (zero diagonal), perform the singular value decomposition logG = V Σ V^T, and retain the largest d positive singular values (d=2 or d=3) to compute the low-dimensional embedding Yd = Vd Σd^(1/2). Save the resulting N×d matrix as bkfd_embedding.csv and render the cells with z.plotY(size=(10,10), color='stages'|'dftime'|'branches'). This embedding is the coordinate system on which get_dtflow_times(root_list) orders pseudotime and get_branches(Num) detects sub-branches, so validate it before passing it downstream.

## Related tools

- **DTFLOW** (Implements this skill's pipeline: get_transition_matrix(k) for the kNN Markov matrix, get_diffusionRWR(p) for the RWR diffusion distributions, get_Y(slover='svd',dim) for the SVD-based BKFD embedding, and plotY for visualization) — https://github.com/statway/DTFLOW
- **Python** (Language in which DTFLOW's source is implemented and run (installed via Anaconda; example notebook guo2010.ipynb opened in Jupyter))

## Examples

```
z = DTFLOW(raw_df,time_stages); M = z.get_transition_matrix(10); S = z.get_diffusionRWR(0.9); Y = z.get_Y(slover='svd',2); z.plotY(size=(10,10),color ='stages')
```

## Evaluation signals

- The Bhattacharyya kernel matrix G has unit diagonal (every G_ii = 1), so logG has zero diagonal and ||y_i||^2 = log G_ii = 0 holds in the embedding.
- Each row of the diffusion matrix S is a non-vanishing probability distribution: all entries s_ij > 0 and every row sums to 1.
- All retained singular values in Σd are strictly positive and Yd has shape N×d with d=2 or 3.
- Downstream benchmark: on the ME dataset (48 genes, 438 single cells) the pseudotime derived from this embedding reaches Kendall rank correlation 0.862 for DTFLOW (vs 0.796 Scanpy and 0.761 Monocle2).
- z.plotY(size=(10,10)) renders the N×d coordinates colored by 'stages', 'dftime', or 'branches' without error on the guo2010 example.

## Limitations

- BKFD uses the same restart probability for all the nodes, which may limit the effectiveness of the random walk.
- Parameter values (Gaussian kernel width, number of neighbors k, restart probability 1-p) may vary from dataset to dataset and are not standardized.
- Expressing each cell by a distribution vector in a better way remains an open challenge acknowledged by the authors.

## Evidence

- [abstract] we first establish a stationary distribution for each cell to represent the transition of cellular developmental states based on the random walk with restart algorithm: "we first establish a stationary distribution for each cell to represent the transition of cellular developmental states based on the random walk with restart algorithm"
- [abstract] a new method called Bhattacharyya kernel feature decomposition (BKFD) to reduce the data dimensions: "a new method called Bhattacharyya kernel feature decomposition (BKFD) to reduce the data dimensions"
- [results] the ordering accuracy is better when the value of p is around 0.9. Thus, we use p=0.9 in this work: "the ordering accuracy is better when the value of p is around 0.9. Thus, we use p=0.9 in this work"
- [results] We test different values of k and find that the results are better if k=10: "We test different values of k and find that the results are better if k=10"
- [results] The calculated Kendall rank correlation coefficients are 0.862, 0.796, 0.761 for DTFLOW, Scanpy, and Monocle2, respectively: "The calculated Kendall rank correlation coefficients are 0.862, 0.796, 0.761 for DTFLOW, Scanpy, and Monocle2, respectively"
- [readme] 4. dimension reduction with SVD, and plot the result of dimension reduction: "4. dimension reduction with SVD, and plot the result of dimension reduction"
- [readme] z.plotY(size=(10,10),color ='stages'): "z.plotY(size=(10,10),color ='stages')"
- [discussion] BKFD uses the same restart probability for all the nodes, and this may limit the effectiveness of random walk [49].: "BKFD uses the same restart probability for all the nodes, and this may limit the effectiveness of random walk [49]."
