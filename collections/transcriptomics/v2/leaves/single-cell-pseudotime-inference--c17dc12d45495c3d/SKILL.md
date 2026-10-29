---
name: single-cell-pseudotime-inference
description: Use when a single-cell gene expression matrix (cells × genes) must be
  ordered along a developmental trajectory, visualized in 2D, or split into branching
  differentiation paths.
when_to_use_negative:
- When only one bifurcation event must be found and a simpler branch detector suffices
  — RSKG is aimed at multi-branching processes with multiple bifurcation points
- When the input is raw sequencing data rather than a pre-processed gene expression
  matrix — the article requires 'A. Pre-process a single-cell dataset into a gene
  expression matrix XN×D with N cells and D genes' first
- When the dataset requires per-node restart probabilities or adaptive parameter tuning
  — BKFD uses the same restart probability for all nodes and parameters may vary from
  dataset to dataset without a standardized selection procedure
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: DTFLOW
  role: 'Primary Python implementation: DTFLOW object, get_transition_matrix, get_diffusionRWR,
    get_Y (BKFD SVD), get_dtflow_times, get_branches'
  repo: https://github.com/statway/DTFLOW
- name: Scanpy
  role: Baseline comparator (DPT implementation, the p=1 special case of RWR) and
    pre-processing (e.g., removal of 10 genes with corrupted names in the MMP dataset)
- name: Monocle2
  role: R baseline pseudotime method compared against DTFLOW by Kendall rank correlation
    and ICM-stage resolution
- name: Python
  role: Runtime environment (Anaconda/Jupyter) in which DTFLOW is executed; source
    code is Python
  repo: https://github.com/statway/DTFLOW
provenance:
  source_task_ids:
  - task_004
  - task_001
  - task_003
  - task_002
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2020.08.003
    title: 'DTFLOW: Inference and Visualization of Single-Cell Pseudotime Trajectory
      Using Diffusion Propagation'
  source_content_hash: sha256:87fd9255bbe3a05e5965c7b8b05096202302c2efc4d771d5c17749df34a6fc00
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/statway/DTFLOW
  tools:
  - DTFLOW
  - Monocle2
  - Python
  - Scanpy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# single-cell-pseudotime-inference

## Summary

Infers single-cell pseudotime trajectories using DTFLOW: a kNN Markov graph, random walk with restart diffusion, Bhattacharyya kernel feature decomposition (BKFD) for 2D embedding, and root-distance pseudotime, optionally with RSKG branch detection.

## When to use

Use when a single-cell gene expression matrix (cells × genes) must be ordered along a developmental trajectory, visualized in 2D, or split into branching differentiation paths.

## When NOT to use

- When only one bifurcation event must be found and a simpler branch detector suffices — RSKG is aimed at multi-branching processes with multiple bifurcation points
- When the input is raw sequencing data rather than a pre-processed gene expression matrix — the article requires 'A. Pre-process a single-cell dataset into a gene expression matrix XN×D with N cells and D genes' first
- When the dataset requires per-node restart probabilities or adaptive parameter tuning — BKFD uses the same restart probability for all nodes and parameters may vary from dataset to dataset without a standardized selection procedure

## Inputs

- Gene expression matrix X (N cells × D genes), e.g. pandas DataFrame from the guo2010 ME RT-PCR dataset (438 cells, 48 genes) or MMP MARS-seq dataset (2730 cells, 3451 genes)
- Time stage labels for each cell (e.g., seven ME developmental stages, 1-cell zygote to 64-cell blastocyst)
- Root cell index or list of root indices for pseudotime origin
- Parameters: number of neighbors k=10, restart probability 1-p (p=0.9), minimum sub-branch size Num

## Outputs

- Markov transition matrix M and diffusion distribution matrix S (one stationary distribution vector per cell)
- Low-dimensional BKFD embedding Y (SVD, dim=2) for trajectory visualization
- Pseudotime value in [0,1] for each cell (dftimes)
- List of determined sub-branches from RSKG branch detection
- Comparison statistics such as Kendall rank correlation of inferred vs. known stage ordering (e.g., kendall_correlations.csv)

## How to apply

