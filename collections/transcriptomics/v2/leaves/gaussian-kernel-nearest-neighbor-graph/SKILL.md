---
name: gaussian-kernel-nearest-neighbor-graph
description: Use when a single-cell expression matrix (e.g. 438 cells x 48 genes)
  must be converted into a Markov transition matrix as input to diffusion-based manifold
  learning such as BKFD or diffusion pseudotime.
when_to_use_negative:
- The data are already reduced to approximate principal components or another low-dimensional
  embedding, so a raw-cell k-NN graph adds no diffusion information
- A branching/pseudotime ordering is needed but no graph-based diffusion representation
  is acceptable (e.g. when Monocle2-style independent trajectory inference is required
  for direct comparison)
- The expression matrix is not pre-processed — the article removes genes with corrupted
  names (e.g. 10 genes via Scanpy pre-processing) before building the graph
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: DTFLOW
  role: Python package whose get_transition_matrix(k) builds the k-NN Gaussian-kernel
    Markov transition matrix from the raw expression matrix
  repo: https://github.com/statway/DTFLOW
- name: Python
  role: Implementation language for loading the dataset with pandas and running the
    DTFLOW graph construction
  repo: https://github.com/statway/DTFLOW
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2020.08.003
    title: 'DTFLOW: Inference and Visualization of Single-Cell Pseudotime Trajectory
      Using Diffusion Propagation'
  source_content_hash: sha256:e772c0296a4e59f04af0766cd76b42ca469e913cdb9c9911aa1cb165cb3c9232
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/statway/DTFLOW
  tools:
  - DTFLOW
  - Python
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# Gaussian kernel nearest-neighbor graph

## Summary

Constructs a symmetric Gaussian-kernel affinity graph from k-nearest-neighbor cell-cell Euclidean distances and row-normalizes it into a Markov transition matrix, the first step of DTFLOW's BKFD diffusion-propagation pipeline for single-cell pseudotime.

## When to use

Use when a single-cell expression matrix (e.g. 438 cells x 48 genes) must be converted into a Markov transition matrix as input to diffusion-based manifold learning such as BKFD or diffusion pseudotime.

## When NOT to use

- The data are already reduced to approximate principal components or another low-dimensional embedding, so a raw-cell k-NN graph adds no diffusion information
- A branching/pseudotime ordering is needed but no graph-based diffusion representation is acceptable (e.g. when Monocle2-style independent trajectory inference is required for direct comparison)
- The expression matrix is not pre-processed — the article removes genes with corrupted names (e.g. 10 genes via Scanpy pre-processing) before building the graph

## Inputs

- Gene expression matrix X (N cells x D genes), e.g. the ME dataset of 438 single cells x 48 genes as a pandas DataFrame
- Time stage labels for each cell (e.g. the seven ME developmental stages from 1-cell zygote to 64-cell blastocyst)
- Number of nearest neighbors k (k=10 recommended by the article)

## Outputs

- Row-normalized Markov transition matrix M (N x N) with Gaussian-kernel affinities on the k-NN graph
- A DTFLOW object holding M, ready for the RWR diffusion step get_diffusionRWR(p)

## How to apply

Load the pre-processed gene expression matrix X (N cells x D genes, e.g. the ME high-throughput RT-PCR dataset with 438 cells and 48 genes) together with stage labels. Compute pairwise Euclidean cell-cell distances, connect each cell to its k nearest neighbors (the article finds k=10 works best on the ME dataset), symmetrize the adjacency, weight edges with a Gaussian kernel on the distances, and row-normalize so each row sums to 1, yielding the Markov transition matrix M. In DTFLOW this is done in one call, z.get_transition_matrix(k=10), on a DTFLOW object created as z = DTFLOW(raw_df, time_stages). The matrix M is then consumed by the random-walk-with-restart diffusion step (z.get_diffusionRWR(0.9)), so its row-stochasticity and correct neighborhood size directly determine downstream embedding quality.

## Related tools

- **DTFLOW** (Python package whose get_transition_matrix(k) builds the k-NN Gaussian-kernel Markov transition matrix from the raw expression matrix) — https://github.com/statway/DTFLOW
- **Python** (Implementation language for loading the dataset with pandas and running the DTFLOW graph construction) — https://github.com/statway/DTFLOW

## Examples

```
from DTFLOW import *; z = DTFLOW(raw_df, time_stages); M = z.get_transition_matrix(10)
```

## Evaluation signals

- Each row of the transition matrix M sums to 1 (row-stochastic Markov matrix)
- The graph is built on the k=10 nearest neighbors, the value the article found optimal on the ME dataset
- Downstream, the RWR diffusion matrix S is computed with restart probability p=0.9, and the resulting Kendall rank correlation of pseudotime on ME reaches 0.862 for DTFLOW (vs 0.761 for Monocle2)
- The 2-D BKFD embedding of the ME data places cells in correct chronological order of the seven stages (1-cell through 64-cell) and resolves the ICM intermediate stage

## Limitations

- Parameter values (Gaussian kernel, number of closest neighbors k, restart probability 1-p) may vary from dataset to dataset and are not standardized; k=10 and p=0.9 were tuned for the datasets tested
- The article notes no specific software versions are reported, so exact reproducibility of the graph construction may depend on the DTFLOW repository state
- No specific cutoff or standardized graph-density threshold exists; mis-setting k can degrade the neighborhood structure feeding the diffusion step

## Evidence

- [results] We test different values of k and find that the results are better if k=10: "We test different values of k and find that the results are better if k=10"
- [readme] readme transition matrix call: "Here k is number of each node's neighbors."
- [methods] A. Pre-process a single-cell dataset into a gene expression matrix XN×D with N cells and D genes.: "A. Pre-process a single-cell dataset into a gene expression matrix XN×D with N cells and D genes."
- [results] the ordering accuracy is better when the value of p is around 0.9. Thus, we use p=0.9 in this work: "the ordering accuracy is better when the value of p is around 0.9. Thus, we use p=0.9 in this work"
