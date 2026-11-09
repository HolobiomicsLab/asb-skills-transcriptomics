---
name: root-cell-selection-in-trajectory-inference
description: Use when a single-cell trajectory method requires a biologically early
  reference cell (e.g., stem/progenitor or earliest time-stage cell) to orient pseudotime
  before branch detection with RSKG.
when_to_use_negative:
- When no biologically meaningful early reference cell or time stage exists and an
  unbiased trajectory orientation is required — the root choice directly determines
  pseudotime direction
- When the goal is the upstream diffusion embedding itself (BKFD/SVD dimension reduction),
  which does not require root cells
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DTFLOW
  role: Provides get_dtflow_times(root_list) to compute root-anchored pseudotime and
    get_branches(Num) for downstream RSKG branch detection
  repo: https://github.com/statway/DTFLOW
- name: Python
  role: Implementation language of the DTFLOW package in which root-based pseudotime
    is computed
  repo: https://github.com/statway/DTFLOW
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2020.08.003
    title: 'DTFLOW: Inference and Visualization of Single-Cell Pseudotime Trajectory
      Using Diffusion Propagation'
  source_content_hash: sha256:18a5ae2caeaff2ffe83ffc1a44196ff6b74ece67394e0140d36a5e9147c2fafe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/statway/DTFLOW
  tools:
  - DTFLOW
  - Python
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# root-cell-selection-in-trajectory-inference

## Summary

In DTFLOW, pseudotime is defined as the Bhattacharyya-kernel distance of every cell to one or more designated root cells, so choosing the root cell(s) anchors the direction of the inferred differentiation trajectory. This skill covers supplying root cell indices (root_list) and converting kernel distances into a normalized pseudotime ordering.

## When to use

Use when a single-cell trajectory method requires a biologically early reference cell (e.g., stem/progenitor or earliest time-stage cell) to orient pseudotime before branch detection with RSKG.

## When NOT to use

- When no biologically meaningful early reference cell or time stage exists and an unbiased trajectory orientation is required — the root choice directly determines pseudotime direction
- When the goal is the upstream diffusion embedding itself (BKFD/SVD dimension reduction), which does not require root cells

## Inputs

- Bhattacharyya kernel matrix logG (N x N) from the BKFD/RWR diffusion step
- List of root cell indices (root_list) marking biologically early cells; defaults to None in DTFLOW if not supplied
- Pre-processed gene expression matrix X(N x D) with time stage annotations (e.g., raw_df and time_stages)

## Outputs

- Normalized pseudotime vector T with values in [0, 1] per cell (dftimes from z.get_dtflow_times(root_list))
- Root-anchored cell ordering used to seed RSKG branch detection and pseudotime_branches.csv output

## How to apply

After BKFD diffusion (Markov transition matrix with k=10 neighbors and RWR with restart probability p=0.9), compute the pairwise cell-cell kernel distance D_ij = sqrt(-2*log(G_ij)) from the Bhattacharyya kernel matrix logG. Select root cell index/indices r from the earliest developmental stage of the dataset (e.g., progenitor cells in the mouse myeloid MARS-seq or early-stage cells in the mouse embryo RT-PCR data) and pass them as root_list. Compute pseudotime T as the row D_r,: of the kernel distance matrix — the distance of every cell to the root — or the sum over rows if multiple roots R are given. Min-max normalize T to [0, 1] so the root has pseudotime 0 and the most distant cell has pseudotime 1; the resulting ordering feeds the reverse-searching RSKG branch detection (z.get_branches(Num)), where cells are sorted by decreasing pseudotime.

## Related tools

- **DTFLOW** (Provides get_dtflow_times(root_list) to compute root-anchored pseudotime and get_branches(Num) for downstream RSKG branch detection) — https://github.com/statway/DTFLOW
- **Python** (Implementation language of the DTFLOW package in which root-based pseudotime is computed) — https://github.com/statway/DTFLOW

## Examples

```
from DTFLOW import *; z = DTFLOW(raw_df, time_stages); M = z.get_transition_matrix(10); S = z.get_diffusionRWR(0.9); dftimes = z.get_dtflow_times(root_list)
```

## Evaluation signals

- All pseudotime values lie within [0, 1] by construction of the min-max normalization (Equation 17)
- The designated root cell(s) receive the minimum pseudotime value (0) after normalization
- Script runs without error and downstream RSKG returns only sub-branches containing at least Num (minimum branch size) cells
- Root choice is consistent with known biology, e.g., progenitor cells of the MMP MARS-seq dataset or early-stage cells of the ME RT-PCR dataset
- Resulting ordering reproduces reported Kendall rank correlations (e.g., 0.862 on ME, 0.761 on MFG) when compared against known stages

## Limitations

- The default of root_list is None, so users must supply biologically justified root cells or accept DTFLOW's default behavior
- Parameter values (number of neighbors k, restart probability p) may vary from dataset to dataset and are not standardized
- Ordering accuracy depends on p; the authors found accuracy best when p is around 0.9 and used p=0.9 throughout

## Evidence

- [intro] dftimes = z.get_dtflow_times(root_list): "dftimes = z.get_dtflow_times(root_list)"
- [readme] here the default of root_list is none.: "here the default of root_list is none."
- [results] the ordering accuracy is better when the value of p is around 0.9. Thus, we use p=0.9 in this work: "the ordering accuracy is better when the value of p is around 0.9. Thus, we use p=0.9 in this work"
- [intro] Another important issue in single-cell studies is to identify branches in the pseudotime trajectories in order to explore the different developmental pathways.: "Another important issue in single-cell studies is to identify branches in the pseudotime trajectories in order to explore the different developmental pathways."
- [results] We test different values of k and find that the results are better if k=10: "We test different values of k and find that the results are better if k=10"
