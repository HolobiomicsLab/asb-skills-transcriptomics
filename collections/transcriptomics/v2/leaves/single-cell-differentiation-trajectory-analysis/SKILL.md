---
name: single-cell-differentiation-trajectory-analysis
description: Use when single-cell gene expression data (e.g. ME, MMP, MFG, or MH datasets)
  must be ordered along pseudotime and multiple bifurcation events of cellular differentiation
  need to be detected.
when_to_use_negative:
- The majority of branching detection approaches can identify only one bifurcation
  point — but if your data truly has a single linear trajectory with no branching,
  a simpler pseudotime method suffices
- Input is already a pseudotime/trajectory annotation rather than a raw single-cell
  expression matrix
- Commercial use is intended — the DTFLOW source code 'only can be used for technology
  research, not for commercial purposes'
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DTFLOW
  role: 'Primary method: provides BKFD diffusion, pseudotime inference (get_dtflow_times),
    and RSKG branch detection (get_branches)'
  repo: https://github.com/statway/DTFLOW
- name: Python
  role: Implementation language; DTFLOW is used via a Python API in Jupyter/Anaconda
  repo: https://github.com/statway/DTFLOW
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2020.08.003
    title: 'DTFLOW: Inference and Visualization of Single-Cell Pseudotime Trajectory
      Using Diffusion Propagation'
  source_content_hash: sha256:109691b0b1248cc8c9ef83ab36203a67a1fa716e165a681a4a52bd081cc51472
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/statway/DTFLOW
  tools:
  - DTFLOW
  - Python
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# single-cell-differentiation-trajectory-analysis

## Summary

DTFLOW-based reconstruction of single-cell pseudotime trajectories: after BKFD diffusion with random walk with restart and a Bhattacharyya kernel, pseudotime is computed as kernel distance from root cells, and the Reverse Searching on kNN graph (RSKG) identifies multi-branching differentiation processes.

## When to use

Use when single-cell gene expression data (e.g. ME, MMP, MFG, or MH datasets) must be ordered along pseudotime and multiple bifurcation events of cellular differentiation need to be detected.

## When NOT to use

- The majority of branching detection approaches can identify only one bifurcation point — but if your data truly has a single linear trajectory with no branching, a simpler pseudotime method suffices
- Input is already a pseudotime/trajectory annotation rather than a raw single-cell expression matrix
- Commercial use is intended — the DTFLOW source code 'only can be used for technology research, not for commercial purposes'

## Inputs

- Raw single-cell gene expression matrix (N cells x D genes, e.g. ME: 438 cells x 48 genes; MH: 40,210 cells x 25,912 genes)
- Time stage labels of single cells (time_stages)
- List of root cell indices (root_list)
- Minimum number of cells required to form a sub-branch (Num)

## Outputs

- Normalized pseudotime vector per cell (values in [0,1])
- List of determined sub-branches from RSKG branch detection
- 2D diffusion map embedding Y from BKFD/SVD for visualization

## How to apply

Run the DTFLOW pipeline in Python: instantiate DTFLOW(raw_df, time_stages), compute the Markov transition matrix with k=10 nearest neighbors, run random walk with restart (restart probability 1-p, p=0.9) to get the diffusion distribution matrix S, and reduce dimensions with SVD. From the Bhattacharyya kernel matrix compute kernel distances D_ij = sqrt(-2*log(G_ij)), take each cell's distance to the root cell(s) as pseudotime, and min-max normalize it to [0,1]. Then run RSKG: starting from the cell with largest pseudotime, iteratively intersect each cell's kNN neighbor set with candidate prop-groups, promoting a group to a determined sub-branch once it reaches the minimum branch size Num, and assign ambiguous cells to the closer determined branch. Validate that pseudotime lies in [0,1] and every sub-branch contains at least Num cells.

## Related tools

- **DTFLOW** (Primary method: provides BKFD diffusion, pseudotime inference (get_dtflow_times), and RSKG branch detection (get_branches)) — https://github.com/statway/DTFLOW
- **Python** (Implementation language; DTFLOW is used via a Python API in Jupyter/Anaconda) — https://github.com/statway/DTFLOW

## Examples

```
z = DTFLOW(raw_df, time_stages); M = z.get_transition_matrix(k); S = z.get_diffusionRWR(p); dftimes = z.get_dtflow_times(root_list); branch = z.get_branches(Num)
```

## Evaluation signals

- All pseudotime values lie within [0,1] by construction of min-max normalization (Equation 17)
- Every returned sub-branch contains at least Num cells (the minimum branch size parameter)
- Kendall rank correlation between inferred pseudotime and known time stages reaches expected values (e.g. 0.862 on the ME dataset, 0.761 on the MFG dataset)
- The full script runs without error and z.get_dtflow_times(root_list) and z.get_branches(Num) both return results
- Restart probability p is set near 0.9 and k=10 nearest neighbors, matching the empirically optimal settings

## Limitations

- Parameter values (Gaussian kernel, number of closest neighbors k, restart probability 1-p) may vary from dataset to dataset and are not standardized
- BKFD uses the same restart probability for all the nodes, which may limit the effectiveness of random walk
- Parameter selection for optimal performance is deferred to future work
- Expressing each cell by a distribution vector in a better way remains an open challenge

## Evidence

- [abstract] RSKG branch detection rationale: "a novel approach named Reverse Searching on k-nearest neighbor graph (RSKG) to identify the multi-branching processes of cellular differentiation"
- [abstract] pseudotime via diffusion distribution: "we first establish a stationary distribution for each cell to represent the transition of cellular developmental states based on the random walk with restart algorithm"
- [results] optimal k parameter: "We test different values of k and find that the results are better if k=10"
- [results] restart probability choice: "the ordering accuracy is better when the value of p is around 0.9. Thus, we use p=0.9 in this work"
- [readme] README branch detection call: "here Num is the minimum number of cells required for forming one sub-branch."
- [readme] README license restriction: "The method only can be used for technology research, not for commercial purposes."
