---
name: singular-value-decomposition-dimensionality-reduction
description: Use when a cell-by-cell similarity matrix (e.g., a Bhattacharyya kernel
  over RWR diffusion distributions) must be converted to a low-dimensional embedding
  for visualization or downstream pseudotime ordering.
when_to_use_negative:
- The input is already a low-dimensional feature embedding or a cell-by-gene expression
  matrix that has not first been transformed into RWR diffusion distributions S
- The kernel matrix has non-positive or zero singular values in the retained dimensions,
  or the diagonal of G deviates from one (logG diagonal must be zero)
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: DTFLOW
  role: Provides the BKFD implementation, including get_diffusionRWR to produce S
    and get_Y(slover='svd', dim) to compute the SVD embedding
  repo: https://github.com/statway/DTFLOW
- name: Python
  role: Language in which the DTFLOW source code is distributed and executed (via
    Anaconda/Jupyter, e.g. guo2010.ipynb)
  repo: https://github.com/statway/DTFLOW
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2020.08.003
    title: 'DTFLOW: Inference and Visualization of Single-Cell Pseudotime Trajectory
      Using Diffusion Propagation'
  source_content_hash: sha256:8140ac369be80d3bdef5cb5cfecf9f32a23f9def87513f5658665de66222c33e
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
yes_rate: 0.8888888888888888
n_claims: 9
---

# singular-value-decomposition-dimensionality-reduction

## Summary

SVD-based low-dimensional embedding of single cells as implemented in the BKFD step of DTFLOW: eigendecomposition of the log of the Bhattacharyya kernel matrix of RWR diffusion distributions yields an N×d embedding (Yd = Vd Σd^(1/2), d=2 or 3) used for pseudotime ordering and visualization.

## When to use

Use when a cell-by-cell similarity matrix (e.g., a Bhattacharyya kernel over RWR diffusion distributions) must be converted to a low-dimensional embedding for visualization or downstream pseudotime ordering.

## When NOT to use

- The input is already a low-dimensional feature embedding or a cell-by-gene expression matrix that has not first been transformed into RWR diffusion distributions S
- The kernel matrix has non-positive or zero singular values in the retained dimensions, or the diagonal of G deviates from one (logG diagonal must be zero)

## Inputs

- Stationary diffusion matrix S from RWR (N×N), computed with p=0.9 on the kNN Markov transition matrix (k=10)
- Bhattacharyya kernel matrix G_ij = sqrt(<s_i, s_j>) with unit diagonal
- Target embedding dimension dim (2 or 3)

## Outputs

- Low-dimensional embedding matrix Yd (N×d, d=2 or 3), exported as bkfd_embedding.csv
- Retained positive singular values and singular vectors from SVD of logG

## How to apply

Starting from the stationary diffusion matrix S produced by random walk with restart (restart probability 1-p with p=0.9 on a kNN Markov transition matrix, k=10), construct the Bhattacharyya kernel matrix G with G_ij = sqrt(<s_i, s_j>) so each diagonal element equals one. Apply the element-wise logarithm to G to form logG, which defines a valid inner-product kernel via the mapping phi(s_i). Perform singular value decomposition logG = V Σ V^T and retain the largest d positive singular values (d=2 or d=3) to compute the low-dimensional embedding Yd = Vd Σd^(1/2), e.g., via DTFLOW's z.get_Y(slover='svd', dim). Save the resulting N×d matrix as bkfd_embedding.csv for downstream pseudotime ordering (z.get_dtflow_times) and branch detection (z.get_branches).

## Related tools

- **DTFLOW** (Provides the BKFD implementation, including get_diffusionRWR to produce S and get_Y(slover='svd', dim) to compute the SVD embedding) — https://github.com/statway/DTFLOW
- **Python** (Language in which the DTFLOW source code is distributed and executed (via Anaconda/Jupyter, e.g. guo2010.ipynb)) — https://github.com/statway/DTFLOW

## Examples

```
z = DTFLOW(raw_df, time_stages); M = z.get_transition_matrix(10); S = z.get_diffusionRWR(0.9); Y = z.get_Y(slover='svd', 2)
```

## Evaluation signals

- Bhattacharyya kernel matrix G has unit diagonal (G_ii = 1 for all i)
- logG has zero diagonal, consistent with ||y_i||^2 = log G_ii = 0
- All retained (top d) singular values are positive before forming Yd = Vd Σd^(1/2)
- Yd is an N×d matrix (d=2 or 3) whose row count matches the number of input cells
- Embedding reproduces the DTFLOW visualization when plotted via z.plotY(color='stages') and supports the reported Kendall rank correlations (e.g., 0.862 on the ME dataset with k=10, p=0.9)

## Limitations

- BKFD uses the same restart probability for all the nodes, and this may limit the effectiveness of random walk
- Parameter values (number of nearest neighbors k, restart probability 1-p) may vary from dataset to dataset and are not standardized; DTFLOW results were best with k=10 and p around 0.9
- Retaining only d=2 or d=3 dimensions may discard structure needed for complex, multi-branching trajectories

## Evidence

- [methods] RWR/SVD embedding via log kernel: [evidence span withheld — anchor status: not_found]
- [readme] SVD API call in README: "dimension reduction with SVD, and plot the result of dimension reduction Y = z.get_Y(slover='svd',dim)"
- [readme] RWR diffusion matrix input: "S = z.get_diffusionRWR(p) Here 1-p is the restart probability of RWR processes."
- [abstract] BKFD purpose: "a new method called Bhattacharyya kernel feature decomposition (BKFD) to reduce the data dimensions"
- [results] Parameter choice: "the ordering accuracy is better when the value of p is around 0.9. Thus, we use p=0.9 in this work"
- [discussion] Limitation on restart probability: "BKFD uses the same restart probability for all the nodes, and this may limit the effectiveness of random walk"
