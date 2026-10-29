---
name: diffusion-embedding-visualization
description: Use when a labelled single-cell expression matrix needs a low-dimensional
  diffusion-based embedding that must preserve chronological stage order (e.g. 1-cell
  through 64-cell ME stages) or resolve branching intermediates visually.
when_to_use_negative:
- When the goal is branch/pseudotime inference rather than visualization of stage
  ordering — DTFLOW provides separate steps (z.get_dtflow_times, z.get_branches) not
  covered by this embedding skill
- When the dataset has no stage/time labels or cells are not expected to lie on a
  developmental continuum, since validation of the embedding depends on correct chronological
  placement
- When a different manifold learning method (e.g. Monocle2's embedding) is required
  for the deliverable, since only DTFLOW was shown to resolve the ICM intermediate
  stage
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DTFLOW
  role: 'Python package providing the BKFD pipeline: transition matrix, RWR diffusion,
    SVD embedding, and stage-colored plotting (z.get_transition_matrix, z.get_diffusionRWR,
    z.get_Y, z.plotY)'
  repo: https://github.com/statway/DTFLOW
- name: Python
  role: Implementation language for DTFLOW; pandas loads the expression matrix and
    matplotlib renders the 2-D embedding
  repo: https://github.com/statway/DTFLOW
- name: Monocle2
  role: 'Baseline comparison: its embedding of the same cells is overlaid to check
    whether the ICM intermediate stage is resolved'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2020.08.003
    title: 'DTFLOW: Inference and Visualization of Single-Cell Pseudotime Trajectory
      Using Diffusion Propagation'
  source_content_hash: sha256:9cd39f641be9a05a337b2d835d70d2d34a5e0a13105b6eee655cdfaf445b76e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/statway/DTFLOW
  tools:
  - DTFLOW
  - Monocle2
  - Python
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# diffusion-embedding-visualization

## Summary

Constructs a 2-D diffusion embedding of single-cell data using DTFLOW's Bhattacharyya kernel feature decomposition (BKFD): a k-NN Markov transition matrix, random walk with restart diffusion, log-Bhattacharyya kernel SVD, and a matplotlib stage-colored plot. Applied to recover developmental stage ordering (e.g. the 438-cell x 48-gene mouse embryo RT-PCR dataset) and intermediate states such as the ICM.

## When to use

Use when a labelled single-cell expression matrix needs a low-dimensional diffusion-based embedding that must preserve chronological stage order (e.g. 1-cell through 64-cell ME stages) or resolve branching intermediates visually.

## When NOT to use

- When the goal is branch/pseudotime inference rather than visualization of stage ordering — DTFLOW provides separate steps (z.get_dtflow_times, z.get_branches) not covered by this embedding skill
- When the dataset has no stage/time labels or cells are not expected to lie on a developmental continuum, since validation of the embedding depends on correct chronological placement
- When a different manifold learning method (e.g. Monocle2's embedding) is required for the deliverable, since only DTFLOW was shown to resolve the ICM intermediate stage

## Inputs

- Single-cell gene expression matrix (N cells x D genes) as a pandas DataFrame, e.g. the ME high-throughput RT-PCR dataset with 438 cells and 48 genes
- Time stage labels for each cell (the seven developmental stages from 1-cell zygote to 64-cell blastocyst)
- Parameters: number of nearest neighbors k (e.g. 10), restart probability 1-p (e.g. p=0.9), target embedding dimension dim (e.g. 2)

## Outputs

- 2-D BKFD embedding Y (one low-dimensional coordinate pair per cell)
- 2-D matplotlib scatter plot colored by developmental stage (z.plotY color='stages')
- Optional comparison overlay against a Monocle2 embedding of the same cells to assess ICM separation

## How to apply

Load the single-cell dataset (e.g. the ME high-throughput RT-PCR matrix of 438 cells x 48 genes) with stage labels into a pandas DataFrame and instantiate a DTFLOW object with the time stages. Build the k-nearest-neighbor graph (k=10 was optimal in the paper) from Euclidean cell-cell distances, symmetrize with a Gaussian kernel, and row-normalize to obtain the Markov transition matrix M via z.get_transition_matrix(10). Run random walk with restart at restart probability 1-p with p=0.9 (z.get_diffusionRWR(0.9)) to get one stationary distribution vector per cell, then form the Bhattacharyya kernel G=<si,sj>, take its elementwise log, and keep the d=2 largest singular values via SVD to obtain the embedding Y = z.get_Y(slover='svd', dim=2). Plot with z.plotY(size=(10,10), color='stages'), coloring cells by developmental stage, and optionally overlay a Monocle2 embedding of the same cells to check whether intermediate states such as the inner cell mass (ICM) are separated.

## Related tools

- **DTFLOW** (Python package providing the BKFD pipeline: transition matrix, RWR diffusion, SVD embedding, and stage-colored plotting (z.get_transition_matrix, z.get_diffusionRWR, z.get_Y, z.plotY)) — https://github.com/statway/DTFLOW
- **Python** (Implementation language for DTFLOW; pandas loads the expression matrix and matplotlib renders the 2-D embedding) — https://github.com/statway/DTFLOW
- **Monocle2** (Baseline comparison: its embedding of the same cells is overlaid to check whether the ICM intermediate stage is resolved)

## Examples

```
z = DTFLOW(raw_df, time_stages); M = z.get_transition_matrix(10); S = z.get_diffusionRWR(0.9); Y = z.get_Y(slover='svd', dim=2); z.plotY(size=(10,10), color='stages')
```

## Evaluation signals

- Cells in the embedding must appear in the correct chronological order of the seven ME stages (1-cell to 64-cell); DTFLOW achieved Kendall rank correlation 0.862 on ME vs 0.761 for Monocle2
- The embedding should be computed with k=10 neighbors and restart probability p=0.9, the values reported as optimal in the paper
- Only the DTFLOW (BKFD) embedding should visibly resolve the ICM intermediate developmental stage; if the ICM merges with adjacent stages, parameters or graph construction should be revisited
- Y must have exactly dim=2 columns after SVD of the log-Bhattacharyya kernel matrix G=<si,sj>
- The plotted figure (size=(10,10)) must color cells by stage labels and separate all seven labelled stages

## Limitations

- Parameter values (Gaussian kernel width, number of closest neighbors k, restart probability 1-p) may vary from dataset to dataset and are not standardized
- BKFD uses the same restart probability for all nodes, which may limit the effectiveness of random walk
- Expressing each cell by a distribution vector in a better way remains an open challenge for future study

## Evidence

- [results] Kendall coefficients ME: "The calculated Kendall rank correlation coefficients are 0.862, 0.796, 0.761 for DTFLOW, Scanpy, and Monocle2, respectively"
- [results] restart probability p: "the ordering accuracy is better when the value of p is around 0.9. Thus, we use p=0.9 in this work"
- [results] k nearest neighbors: "We test different values of k and find that the results are better if k=10"
- [readme] SVD embedding call: "Y = z.get_Y(slover='svd',dim)
z.plotY(size=(10,10),color ='stages')"
- [methods] ME dataset description: "The first dataset is the high-throughput RT-PCR dataset [42] that describes the early-stages of the developmental process for mouse embryo (ME)."
