---
name: random-walk-with-restart-diffusion
description: Use when a pre-processed N×D single-cell expression matrix must be converted
  into per-cell stationary diffusion distributions over a k-nearest-neighbor graph
  as a precursor to pseudotime ordering or BKFD dimensionality reduction.
when_to_use_negative:
- The input is raw, unprocessed counts — the article requires pre-processing into
  a gene expression matrix X (N×D) before transition-matrix construction
- The analysis goal is Scanpy DPT specifically and you intend to reproduce DPT exactly
  — RWR here is the more general algorithm of which DPT is the p=1 special case; using
  p=0.9 gives DTFLOW's setting, not DPT
- You need a principled, dataset-specific parameter selection — the article notes
  parameter values (Gaussian kernel width, k, restart probability 1-p) may vary from
  dataset to dataset and defers optimal selection to future work
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DTFLOW
  role: Provides get_transition_matrix(k) for the kNN Gaussian-kernel Markov matrix
    and get_diffusionRWR(p) for the stationary RWR diffusion distributions
  repo: https://github.com/statway/DTFLOW
- name: Python
  role: Implementation language; DTFLOW source is distributed as a Python package
    used via Anaconda/Jupyter (guo2010.ipynb example)
  repo: https://github.com/statway/DTFLOW
- name: Scanpy
  role: Reference diffusion pseudotime (DPT) implementation — the p=1 special case
    of RWR — used as a benchmark method on the same pre-processed matrix
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2020.08.003
    title: 'DTFLOW: Inference and Visualization of Single-Cell Pseudotime Trajectory
      Using Diffusion Propagation'
  source_content_hash: sha256:4b2e784e20dfb971600207bfa718df444943d726065c0817c78bcfb23e288475
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/statway/DTFLOW
  tools:
  - DTFLOW
  - Python
  - Scanpy
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# random-walk-with-restart-diffusion

## Summary

Builds a per-cell stationary diffusion distribution by running random walk with restart (RWR) over a kNN-derived Markov transition matrix, as implemented in DTFLOW's get_diffusionRWR(p) with restart probability 1-p and default p=0.9. This step is the diffusion-propagation core of DTFLOW pseudotime inference and the generalization of which diffusion pseudotime (DPT, Scanpy) is the p=1 special case.

## When to use

Use when a pre-processed N×D single-cell expression matrix must be converted into per-cell stationary diffusion distributions over a k-nearest-neighbor graph as a precursor to pseudotime ordering or BKFD dimensionality reduction.

## When NOT to use

- The input is raw, unprocessed counts — the article requires pre-processing into a gene expression matrix X (N×D) before transition-matrix construction
- The analysis goal is Scanpy DPT specifically and you intend to reproduce DPT exactly — RWR here is the more general algorithm of which DPT is the p=1 special case; using p=0.9 gives DTFLOW's setting, not DPT
- You need a principled, dataset-specific parameter selection — the article notes parameter values (Gaussian kernel width, k, restart probability 1-p) may vary from dataset to dataset and defers optimal selection to future work

## Inputs

- Pre-processed single-cell gene expression matrix X (N cells × D genes), e.g. the guo2010 ME RT-PCR dataset (438 cells × 48 genes) from github.com/statway/DTFLOW
- Number of nearest neighbors k (best result with k=10 per the paper)
- Restart-probability parameter p for get_diffusionRWR (p=0.9 used throughout the paper)

## Outputs

- Row-normalized Markov transition probability matrix M (N×N) from get_transition_matrix(k)
- N×N diffusion distribution matrix S of stationary RWR distributions (e.g. saved as diffusion_matrix_S.npy), with each row a probability distribution summing to 1

## How to apply

Load the pre-processed gene expression matrix X (N cells × D genes) into a DTFLOW object and build the Markov transition matrix via z.get_transition_matrix(k), finding the k=10 most similar neighbors (including the cell itself) by Euclidean kNN, converting distances to symmetric Gaussian kernel weights K(xi,xj)=exp(-||xi-xj||^2/(2*sigma_i*sigma_j)) with local width sigma_i set by the distance to the k-th nearest neighbor, then row-normalizing. Compute the stationary diffusion matrix via z.get_diffusionRWR(p), which evaluates S=(1-p)(I-pM)^(-1); the paper found ordering accuracy best around p=0.9 and uses p=0.9 throughout (note 1-p is the restart probability in the API). Save the resulting N×N matrix (e.g. diffusion_matrix_S.npy) and verify that every row of S is a non-vanishing probability distribution: s_ij>0 with each row summing to 1. Downstream, feed S into SVD-based BKFD embedding (z.get_Y(slover='svd',dim)) and pseudotime extraction (z.get_dtflow_times(root_list)).

## Related tools

- **DTFLOW** (Provides get_transition_matrix(k) for the kNN Gaussian-kernel Markov matrix and get_diffusionRWR(p) for the stationary RWR diffusion distributions) — https://github.com/statway/DTFLOW
- **Python** (Implementation language; DTFLOW source is distributed as a Python package used via Anaconda/Jupyter (guo2010.ipynb example)) — https://github.com/statway/DTFLOW
- **Scanpy** (Reference diffusion pseudotime (DPT) implementation — the p=1 special case of RWR — used as a benchmark method on the same pre-processed matrix)

## Examples

```
from DTFLOW import *; z = DTFLOW(raw_df, time_stages); M = z.get_transition_matrix(10); S = z.get_diffusionRWR(0.9); np.save('diffusion_matrix_S.npy', S)
```

## Evaluation signals

- Every row of the N×N diffusion matrix S is a non-vanishing probability distribution: s_ij>0 and each row sums to 1
- The closed-form result S=(1-p)(I-pM)^(-1) with p=0.9 matches the output of z.get_diffusionRWR(p) on the same transition matrix M
- Downstream Kendall rank correlation of DTFLOW pseudotime against known developmental stages is ≈0.862 on the ME dataset (vs 0.796 for Scanpy DPT and 0.761 for Monocle2)
- kNN graph uses k=10 neighbors including the cell itself, with Gaussian kernel symmetrized so K(xi,xj)=K(xj,xi), and M is row-normalized
- Matrix S is saved (e.g. diffusion_matrix_S.npy) with shape N×N matching the cell count of the input expression matrix

## Limitations

- Parameter values (Gaussian kernel width, number of neighbors k, restart probability 1-p) may vary from dataset to dataset and are not standardized; parameter selection for optimal performance is deferred to future work
- BKFD/RWR uses the same restart probability for all the nodes, which may limit the effectiveness of the random walk
- The method can only be used for technology research, not for commercial purposes (README license statement)

## Evidence

- [methods] kNN Markov + RWR workflow: [evidence span withheld — anchor status: not_found]
- [results] RWR closed form and default p: "the ordering accuracy is better when the value of p is around 0.9. Thus, we use p=0.9 in this work"
- [methods] RWR vs DPT relationship: "the RWR algorithm can be regarded as a more general approach and DPT is a special case of the RWR algorithm (i.e., p=1)"
- [readme] README API for the RWR step: "get one diffusion distribution matrix by the RWR processes. S = z.get_diffusionRWR(p) Here 1-p is the restart probability of RWR processes."
- [results] best k value: "We test different values of k and find that the results are better if k=10"