Load the cell-by-gene matrix (e.g., the guo2010 ME RT-PCR dataset: 438 cells × 48 genes) and stage labels into a DTFLOW object via `z = DTFLOW(raw_df, time_stages)`. Build the Markov transition matrix with k=10 nearest neighbors (`z.get_transition_matrix(10)`) — the value the paper found optimal — then run random walk with restart at p=0.9 (`z.get_diffusionRWR(0.9)`) to obtain one stationary distribution per cell (DPT with p=1 is the special case of this RWR). Take the log of the Bhattacharyya kernel matrix G=<si,sj> and SVD-reduce to d=2 dimensions (`z.get_Y(slover='svd', dim=2)`), then compute pseudotime as the min-max-normalized kernel distance of each cell to the root cell(s) via `z.get_dtflow_times(root_list)`. For multi-branch processes, run RSKG (`z.get_branches(Num)`) by reverse-searching the kNN graph in decreasing pseudotime order, merging/splitting candidate groups against the minimum branch size Num.

## Related tools

- **DTFLOW** (Primary Python implementation: DTFLOW object, get_transition_matrix, get_diffusionRWR, get_Y (BKFD SVD), get_dtflow_times, get_branches) — https://github.com/statway/DTFLOW
- **Scanpy** (Baseline comparator (DPT implementation, the p=1 special case of RWR) and pre-processing (e.g., removal of 10 genes with corrupted names in the MMP dataset))
- **Monocle2** (R baseline pseudotime method compared against DTFLOW by Kendall rank correlation and ICM-stage resolution)
- **Python** (Runtime environment (Anaconda/Jupyter) in which DTFLOW is executed; source code is Python) — https://github.com/statway/DTFLOW

## Examples

```
from DTFLOW import *; z = DTFLOW(raw_df, time_stages); M = z.get_transition_matrix(10); S = z.get_diffusionRWR(0.9); Y = z.get_Y(slover='svd', dim=2); dftimes = z.get_dtflow_times(root_list)
```

## Evaluation signals

- Kendall rank correlation of inferred pseudotime vs. known stage ordering should approach the paper's values on ME: 0.862 (DTFLOW) vs. 0.796 (Scanpy) and 0.761 (Monocle2)
- The 2D BKFD embedding must place cells in the correct chronological order of the seven ME developmental stages (1-cell through 64-cell) and resolve the ICM intermediate state
- All normalized pseudotime values lie within [0,1] by construction of the min-max normalization (Equation 17)
- Every RSKG sub-branch contains at least Num cells (the minimum number required for forming one sub-branch)
- Under 50-times-repeated 90% subsampling of ME, mean Spearman correlation to the full-data ordering is high with variance smaller than Scanpy's (stability check)

## Limitations

- Parameter values (Gaussian kernel, number of closest neighbors k, restart probability 1-p) may vary from dataset to dataset and are not standardized; parameter selection for optimal performance is deferred to future work
- BKFD uses the same restart probability for all nodes, which may limit the effectiveness of random walk
- Expressing each cell by a distribution vector in a better way remains an open challenge for future study
- No specific software versions are reported; the DTFLOW method 'only can be used for technology research, not for commercial purposes' (README license note)

## Evidence

- [results] The calculated Kendall rank correlation coefficients are 0.862, 0.796, 0.761 for DTFLOW, Scanpy, and Monocle2, respectively: "The calculated Kendall rank correlation coefficients are 0.862, 0.796, 0.761 for DTFLOW, Scanpy, and Monocle2, respectively"
- [results] We test different values of k and find that the results are better if k=10: "We test different values of k and find that the results are better if k=10"
- [methods] the RWR algorithm can be regarded as a more general approach and DPT is a special case of the RWR algorithm (i.e., p=1): "the RWR algorithm can be regarded as a more general approach and DPT is a special case of the RWR algorithm (i.e., p=1)"
- [abstract] a new method called Bhattacharyya kernel feature decomposition (BKFD) to reduce the data dimensions: "a new method called Bhattacharyya kernel feature decomposition (BKFD) to reduce the data dimensions"
- [readme] z = DTFLOW(raw_df,time_stages): "z = DTFLOW(raw_df,time_stages)"
- [results] the ordering accuracy is better when the value of p is around 0.9. Thus, we use p=0.9 in this work: "the ordering accuracy is better when the value of p is around 0.9. Thus, we use p=0.9 in this work"
