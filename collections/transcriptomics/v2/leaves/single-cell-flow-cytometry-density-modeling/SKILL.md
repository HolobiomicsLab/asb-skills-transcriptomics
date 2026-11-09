---
name: single-cell-flow-cytometry-density-modeling
description: Use when you have an nx1 vector of pseudotime values (from Wanderlust,
  DPT/Diffusion Maps, etc.) and an nx1 vector of marker signals from n single cells
  and need the joint density p_s(s,y) on a regular grid as MAPiT input.
when_to_use_negative:
- You already have the joint density p_s(s,y) and only need the transformation or
  real-scale profiles - use preMAPiT.m and MAPiT.m instead of re-estimating the density
- No defensible target distribution on the real scale can be constructed (e.g. stem-cell
  differentiation processes where cell death and division rates at every stage are
  unknown) - the downstream MAPiT transformation would not be valid
- The cell order from the trajectory-inference algorithm is untrustworthy - MAPiT
  is reliant on the correctness of the provided cell order, so the density would encode
  a flawed ordering
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3308
tools:
- name: MATLAB
  role: Host environment in which jointDensityPseudotimeY.m performs Gaussian kernel
    density estimation of the joint pseudotime-marker density
  repo: https://github.com/karstenkuritz/MAPiT
- name: Kernel Density Estimation with Linked BCs
  role: Provides KDE with linked boundary conditions used to obtain the distribution
    in pseudotime for the cell-cycle example
  repo: https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git
- name: Wanderlust
  role: Upstream algorithm (via the Cyt3 toolbox) that produces the pseudotime values
    fed into the density estimation
  repo: https://github.com/dpeerlab/cyt3
- name: Diffusion Maps
  role: Alternative upstream algorithm producing pseudotime values from single-cell
    experimental data for density estimation
  repo: https://www.helmholtz-muenchen.de/icb/research/groups/machine-learning/projects/dpt/index.html
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1038/s41598-020-60400-z
    title: Reconstructing temporal and spatial dynamics from single-cell pseudotime
      using prior knowledge of real scale cell densities
  source_content_hash: sha256:d427ef0be1f8e6de07e29c789ffef364b5341137ae73482da5ad4c3e20f1c760
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/karstenkuritz/MAPiT
  tools:
  - Diffusion Maps
  - Kernel Density Estimation with Linked BCs
  - MAPiT.m
  - MATLAB
  - Wanderlust
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# single-cell-flow-cytometry-density-modeling

## Summary

Estimates the joint density p_s(s,y) of pseudotime and single-cell marker signals (e.g. DNA, Ki-67, geminin from flow cytometry) by Gaussian kernel density estimation, providing the pseudotime-scale distribution that the MAPiT measure-preserving transformation requires.

## When to use

Use when you have an nx1 vector of pseudotime values (from Wanderlust, DPT/Diffusion Maps, etc.) and an nx1 vector of marker signals from n single cells and need the joint density p_s(s,y) on a regular grid as MAPiT input.

## When NOT to use

- You already have the joint density p_s(s,y) and only need the transformation or real-scale profiles - use preMAPiT.m and MAPiT.m instead of re-estimating the density
- No defensible target distribution on the real scale can be constructed (e.g. stem-cell differentiation processes where cell death and division rates at every stage are unknown) - the downstream MAPiT transformation would not be valid
- The cell order from the trajectory-inference algorithm is untrustworthy - MAPiT is reliant on the correctness of the provided cell order, so the density would encode a flawed ordering

## Inputs

- nx1 vector of pseudotime values from a trajectory-inference algorithm (e.g. Wanderlust via the Cyt3 toolbox or Diffusion Maps), importable from a .csv file
- nx1 vector of marker signal intensities from n single cells (e.g. flow cytometry measurements of DNA, RNA, Ki-67, p27, or mAG-hGeminin (1-110))
- choice of boundary-condition mode: reflecting BCs (spheroid/spatial case) or linked BCs (cell-cycle case)

## Outputs

- Joint density p_s(s,y) of pseudotime and marker signal on a regular grid, normalized to integrate to approximately one
- joint_density_pseudotime_marker.mat (MATLAB .mat file) passed downstream to preMAPiT.m

## How to apply

Import the nx1 pseudotime vector from the trajectory-inference algorithm and the nx1 marker-signal vector (e.g. from a .csv file) into MATLAB. Estimate the joint density with jointDensityPseudotimeY.m using Gaussian kernel density estimation and Silverman's-rule bandwidths in both the pseudotime and marker dimensions. Choose boundary conditions by application: reflecting boundary conditions at the domain edges for the spheroid (spatial, distance-from-surface) case, and linked boundary conditions (MColbrook/Kernel-Density-Estimation-with-Linked-BCs, used in the cell-cycle example) for circular quantities like cell-cycle position. Return the joint density p_s(s,y) on a regular grid over pseudotime and marker intensity, normalize it, and save it as joint_density_pseudotime_marker.mat before passing it to preMAPiT.m. Typical inputs are flow-cytometric measurements of HCT116 spheroid cells stained for DNA, RNA, Ki-67 and p27, or NCI-H460/geminin cells measured for DNA and mAG-hGeminin (1-110).

## Related tools

- **MATLAB** (Host environment in which jointDensityPseudotimeY.m performs Gaussian kernel density estimation of the joint pseudotime-marker density) — https://github.com/karstenkuritz/MAPiT
- **Kernel Density Estimation with Linked BCs** (Provides KDE with linked boundary conditions used to obtain the distribution in pseudotime for the cell-cycle example) — https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git
- **Wanderlust** (Upstream algorithm (via the Cyt3 toolbox) that produces the pseudotime values fed into the density estimation) — https://github.com/dpeerlab/cyt3
- **Diffusion Maps** (Alternative upstream algorithm producing pseudotime values from single-cell experimental data for density estimation) — https://www.helmholtz-muenchen.de/icb/research/groups/machine-learning/projects/dpt/index.html

## Examples

```
pseudotime = csvread('pseudotime.csv'); marker = csvread('marker_signal.csv'); p_s_y = jointDensityPseudotimeY(pseudotime, marker);
```

## Evaluation signals

- The estimated joint density integrates to approximately one over its support (normalization check)
- The marginal density over pseudotime reproduces the input cell distribution before the density is passed to preMAPiT.m
- All density values are non-negative on the regular grid over pseudotime and marker intensity
- Output is saved as joint_density_pseudotime_marker.mat and jointDensityPseudotimeY.m executes without error in MATLAB
- Boundary handling matches the application: reflecting BCs at domain edges for the spheroid case, linked BCs for the cell-cycle case

## Limitations

- MAPiT (and hence this density step) is reliant on the correctness of the cell order provided by the trajectory-inference algorithm
- Constructing the real-scale target distribution for processes like stem-cell differentiation is challenging; the paper only derives it theoretically for cell cycle and spheroid cases, and cell death/division rates along the process may need to be measured
- In the imaging experiment, geminin intensity spanned several orders of magnitude and exceeded the detection range, so G1-phase cells fell below the detection limit - detector range constrains the marker dimension of the density

## Evidence

- [readme] readme: "Entry point of pseudotime values and single cell data is step 3."
- [methods] methods: "The function [`jointDensityPseudotimeY.m`](jointDensityPseudotimeY.m) takes a `nx1` vector of pseudotime values from the pseudotime algorithm"
- [results] results: "cells were in addition stained for DNA as measure for cell cycle stage, RNA as indicator for transcriptional activity, Ki-67 as marker for proliferation and p27 as marker for quiescence"
