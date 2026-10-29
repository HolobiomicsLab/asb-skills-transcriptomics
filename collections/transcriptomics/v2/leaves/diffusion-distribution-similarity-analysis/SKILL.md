---
name: diffusion-distribution-similarity-analysis
description: Use when single-cell data has been converted into per-cell stationary
  diffusion distributions (RWR matrix S) and a kernel-based, unit-diagonal similarity
  embedding is needed for pseudotime ordering or 2D/3D visualization.
when_to_use_negative:
- Input is a raw gene expression matrix rather than a per-cell diffusion distribution
  matrix — DTFLOW requires the RWR step (get_transition_matrix, get_diffusionRWR)
  to run first
- The analysis requires comparing cells by their raw expression profiles instead of
  by diffusion-based distributional similarity
- No well-defined developmental/trajectory structure exists, so diffusion propagation
  does not capture meaningful transition dynamics
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: DTFLOW
  role: Python implementation exposing get_transition_matrix, get_diffusionRWR, and
    get_Y(slover='svd', dim) for the kernel construction and SVD embedding
  repo: https://github.com/statway/DTFLOW
- name: Python
  role: Language in which DTFLOW is implemented and distributed (run via Anaconda/Jupyter,
    e.g., guo2010.ipynb)
  repo: https://github.com/statway/DTFLOW
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2020.08.003
    title: 'DTFLOW: Inference and Visualization of Single-Cell Pseudotime Trajectory
      Using Diffusion Propagation'
  source_content_hash: sha256:3d901f59c95d21cb174c6345564d01a91c0812ccb5521247be8fb031fd6492b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/statway/DTFLOW
  tools:
  - DTFLOW
  - Python
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# diffusion-distribution-similarity-analysis

## Summary

Reconstructs the Bhattacharyya kernel matrix from random walk with restart (RWR) diffusion distributions and derives a low-dimensional SVD embedding, the core of DTFLOW's Bhattacharyya kernel feature decomposition (BKFD) step.

## When to use

Use when single-cell data has been converted into per-cell stationary diffusion distributions (RWR matrix S) and a kernel-based, unit-diagonal similarity embedding is needed for pseudotime ordering or 2D/3D visualization.

## When NOT to use

- Input is a raw gene expression matrix rather than a per-cell diffusion distribution matrix — DTFLOW requires the RWR step (get_transition_matrix, get_diffusionRWR) to run first
- The analysis requires comparing cells by their raw expression profiles instead of by diffusion-based distributional similarity
- No well-defined developmental/trajectory structure exists, so diffusion propagation does not capture meaningful transition dynamics

## Inputs

- Stationary diffusion distribution matrix S (N×N) from z.get_diffusionRWR(p), computed by RWR on the kNN Markov transition matrix M = z.get_transition_matrix(k)
- Restart probability p (DTFLOW uses p=0.9) and neighbor count k (DTFLOW uses k=10)
- Target embedding dimension d (2 or 3)

## Outputs

- Bhattacharyya kernel matrix G (N×N, unit diagonal)
- Log-kernel matrix logG (N×N, zero diagonal)
- Low-dimensional embedding matrix Yd (N×d), e.g., saved as bkfd_embedding.csv

## How to apply

Starting from the stationary diffusion matrix S produced by RWR on the kNN Markov transition matrix (k=10 neighbors, restart probability p=0.9 per DTFLOW's tuning), compute the Bhattacharyya kernel G_ij = sqrt(<s_i, s_j>) from inner products between rows of S; every diagonal element of G must equal one. Take the element-wise logarithm of G to form logG, which defines a valid inner-product kernel via the mapping phi(s_i) and has a zero diagonal because log G_ii = 0. Decompose logG by singular value decomposition (logG = V Σ V^T), retain the largest d positive singular values (d=2 or d=3), and form the N×d embedding Yd = Vd Σd^(1/2). Save Yd (e.g., bkfd_embedding.csv) for downstream DTFLOW pseudotime ordering and branch detection. Note that DPT is a special case of this RWR formulation with p=1, so the restart probability is a tunable decision point.

## Related tools

- **DTFLOW** (Python implementation exposing get_transition_matrix, get_diffusionRWR, and get_Y(slover='svd', dim) for the kernel construction and SVD embedding) — https://github.com/statway/DTFLOW
- **Python** (Language in which DTFLOW is implemented and distributed (run via Anaconda/Jupyter, e.g., guo2010.ipynb)) — https://github.com/statway/DTFLOW

## Examples

```
from DTFLOW import *; z = DTFLOW(raw_df,time_stages); M = z.get_transition_matrix(10); S = z.get_diffusionRWR(0.9); Y = z.get_Y(slover='svd', 2)
```

## Evaluation signals

- Kernel matrix G has unit diagonal (G_ii = sqrt(<s_i, s_i>) = 1 for all i)
- logG has zero diagonal, consistent with ||y_i||^2 = log G_ii = 0 in the embedding space
- All retained singular values (d=2 or 3) are strictly positive
- Embedding Yd has shape N×d and rows correspond one-to-one with cells in S
- Downstream pseudotime Kendall rank correlation (e.g., 0.862 on the ME dataset with DTFLOW) is reproducible with p=0.9, k=10

## Limitations

- BKFD uses the same restart probability for all the nodes, which may limit the effectiveness of the random walk
- Parameter values (Gaussian kernel, number of closest neighbors k, restart probability 1-p) may vary from dataset to dataset and are not standardized
- Expressing each cell by a distribution vector in a better way remains an open challenge
- The DTFLOW repository states the method 'only can be used for technology research, not for commercial purposes'

## Evidence

- [intro] This method uses a new manifold learning method, named Bhattacharyya kernel feature: "This method uses a new manifold learning method, named Bhattacharyya kernel feature"
- [abstract] a new method called Bhattacharyya kernel feature decomposition (BKFD) to reduce the data dimensions: "a new method called Bhattacharyya kernel feature decomposition (BKFD) to reduce the data dimensions"
- [abstract] we first establish a stationary distribution for each cell to represent the transition of cellular developmental states based on the random walk with restart algorithm: "we first establish a stationary distribution for each cell to represent the transition of cellular developmental states based on the random walk with restart algorithm"
- [methods] the RWR algorithm can be regarded as a more general approach and DPT is a special case of the RWR algorithm (i.e., p=1): "the RWR algorithm can be regarded as a more general approach and DPT is a special case of the RWR algorithm (i.e., p=1)"
- [readme] S = z.get_diffusionRWR(p): "S = z.get_diffusionRWR(p)"
- [readme] Y = z.get_Y(slover='svd',dim): "Y = z.get_Y(slover='svd',dim)"
- [results] the ordering accuracy is better when the value of p is around 0.9. Thus, we use p=0.9 in this work: "the ordering accuracy is better when the value of p is around 0.9. Thus, we use p=0.9 in this work"
