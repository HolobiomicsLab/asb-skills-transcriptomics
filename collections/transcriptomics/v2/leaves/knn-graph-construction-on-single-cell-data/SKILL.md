---
name: knn-graph-construction-on-single-cell-data
description: Use when you have a pre-processed N-cell × D-gene expression matrix and
  need a cell-to-cell transition probability graph (Markov matrix) as input to random
  walk or diffusion-based trajectory methods.
when_to_use_negative:
- The data are raw counts that have not been pre-processed into a filtered expression
  matrix (DTFLOW requires a pre-processed X with N cells and D genes first)
- The downstream task needs gene-level features rather than cell-to-cell transition
  probabilities (this skill discards gene dimensions by design)
- You need trajectory branch detection directly (RSKG branch detection via z.get_branches(Num)
  is a separate skill step, not part of kNN graph construction)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DTFLOW
  role: Provides get_transition_matrix(k) to build the kNN Gaussian-kernel Markov
    transition matrix and get_diffusionRWR(p) to obtain the diffusion distribution
    matrix
  repo: https://github.com/statway/DTFLOW
- name: Python
  role: Implementation language of DTFLOW, used via Anaconda/Jupyter notebook (e.g.
    guo2010.ipynb)
  repo: https://github.com/statway/DTFLOW
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2020.08.003
    title: 'DTFLOW: Inference and Visualization of Single-Cell Pseudotime Trajectory
      Using Diffusion Propagation'
  source_content_hash: sha256:1e2d989a1c4ade6da8c5af5a892165b582c78fd525509b9415dc25be1c9e6c78
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/statway/DTFLOW
  tools:
  - DTFLOW
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# knn-graph-construction-on-single-cell-data

## Summary

Builds a k-nearest neighbor graph over single cells and converts it into a row-normalized Markov transition matrix (DTFLOW's get_transition_matrix), the substrate for RWR diffusion distributions in pseudotime inference.

## When to use

Use when you have a pre-processed N-cell × D-gene expression matrix and need a cell-to-cell transition probability graph (Markov matrix) as input to random walk or diffusion-based trajectory methods.

## When NOT to use

- The data are raw counts that have not been pre-processed into a filtered expression matrix (DTFLOW requires a pre-processed X with N cells and D genes first)
- The downstream task needs gene-level features rather than cell-to-cell transition probabilities (this skill discards gene dimensions by design)
- You need trajectory branch detection directly (RSKG branch detection via z.get_branches(Num) is a separate skill step, not part of kNN graph construction)

## Inputs

- Pre-processed single-cell gene expression matrix X (N cells × D genes)
- Number of nearest neighbors k (e.g. k=10, including the cell itself)

## Outputs

- Row-normalized N×N Markov transition probability matrix M
- N×N diffusion distribution matrix S from RWR (saved as diffusion_matrix_S.npy), with each row a probability distribution summing to 1

## How to apply

Start from the pre-processed gene expression matrix X (N cells × D genes), e.g. after Scanpy pre-processing. Compute pairwise Euclidean distances and take the k=10 most similar neighbors per cell, including the cell itself (k=10 was found optimal in DTFLOW's results). Transform distances into symmetric Gaussian kernel weights K(xi,xj)=exp(-||xi-xj||^2/(2*sigma_i*sigma_j)), with each local width sigma_i set to the distance from cell i to its k-th nearest neighbor, and symmetrize K(xi,xj)=K(xj,xi) for one-way neighbor pairs. Row-normalize the kernel to obtain the Markov transition probability matrix M (Equation 3), reproducing DTFLOW's z.get_transition_matrix(k). Downstream, feed M into the RWR step S=(1-p)(I-pM)^(-1) via z.get_diffusionRWR(p) with p=0.9 to get per-cell stationary diffusion distributions.

## Related tools

- **DTFLOW** (Provides get_transition_matrix(k) to build the kNN Gaussian-kernel Markov transition matrix and get_diffusionRWR(p) to obtain the diffusion distribution matrix) — https://github.com/statway/DTFLOW
- **Python** (Implementation language of DTFLOW, used via Anaconda/Jupyter notebook (e.g. guo2010.ipynb)) — https://github.com/statway/DTFLOW

## Examples

```
from DTFLOW import *; z = DTFLOW(raw_df, time_stages); M = z.get_transition_matrix(10); S = z.get_diffusionRWR(0.9)
```

## Evaluation signals

- Every row of the diffusion matrix S is a non-vanishing probability distribution: s_ij > 0 and each row sums to 1
- M is row-normalized: each row of the transition matrix sums to 1
- Kernel symmetry holds after symmetrization: K(xi,xj) = K(xj,xi) for all one-way neighbor pairs
- Each cell has exactly k=10 neighbors including itself in the kNN graph
- Resulting pseudotime ordering matches published benchmarks (e.g. Kendall rank correlation 0.862 for DTFLOW on the ME dataset, 0.761 on MFG)

## Limitations

- Parameter values (Gaussian kernel width, number of closest neighbors k, restart probability 1-p) may vary from dataset to dataset and are not standardized
- BKFD/RWR uses the same restart probability for all nodes, which may limit the effectiveness of the random walk
- DTFLOW is licensed for technology research only, not commercial purposes (per README)

## Evidence

- [readme] M= z.get_transition_matrix(k); Here k is number of each node's neighbors.: "M= z.get_transition_matrix(k)
``nHere k is number of each node's neighbors."
- [results] We test different values of k and find that the results are better if k=10: "We test different values of k and find that the results are better if k=10"
- [results] the ordering accuracy is better when the value of p is around 0.9. Thus, we use p=0.9 in this work: "the ordering accuracy is better when the value of p is around 0.9. Thus, we use p=0.9 in this work"
- [methods] A. Pre-process a single-cell dataset into a gene expression matrix XN×D with N cells and D genes.: "A. Pre-process a single-cell dataset into a gene expression matrix XN×D with N cells and D genes."
- [discussion] the values of these parameters may vary from dataset to dataset.: "the values of these parameters may vary from dataset to dataset."
