---
name: gaussian-kernel-affinity-computation
description: Use when a single-cell gene expression matrix (N cells x D genes) must
  be converted into a probabilistic neighborhood graph or Markov transition matrix
  as the substrate for diffusion/random-walk based analyses such as RWR, diffusion
  pseudotime, or trajectory inference.
when_to_use_negative:
- Input data is raw/unnormalized counts rather than a pre-processed gene expression
  matrix — DTFLOW assumes step A (pre-processing into X_NxD) has already been done
- The analysis requires a binary adjacency graph without distance-weighted affinities,
  or needs global (not locally scaled) bandwidths
- The dataset is so large that the O(N^2) pairwise distance and N x N dense matrices
  are infeasible (e.g. the MH dataset with 40,210 cells took 3108.35 s)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: DTFLOW
  role: Python package whose get_transition_matrix(k) builds the kNN Gaussian kernel
    affinity and row-normalized Markov transition matrix
  repo: https://github.com/statway/DTFLOW
- name: Python
  role: Implementation language for the kNN search, Gaussian kernel computation, and
    transition-matrix normalization
  repo: https://github.com/statway/DTFLOW
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2020.08.003
    title: 'DTFLOW: Inference and Visualization of Single-Cell Pseudotime Trajectory
      Using Diffusion Propagation'
  source_content_hash: sha256:dc1f276c99628521a1f78a281052480f78f21fc2bd9a3076c811789b49fe2d0c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/statway/DTFLOW
  tools:
  - DTFLOW
  - Python
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Gaussian kernel affinity computation

## Summary

Transforms pairwise Euclidean distances between single cells into a symmetric Gaussian kernel affinity matrix, with local bandwidth sigma_i set by each cell's distance to its k-th nearest neighbor (k=10 in DTFLOW), then row-normalizes to obtain the Markov transition matrix M used for random walk with restart diffusion.

## When to use

Use when a single-cell gene expression matrix (N cells x D genes) must be converted into a probabilistic neighborhood graph or Markov transition matrix as the substrate for diffusion/random-walk based analyses such as RWR, diffusion pseudotime, or trajectory inference.

## When NOT to use

- Input data is raw/unnormalized counts rather than a pre-processed gene expression matrix — DTFLOW assumes step A (pre-processing into X_NxD) has already been done
- The analysis requires a binary adjacency graph without distance-weighted affinities, or needs global (not locally scaled) bandwidths
- The dataset is so large that the O(N^2) pairwise distance and N x N dense matrices are infeasible (e.g. the MH dataset with 40,210 cells took 3108.35 s)

## Inputs

- Pre-processed single-cell gene expression matrix X (N cells x D genes), e.g. pandas DataFrame raw_df
- Number of nearest neighbors k (DTFLOW default/best: k=10)

## Outputs

- Row-normalized Markov transition probability matrix M (N x N), each row summing to 1
- Symmetric Gaussian kernel affinity matrix (pre-normalization) over the kNN graph

## How to apply

Load the pre-processed expression matrix X (N cells x D genes) and find the k=10 nearest neighbors of each cell (including the cell itself) on pairwise Euclidean distances. Convert distances into symmetric Gaussian kernel weights K(xi,xj)=exp(-||xi-xj||^2/(2*sigma_i*sigma_j)), where each local bandwidth sigma_i is the distance from cell i to its k-th nearest neighbor; symmetrize one-way neighbor pairs so K(xi,xj)=K(xj,xi). Row-normalize the kernel to obtain the Markov transition probability matrix M (Equation 3 in DTFLOW, reproducing get_transition_matrix(k)). The kNN graph with local-scaling bandwidths (Zelnik-Manor & Perona-style) makes the affinity robust to varying local cell densities; downstream RWR then uses S=(1-p)(I-pM)^(-1) with p=0.9.

## Related tools

- **DTFLOW** (Python package whose get_transition_matrix(k) builds the kNN Gaussian kernel affinity and row-normalized Markov transition matrix) — https://github.com/statway/DTFLOW
- **Python** (Implementation language for the kNN search, Gaussian kernel computation, and transition-matrix normalization) — https://github.com/statway/DTFLOW

## Examples

```
from DTFLOW import *; z = DTFLOW(raw_df, time_stages); M = z.get_transition_matrix(10)
```

## Evaluation signals

- Each row of the resulting transition matrix M is a valid probability distribution: all entries > 0 and each row sums to 1
- The kernel matrix is symmetric: K(xi,xj) == K(xj,xi) after symmetrizing one-way neighbor pairs
- With k=10, each cell's affinity row has non-negligible weights concentrated on at most 10 neighbors (including itself)
- Downstream RWR diffusion matrix S=(1-p)(I-pM)^(-1) also has rows that are non-vanishing distributions summing to 1
- Varying k changes results; the paper reports k=10 gives better pseudotime ordering accuracy than other tested values

## Limitations

- Parameter values (Gaussian kernel bandwidth, number of neighbors k, restart probability 1-p) may vary from dataset to dataset and are not standardized; optimal parameter selection is deferred to future work
- DTFLOW's BKFD uses the same restart probability for all nodes, which may limit the effectiveness of the random walk
- The method (per README) can only be used for technology research, not for commercial purposes

## Evidence

- [methods] Find the k=10 most similar neighbors (including the cell itself) of each cell via kNN on pairwise Euclidean distances and transform distances into symmetric Gaussian kernel weights K(xi,xj)=exp(-||xi-xj||^2/(2*sigma_i*sigma_j)): [evidence span withheld — anchor status: not_found]
- [methods] Normalize the kernel to obtain the row-normalized Markov transition probability matrix M (Equation 3), reproducing get_transition_matrix(k).: [evidence span withheld — anchor status: not_found]
- [results] We test different values of k and find that the results are better if k=10: "We test different values of k and find that the results are better if k=10"
- [readme] M= z.get_transition_matrix(k): "2. get one Markov transition_matrix.   

``nM= z.get_transition_matrix(k)
``nHere k is number of each node's neighbors."
- [discussion] the values of these parameters may vary from dataset to dataset: "the values of these parameters may vary from dataset to dataset."
