---
name: knn-graph-branch-detection
description: Use when a pseudotime-ordered single-cell trajectory (e.g., DTFLOW pseudotime
  with root cell indices) must be split into sub-branches at bifurcation points, and
  existing tools detect only a single bifurcation.
when_to_use_negative:
- When only a single bifurcation point is expected and a simpler branching method
  suffices — RSKG targets multi-branching processes with multiple bifurcation events
- When the data are already a completed, independently validated branch/cluster assignment
  table rather than a pseudotime-ordered trajectory requiring branch discovery
- When no defensible root cell or root list can be supplied, since RSKG reverse-searches
  the kNN graph in decreasing pseudotime order anchored at root cells
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0085
tools:
- name: DTFLOW
  role: Provides the RSKG branch detection routine get_branches(Num) operating on
    the kNN graph and DTFLOW pseudotime
  repo: https://github.com/statway/DTFLOW
- name: Python
  role: Implementation language of DTFLOW used to run the RSKG workflow and write
    per-cell branch labels
  repo: https://github.com/statway/DTFLOW
- name: Monocle2
  role: External branch-assignment baseline used to compare against RSKG branches
    (e.g., on the MMP dataset)
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2020.08.003
    title: 'DTFLOW: Inference and Visualization of Single-Cell Pseudotime Trajectory
      Using Diffusion Propagation'
  source_content_hash: sha256:ff27641a066564330d983c5f69722f38031a6bf6523f8bf11bc5e38f1ef4d2e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/statway/DTFLOW
  tools:
  - DTFLOW
  - Monocle2
  - Python
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# knn-graph-branch-detection

## Summary

Implements DTFLOW's Reverse Searching on the k-nearest neighbor graph (RSKG) to identify multi-branching processes of cellular differentiation, including multiple bifurcation events, along a pseudotime trajectory.

## When to use

Use when a pseudotime-ordered single-cell trajectory (e.g., DTFLOW pseudotime with root cell indices) must be split into sub-branches at bifurcation points, and existing tools detect only a single bifurcation.

## When NOT to use

- When only a single bifurcation point is expected and a simpler branching method suffices — RSKG targets multi-branching processes with multiple bifurcation events
- When the data are already a completed, independently validated branch/cluster assignment table rather than a pseudotime-ordered trajectory requiring branch discovery
- When no defensible root cell or root list can be supplied, since RSKG reverse-searches the kNN graph in decreasing pseudotime order anchored at root cells

## Inputs

- Bhattacharyya kernel matrix logG from the BKFD/RWR diffusion step (z.get_diffusionRWR(p) with p=0.9)
- kNN graph adjacency structure A (k=10 nearest neighbors from the Markov transition matrix)
- Root cell index list (root_list) for pseudotime computation
- Minimum sub-branch size parameter Num (minimum number of cells required to form one sub-branch)

## Outputs

- List of determined sub-branches (per-cell branch assignments) from z.get_branches(Num)
- Normalized pseudotime vector T in [0, 1]
- Branch-labeled trajectory plot (z.plotY(color='branches'))
- Per-cell branch label table, e.g. branch_comparison.csv comparing RSKG branches to Monocle2 and Scanpy assignments

## How to apply

Load the Bhattacharyya kernel matrix logG and the kNN graph adjacency structure A (built with k=10 nearest neighbors, the value the paper reports as optimal) plus the root cell indices from the upstream BKFD diffusion step. Compute pairwise kernel distances D_ij = sqrt(-2*log(G_ij)), derive the pseudotime vector T as the distance of every cell to the root cell(s) and min-max normalize it to [0, 1] (Equation 17). Sort cells in decreasing pseudotime order and run RSKG: iteratively intersect each cell's neighbor set A_idj with candidate groups (prop-groups); when an intersected list reaches the minimum branch size n (the Num parameter of z.get_branches(Num)), promote it to a determined sub-branch, otherwise merge it as a new candidate group. Finally, assign remaining cells whose neighbor sets intersect both prop-groups and sub-branches to the closer determined branch. Validate that every returned sub-branch contains at least n cells and that the branches reproduce the known lineage structure (e.g., the erythroid/CMP/GMP/DC split on the 2730-cell MMP MARS-seq dataset).

## Related tools

- **DTFLOW** (Provides the RSKG branch detection routine get_branches(Num) operating on the kNN graph and DTFLOW pseudotime) — https://github.com/statway/DTFLOW
- **Python** (Implementation language of DTFLOW used to run the RSKG workflow and write per-cell branch labels) — https://github.com/statway/DTFLOW
- **Monocle2** (External branch-assignment baseline used to compare against RSKG branches (e.g., on the MMP dataset))

## Examples

```
z = DTFLOW(raw_df, time_stages); M = z.get_transition_matrix(10); S = z.get_diffusionRWR(0.9); dftimes = z.get_dtflow_times(root_list); branch = z.get_branches(Num)
```

## Evaluation signals

- Script runs without error and all normalized pseudotime values lie within [0, 1] by construction of Equation (17)
- Every returned sub-branch contains at least Num cells (the minimum sub-branch size)
- On the MMP dataset (2730 cells, 3451 informative genes), RSKG branches reproduce the erythroid/CMP/GMP/DC lineage split across the 19 known progenitor clusters
- Detected branches align with the branch assignments reported for Monocle2 and Scanpy, noting Scanpy's root cell may be developmentally unreasonable on MMP
- Branch-labeled visualization z.plotY(color='branches') shows distinct terminal sub-branches at expected bifurcation points

## Limitations

- Parameter values (Gaussian kernel, number of closest neighbors k, restart probability 1-p) may vary from dataset to dataset and are not standardized; k=10 and p=0.9 were the paper's optimal choices
- RSKG requires a minimum sub-branch size Num; too large a value merges small terminal sub-branches, too small a value splits noise
- In the MMP comparison, Scanpy identified four matching sub-branches but its root cell was not in the initial group, which is developmentally unreasonable — root placement is a known failure mode in branch detection

## Evidence

- [abstract] a novel approach named Reverse Searching on k-nearest neighbor graph (RSKG) to identify the multi-branching processes of cellular differentiation: "a novel approach named Reverse Searching on k-nearest neighbor graph (RSKG) to identify the multi-branching processes of cellular differentiation"
- [methods] RSKG description: [evidence span withheld — anchor status: not_found]
- [results] k and p parameter choice: "We test different values of k and find that the results are better if k=10"
- [readme] README get_branches invocation: "here Num is the minimum number of cells required for forming one sub-branch."
- [results] MMP Scanpy comparison finding: [evidence span withheld — anchor status: not_found]
