---
name: boundary-condition-handling-density-estimation
description: Use when a nx1 vector of pseudotime values and a nx1 vector of marker
  signals from n single cells must be turned into a joint density p_s(s,y) on a regular
  grid as MAPiT input, especially when domain edges distort plain KDE.
when_to_use_negative:
- No pseudotemporal ordering of the cells exists yet — step 1 of the MAPiT workflow
  (generate pseudotime with Wanderlust, DPT or another algorithm) must be done first,
  since this skill consumes its nx1 output vector
- The real-scale (true-scale) cell distribution required by MAPiT cannot be defined,
  e.g. for processes like stem cell differentiation where the target distribution
  construction is acknowledged to be more challenging
- The goal is the downstream transformation itself rather than density estimation
  — use preMAPiT.m/MAPiT.m instead, which take the output of this step
tools:
- name: Kernel Density Estimation with Linked BCs
  role: Provides the linked-boundary-condition KDE (Cts_samples/Cts_func, Discrete_linked)
    used to obtain the distribution in pseudotime for the cell cycle example
  repo: https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git
- name: jointDensityPseudotimeY.m
  role: MAPiT function that takes the nx1 pseudotime vector and nx1 marker-signal
    vector and returns the joint density of pseudotime and markers as MAPiT input
  repo: https://github.com/karstenkuritz/MAPiT
- name: MATLAB
  role: Environment in which the KDE scripts are run; MAPiT consists of Matlab scripts
    adapted and called within a Matlab session per application case
  repo: https://github.com/karstenkuritz/MAPiT
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-60400-z
    title: Reconstructing temporal and spatial dynamics from single-cell pseudotime
      using prior knowledge of real scale cell densities
  source_content_hash: sha256:cb385beaf061f4525c530c09261b384b21b6429d778ed84678a6dab8cfd88ab5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git
  tools:
  - Kernel Density Estimation with Linked BCs
  - MATLAB
  - jointDensityPseudotimeY.m
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# boundary-condition-handling-density-estimation

## Summary

Estimates the joint density p_s(s,y) of pseudotime and marker signal in MATLAB by Gaussian kernel density estimation with boundary-aware smoothing, so that MAPiT's measure-preserving transformation receives a properly normalized density even at domain edges (spheroid surface, cell-cycle start/end).

## When to use

Use when a nx1 vector of pseudotime values and a nx1 vector of marker signals from n single cells must be turned into a joint density p_s(s,y) on a regular grid as MAPiT input, especially when domain edges distort plain KDE.

## When NOT to use

- No pseudotemporal ordering of the cells exists yet — step 1 of the MAPiT workflow (generate pseudotime with Wanderlust, DPT or another algorithm) must be done first, since this skill consumes its nx1 output vector
- The real-scale (true-scale) cell distribution required by MAPiT cannot be defined, e.g. for processes like stem cell differentiation where the target distribution construction is acknowledged to be more challenging
- The goal is the downstream transformation itself rather than density estimation — use preMAPiT.m/MAPiT.m instead, which take the output of this step

## Inputs

- nx1 vector of pseudotime values from a trajectory-inference algorithm (Wanderlust/Cyt3 or Diffusion Maps/DPT)
- nx1 vector of marker signals from n single cells, imported into MATLAB e.g. from a .csv file
- Choice of boundary conditions: reflecting (spheroid/spatial application) or linked (cell-cycle application)

## Outputs

- Joint density p_s(s,y) on a regular grid over pseudotime and marker intensity
- MATLAB data file joint_density_pseudotime_marker.mat for consumption by preMAPiT.m
- Marginal density over pseudotime used for the normalization/validation check

## How to apply

Import the nx1 pseudotime vector (from Wanderlust via the Cyt3 toolbox or from Diffusion Maps/DPT) together with the nx1 marker-signal vector into MATLAB, e.g. from a .csv file. Call jointDensityPseudotimeY.m from the MAPiT repository to compute the joint density of pseudotime and marker signal using Gaussian kernel density estimation, applying Silverman's-rule bandwidths in both the pseudotime and the marker dimension. Choose the boundary treatment by application: for the spheroid (spatial distance-from-surface) application use KDE with reflecting boundary conditions at the domain edges; for the cell-cycle application use KDE with linked boundary conditions via the MColbrook/Kernel-Density-Estimation-with-Linked-BCs code (function Cts_samples takes a vector D of samples). Return p_s(s,y) on a regular grid over pseudotime and marker intensity and save it as joint_density_pseudotime_marker.mat. Before passing the result to preMAPiT.m, verify that the joint density integrates to approximately one over its support and that the marginal density over pseudotime reproduces the input cell distribution.

## Related tools

- **Kernel Density Estimation with Linked BCs** (Provides the linked-boundary-condition KDE (Cts_samples/Cts_func, Discrete_linked) used to obtain the distribution in pseudotime for the cell cycle example) — https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git
- **jointDensityPseudotimeY.m** (MAPiT function that takes the nx1 pseudotime vector and nx1 marker-signal vector and returns the joint density of pseudotime and markers as MAPiT input) — https://github.com/karstenkuritz/MAPiT
- **MATLAB** (Environment in which the KDE scripts are run; MAPiT consists of Matlab scripts adapted and called within a Matlab session per application case) — https://github.com/karstenkuritz/MAPiT

## Examples

```
p = jointDensityPseudotimeY(pseudotime, marker);  % nx1 vectors imported from .csv; returns joint density p_s(s,y) for preMAPiT.m
```

## Evaluation signals

- The estimated joint density integrates to approximately one over its support (normalization check)
- The marginal density over pseudotime reproduces the input cell distribution before being passed to preMAPiT.m
- Edge behavior matches the chosen boundary scheme: no spurious mass leakage at spheroid-domain edges with reflecting BCs, and periodic/linked consistency at cell-cycle boundaries
- Output p_s(s,y) is defined on a regular grid over both pseudotime and marker intensity and saved as joint_density_pseudotime_marker.mat
- preMAPiT.m accepts the saved .mat without shape errors (input was nx1 pseudotime and nx1 marker vectors from the same n cells)

## Limitations

- MAPiT is reliant on the correctness of the pseudotemporal ordering provided by the trajectory-inference algorithm, so density estimation on a wrong ordering propagates the error
- The boundary-condition scheme must match the application (reflecting for spheroid spatial scale, linked for cell-cycle); applying the wrong scheme distorts edge density
- Downstream MAPiT use additionally requires a defensible true-scale distribution; for processes like stem cell differentiation this may need cell death and cell division rates at every stage, which are not provided

## Evidence

- [readme] linked BCs used for cell-cycle pseudotime distribution: "kernel density estmation with linked boundary conditions (https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git) is used to obtain distribution in pseudotime"
- [methods] jointDensityPseudotimeY.m input contract: "The function [`jointDensityPseudotimeY.m`](jointDensityPseudotimeY.m) takes a `nx1` vector of pseudotime values from the pseudotime algorithm"
- [intro] dependence on correct cell ordering: "The method makes use of pseudotemporal ordering obtained from trajectory inference algorithms and hence is reliant on the correctness of the provided cell order."
