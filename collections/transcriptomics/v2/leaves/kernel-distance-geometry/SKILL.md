---
name: kernel-distance-geometry
description: Use when single-cell data has been transformed into per-cell stationary
  diffusion distributions (RWR matrix S) and a kernel-based low-dimensional embedding
  is needed for pseudotime ordering or trajectory visualization.
when_to_use_negative:
- Input is raw gene expression without an RWR diffusion distribution step — the Bhattacharyya
  kernel requires per-cell probability distribution vectors, not raw counts or normalized
  expression
- The goal is branch detection or pseudotime ordering itself — this skill only produces
  the BKFD embedding; use RSKG branch detection and root-cell-based ordering downstream
  instead
- Downstream method requires a Euclidean distance on expression space (e.g., PCA on
  expression) rather than a kernel-induced embedding on diffusion distributions
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: DTFLOW
  role: 'Implements the BKFD pipeline: get_diffusionRWR produces S, get_Y(slover=''svd'',
    dim) performs the kernel-based SVD embedding, plotY visualizes it'
  repo: https://github.com/statway/DTFLOW
- name: Python
  role: Implementation language of the DTFLOW source code used to compute the kernel
    matrix, log transform, and SVD
  repo: https://github.com/statway/DTFLOW
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2020.08.003
    title: 'DTFLOW: Inference and Visualization of Single-Cell Pseudotime Trajectory
      Using Diffusion Propagation'
  source_content_hash: sha256:2afabc7f9351e8035b6520ed89736fb96fb41267870f525742fb4031e08d153c
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
yes_rate: 0.875
n_claims: 8
---

# kernel-distance-geometry

## Summary

Constructs a Bhattacharyya kernel matrix from RWR diffusion distributions and derives a low-dimensional cell embedding via SVD of its element-wise logarithm, as implemented in the BKFD step of DTFLOW for single-cell pseudotime visualization.

## When to use

Use when single-cell data has been transformed into per-cell stationary diffusion distributions (RWR matrix S) and a kernel-based low-dimensional embedding is needed for pseudotime ordering or trajectory visualization.

## When NOT to use

- Input is raw gene expression without an RWR diffusion distribution step — the Bhattacharyya kernel requires per-cell probability distribution vectors, not raw counts or normalized expression
- The goal is branch detection or pseudotime ordering itself — this skill only produces the BKFD embedding; use RSKG branch detection and root-cell-based ordering downstream instead
- Downstream method requires a Euclidean distance on expression space (e.g., PCA on expression) rather than a kernel-induced embedding on diffusion distributions

## Inputs

- Stationary diffusion matrix S (N×N) from RWR on the kNN Markov transition matrix, with restart probability 1-p (p=0.9, k=10)
- Number of retained dimensions d (2 or 3)

## Outputs

- Bhattacharyya kernel matrix G (N×N, unit diagonal)
- logG matrix (element-wise log of G, zero diagonal)
- N×d low-dimensional embedding matrix Yd (e.g., saved as bkfd_embedding.csv)

## How to apply

Start from the stationary diffusion matrix S produced by random walk with restart (restart probability 1-p with p=0.9) on the kNN Markov transition matrix (k=10 nearest neighbors). Build the Bhattacharyya kernel matrix G with G_ij = sqrt(<s_i, s_j>), i.e., the square root of inner products between rows of S; because each row of S is a probability distribution, the diagonal of G equals one. Take the element-wise logarithm logG, which defines a valid inner-product kernel under the mapping phi(s_i) and has zero diagonal (||y_i||^2 = log G_ii = 0). Perform singular value decomposition logG = V Σ V^T, retain the largest d positive singular values (d=2 or d=3), and compute the embedding Yd = Vd Σd^(1/2). In DTFLOW this is exposed as z.get_Y(slover='svd', dim), after which the N×d embedding is saved (e.g., bkfd_embedding.csv) for downstream pseudotime ordering and plotting.

## Related tools

- **DTFLOW** (Implements the BKFD pipeline: get_diffusionRWR produces S, get_Y(slover='svd', dim) performs the kernel-based SVD embedding, plotY visualizes it) — https://github.com/statway/DTFLOW
- **Python** (Implementation language of the DTFLOW source code used to compute the kernel matrix, log transform, and SVD) — https://github.com/statway/DTFLOW

## Examples

```
from DTFLOW import *; z = DTFLOW(raw_df, time_stages); M = z.get_transition_matrix(10); S = z.get_diffusionRWR(0.9); Y = z.get_Y(slover='svd', 2)
```

## Evaluation signals

- Bhattacharyya kernel matrix G has unit diagonal (each s_i is a probability distribution, so <s_i,s_i> = 1)
- logG has zero diagonal, satisfying ||y_i||^2 = log G_ii = 0
- All retained singular values (d=2 or 3) are positive
- Embedding Yd is N×d and reproduces the DTFLOW visualization when passed to z.plotY (color by stages/dftime/branches)
- Compare against scanpy/Monocle2 embeddings via Kendall rank correlation of downstream pseudotime (DTFLOW achieved 0.862 on ME and 0.761 on MFG datasets)

## Limitations

- BKFD uses the same restart probability for all nodes, which may limit the effectiveness of the random walk
- Parameter values (Gaussian kernel, number of neighbors k, restart probability) may vary from dataset to dataset and are not standardized; k=10 and p=0.9 were empirically best but selection is deferred to future work
- Expressing each cell by a distribution vector in a better way remains an open challenge

## Evidence

- [other] BKFD uses the random walk with restart (RWR) method to transform each data point into a discrete distribution, and uses the Bhattacharyya kernel to calculate the similarities between cells.: [evidence span withheld — anchor status: not_found]
- [readme] RWR/DTFLOW API for diffusion matrix and SVD embedding: "S = z.get_diffusionRWR(p)"
- [readme] DTFLOW API call for SVD-based dimension reduction: "Y = z.get_Y(slover='svd',dim)"
- [results] Restart probability choice p=0.9: "the ordering accuracy is better when the value of p is around 0.9. Thus, we use p=0.9 in this work"
- [results] Choice of k=10 nearest neighbors: "We test different values of k and find that the results are better if k=10"
- [methods] RWR generality vs DPT special case: "the RWR algorithm can be regarded as a more general approach and DPT is a special case of the RWR algorithm (i.e., p=1)"
