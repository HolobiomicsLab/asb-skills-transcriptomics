---
name: bhattacharyya-kernel-construction
description: Use when a single-cell expression matrix has already been converted to
  a stationary diffusion distribution matrix S via RWR and you need a valid inner-product
  kernel plus 2D/3D embedding for pseudotime ordering.
when_to_use_negative:
- The input is a raw gene expression matrix that has not yet been converted into RWR
  diffusion distributions — build the Markov transition matrix and run RWR first.
- Only a scalar similarity matrix (not per-cell distributions) is available, so Bhattacharyya
  inner products between rows of S cannot be computed.
- The analysis requires diffusion maps (DPT) rather than the Bhattacharyya-kernel
  formulation; note the article treats DPT as the special case of RWR with p=1.
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DTFLOW
  role: 'Implements BKFD: get_diffusionRWR(p) produces the RWR diffusion matrix S
    and get_Y(slover=''svd'',dim) performs SVD on the Bhattacharyya log-kernel to
    yield the embedding Y.'
  repo: https://github.com/statway/DTFLOW
- name: Python
  role: Implementation language for DTFLOW and the BKFD kernel/SVD pipeline.
  repo: https://github.com/statway/DTFLOW
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2020.08.003
    title: 'DTFLOW: Inference and Visualization of Single-Cell Pseudotime Trajectory
      Using Diffusion Propagation'
  source_content_hash: sha256:9d76d70ca7494d3d4a9848011e8ea8e9582d9870ff0a220a90e01a39ddccdca9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/statway/DTFLOW
  tools:
  - DTFLOW
  - Python
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# bhattacharyya-kernel-construction

## Summary

Constructs a Bhattacharyya kernel matrix from the random walk with restart (RWR) diffusion distributions of single cells, then applies element-wise log and SVD to obtain the BKFD low-dimensional embedding used by DTFLOW for pseudotime inference.

## When to use

Use when a single-cell expression matrix has already been converted to a stationary diffusion distribution matrix S via RWR and you need a valid inner-product kernel plus 2D/3D embedding for pseudotime ordering.

## When NOT to use

- The input is a raw gene expression matrix that has not yet been converted into RWR diffusion distributions — build the Markov transition matrix and run RWR first.
- Only a scalar similarity matrix (not per-cell distributions) is available, so Bhattacharyya inner products between rows of S cannot be computed.
- The analysis requires diffusion maps (DPT) rather than the Bhattacharyya-kernel formulation; note the article treats DPT as the special case of RWR with p=1.

## Inputs

- Stationary diffusion distribution matrix S (N×N) from RWR on the kNN Markov transition matrix (k=10, restart probability 1-p with p=0.9)
- Markov transition matrix M from z.get_transition_matrix(k)
- Desired embedding dimension d (2 or 3)

## Outputs

- Bhattacharyya kernel matrix G with unit diagonal
- Log-kernel matrix logG (zero diagonal) used as inner-product kernel
- N×d BKFD low-dimensional embedding matrix Yd (e.g., bkfd_embedding.csv)

## How to apply

Load the stationary diffusion matrix S produced by random walk with restart (restart probability 1-p, with p=0.9 throughout DTFLOW) on the kNN Markov transition matrix (k=10 nearest neighbors). Build the Bhattacharyya kernel G_ij = sqrt(<s_i, s_j>) from the inner products between rows of S, so the diagonal of G equals one. Apply the element-wise logarithm to form logG, which defines a valid inner-product kernel via the mapping phi(s_i), and perform singular value decomposition logG = V Σ V^T. Retain the largest d positive singular values (d=2 or d=3) and compute the embedding Yd = Vd Σd^(1/2), then export the N×d matrix (e.g., bkfd_embedding.csv) for downstream pseudotime and RSKG branch detection. In DTFLOW this is invoked via get_Y(slover='svd', dim) on a DTFLOW object.

## Related tools

- **DTFLOW** (Implements BKFD: get_diffusionRWR(p) produces the RWR diffusion matrix S and get_Y(slover='svd',dim) performs SVD on the Bhattacharyya log-kernel to yield the embedding Y.) — https://github.com/statway/DTFLOW
- **Python** (Implementation language for DTFLOW and the BKFD kernel/SVD pipeline.) — https://github.com/statway/DTFLOW

## Examples

```
from DTFLOW import *; z = DTFLOW(raw_df, time_stages); M = z.get_transition_matrix(10); S = z.get_diffusionRWR(0.9); Y = z.get_Y(slover='svd', 2)
```

## Evaluation signals

- Bhattacharyya kernel matrix G has unit diagonal (G_ii = sqrt(<s_i, s_i>) = 1 for valid distributions).
- logG has zero diagonal, consistent with the norm identity ||y_i||^2 = log G_ii = 0.
- All retained singular values of logG are positive; exactly d (2 or 3) are kept.
- Embedding Yd has shape N×d and is written successfully to bkfd_embedding.csv.
- Downstream pseudotime values derived from D_ij = sqrt(-2*log(G_ij)) fall within [0,1] after min-max normalization.

## Limitations

- BKFD uses the same restart probability (1-p) for all nodes, which may limit the effectiveness of the random walk.
- Parameter values (Gaussian kernel width, number of neighbors k, restart probability) may vary from dataset to dataset and are not standardized.
- Expressing each cell by a distribution vector in a better way remains an open challenge; the chosen representation may not be optimal.

## Evidence

- [methods] RWR diffusion and Bhattacharyya kernel usage: [evidence span withheld — anchor status: not_found]
- [intro] SVD dimension reduction step in DTFLOW workflow: "Y = z.get_Y(slover='svd',dim)"
- [readme] RWR diffusion distribution computation: "S = z.get_diffusionRWR(p)"
- [results] Choice of restart probability p=0.9: "the ordering accuracy is better when the value of p is around 0.9. Thus, we use p=0.9 in this work"
- [methods] DPT as special case of RWR: "the RWR algorithm can be regarded as a more general approach and DPT is a special case of the RWR algorithm (i.e., p=1)"
