---
name: bhattacharyya-kernel-feature-decomposition
description: Use when a single-cell gene expression matrix with developmental stage
  labels must be embedded in low dimensions for pseudotime trajectory visualization,
  e.g. ordering mouse embryo cells across the 1-cell to 64-cell stages.
when_to_use_negative:
- The goal is branch/bifurcation detection rather than dimensionality reduction —
  use the companion RSKG step (z.get_branches) instead.
- The input has not been pre-processed into a gene expression matrix (e.g. raw reads
  or a dataset needing corrupted-gene removal first).
- Cells lack any ordering or stage information needed to validate the chronological
  placement of the embedding.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: DTFLOW
  role: 'Python package implementing BKFD: get_transition_matrix(k), get_diffusionRWR(p),
    and get_Y(slover=''svd'', dim) execute the kNN/Markov matrix, RWR diffusion, and
    SVD embedding steps'
  repo: https://github.com/statway/DTFLOW
- name: Python
  role: Implementation language for DTFLOW and BKFD; source code freely available
    on GitHub
  repo: https://github.com/statway/DTFLOW
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2020.08.003
    title: 'DTFLOW: Inference and Visualization of Single-Cell Pseudotime Trajectory
      Using Diffusion Propagation'
  source_content_hash: sha256:fdd6bed998ed4732429bb29d3fe663919c7c3ee3bcde84535c1e2d4510f9c4ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/statway/DTFLOW
  tools:
  - DTFLOW
  - Python
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# bhattacharyya-kernel-feature-decomposition

## Summary

BKFD is the manifold-learning dimensionality-reduction method at the core of DTFLOW: it represents each cell by a stationary distribution from random walk with restart, forms a Bhattacharyya kernel similarity matrix, and recovers a low-dimensional embedding of single-cell developmental trajectories.

## When to use

Use when a single-cell gene expression matrix with developmental stage labels must be embedded in low dimensions for pseudotime trajectory visualization, e.g. ordering mouse embryo cells across the 1-cell to 64-cell stages.

## When NOT to use

- The goal is branch/bifurcation detection rather than dimensionality reduction — use the companion RSKG step (z.get_branches) instead.
- The input has not been pre-processed into a gene expression matrix (e.g. raw reads or a dataset needing corrupted-gene removal first).
- Cells lack any ordering or stage information needed to validate the chronological placement of the embedding.

## Inputs

- Pre-processed single-cell gene expression matrix X (N cells x D genes), e.g. the high-throughput RT-PCR mouse embryo (ME) dataset with 438 cells and 48 genes
- Vector of time/developmental stage labels for each cell (e.g. seven ME stages from 1-cell zygote to 64-cell blastocyst)
- Number of nearest neighbors k (article value: k=10)
- RWR restart probability parameter p (article value: p=0.9)
- Target embedding dimensionality dim (e.g. 2 for 2-D visualization)

## Outputs

- Markov transition matrix M (row-normalized Gaussian-kernel kNN affinities)
- Diffusion distribution matrix S of per-cell stationary distributions from random walk with restart
- Bhattacharyya kernel matrix G with elementwise log transform (logG)
- Low-dimensional embedding Y of the cells obtained by SVD of logG
- 2-D scatter plot of the embedding colored by stage labels (z.plotY)

## How to apply

Start from a preprocessed N-cell x D-gene expression matrix (e.g. the ME dataset with 438 cells x 48 genes). Build a k-nearest-neighbor graph (k=10 neighbors per node worked best in the article) from Euclidean cell-cell distances, symmetrize with a Gaussian kernel, and row-normalize to get the Markov transition matrix M via z.get_transition_matrix(k=10). Run random walk with restart at restart probability p=0.9 (accuracy was best when p was around 0.9) via z.get_diffusionRWR(0.9) to obtain one stationary distribution vector per cell. Form the Bhattacharyya kernel matrix G=<si,sj>, take its elementwise log, and perform singular value decomposition keeping the largest d singular values (d=2 for 2-D visualization) via z.get_Y(slover='svd', dim=2), then plot with z.plotY(color='stages').

## Related tools

- **DTFLOW** (Python package implementing BKFD: get_transition_matrix(k), get_diffusionRWR(p), and get_Y(slover='svd', dim) execute the kNN/Markov matrix, RWR diffusion, and SVD embedding steps) — https://github.com/statway/DTFLOW
- **Python** (Implementation language for DTFLOW and BKFD; source code freely available on GitHub) — https://github.com/statway/DTFLOW

## Examples

```
from DTFLOW import *; z = DTFLOW(raw_df, time_stages); M = z.get_transition_matrix(10); S = z.get_diffusionRWR(0.9); Y = z.get_Y(slover='svd', dim=2); z.plotY(size=(10,10), color='stages')
```

## Evaluation signals

- Embedding places cells in the correct chronological order of developmental stages; on the ME dataset DTFLOW achieved Kendall rank correlation 0.862 (vs 0.796 Scanpy, 0.761 Monocle2).
- The 2-D embedding resolves intermediate stages — for the ME dataset the inner cell mass (ICM) intermediate stage should be visibly separated.
- Matrix-shape invariants hold: M is row-normalized (rows sum to 1), S contains one stationary distribution vector per cell, and Y has dim columns equal to the requested embedding dimension.
- Best parameter regime reproduced: results improve with k=10 nearest neighbors and restart probability p around 0.9, as reported in the article.
- The pipeline runs end-to-end as in the guo2010.ipynb example: DTFLOW(raw_df, time_stages) -> get_transition_matrix -> get_diffusionRWR -> get_Y -> plotY.

## Limitations

- Parameter values (Gaussian kernel, number of neighbors k, restart probability) may vary from dataset to dataset and are not standardized.
- BKFD uses the same restart probability for all nodes, which may limit the effectiveness of the random walk.
- No specific software implementation versions are reported; only hardware (Lenovo ThinkPad P53, 2.6 GHz CPU) is mentioned.
- Expressing each cell by a distribution vector in a better way, and automatic parameter selection for optimal performance, remain open challenges.
- DTFLOW may only be used for technology research, not for commercial purposes (README license note).

## Evidence

- [abstract] a new method called Bhattacharyya kernel feature decomposition (BKFD) to reduce the data dimensions: "a new method called Bhattacharyya kernel feature decomposition (BKFD) to reduce the data dimensions"
- [results] We test different values of k and find that the results are better if k=10: "We test different values of k and find that the results are better if k=10"
- [results] the ordering accuracy is better when the value of p is around 0.9. Thus, we use p=0.9 in this work: "the ordering accuracy is better when the value of p is around 0.9. Thus, we use p=0.9 in this work"
- [results] The calculated Kendall rank correlation coefficients are 0.862, 0.796, 0.761 for DTFLOW, Scanpy, and Monocle2, respectively: "The calculated Kendall rank correlation coefficients are 0.862, 0.796, 0.761 for DTFLOW, Scanpy, and Monocle2, respectively"
- [readme] Y = z.get_Y(slover='svd',dim) z.plotY(size=(10,10),color ='stages'): "Y = z.get_Y(slover='svd',dim)
z.plotY(size=(10,10),color ='stages')"
- [discussion] BKFD uses the same restart probability for all the nodes, and this may limit the effectiveness of random walk: "BKFD uses the same restart probability for all the nodes, and this may limit the effectiveness of random walk"
