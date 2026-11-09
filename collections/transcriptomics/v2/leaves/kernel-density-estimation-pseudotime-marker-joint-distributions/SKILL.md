---
name: kernel-density-estimation-pseudotime-marker-joint-distributions
description: Use when a MAPiT workflow needs the joint distribution of pseudotime
  (from a Wanderlust/DPT/Diffusion Maps ordering) and a marker signal from n single
  cells, to be estimated by Gaussian KDE on a regular grid before preMAPiT.m.
when_to_use_negative:
- The pseudotemporal ordering from the trajectory-inference algorithm is not trusted
  — MAPiT is reliant on the correctness of the provided cell order, and this KDE step
  assumes that ordering is valid
- You do not have (and cannot construct) a true-scale distribution to pair with the
  pseudotime scale — e.g., for stem-cell differentiation, where constructing a target
  distribution 'might be more challenging' per the authors
- You need kinetics validated against labelled sampling-timepoint data — MAPiT/KDE
  reconstruction does not use label data; the authors only envision such validation
tools:
- name: MAPiT (jointDensityPseudotimeY.m)
  role: MATLAB function that takes the nx1 pseudotime vector and nx1 marker vector
    and returns the joint density of pseudotime and marker signal; entry point for
    pseudotime values and single-cell data in the MAPiT workflow
  repo: https://github.com/karstenkuritz/MAPiT
- name: Kernel Density Estimation with Linked BCs
  role: Supplies kernel density estimation with linked boundary conditions used to
    obtain the distribution in pseudotime for the cell-cycle example
  repo: https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git
- name: Wanderlust (Cyt3 toolbox)
  role: Produces the pseudotemporal ordering / nx1 pseudotime vector that is one of
    the two KDE inputs in the examples
  repo: https://github.com/dpeerlab/cyt3
- name: MATLAB
  role: Runtime environment in which the KDE, jointDensityPseudotimeY.m, and the surrounding
    MAPiT scripts are executed
  repo: https://github.com/karstenkuritz/MAPiT
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-60400-z
    title: Reconstructing temporal and spatial dynamics from single-cell pseudotime
      using prior knowledge of real scale cell densities
  source_content_hash: sha256:d17c00f2d82e8014508432f48848112b5aeeaedc7b32c404fe4f90347ff7c914
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/karstenkuritz/MAPiT
  tools:
  - Kernel Density Estimation with Linked BCs
  - MAPiT (jointDensityPseudotimeY.m)
  - MATLAB
  - Wanderlust (Cyt3 toolbox)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# kernel-density-estimation-pseudotime-marker-joint-distributions

## Summary

Estimates the joint density p_s(s,y) of pseudotime values and a marker signal from n single cells by Gaussian kernel density estimation (KDE) on a regular grid. This density is the input required by MAPiT's measure-preserving transformation (preMAPiT.m) of pseudotime into real-time or spatial scales.

## When to use

Use when a MAPiT workflow needs the joint distribution of pseudotime (from a Wanderlust/DPT/Diffusion Maps ordering) and a marker signal from n single cells, to be estimated by Gaussian KDE on a regular grid before preMAPiT.m.

## When NOT to use

- The pseudotemporal ordering from the trajectory-inference algorithm is not trusted — MAPiT is reliant on the correctness of the provided cell order, and this KDE step assumes that ordering is valid
- You do not have (and cannot construct) a true-scale distribution to pair with the pseudotime scale — e.g., for stem-cell differentiation, where constructing a target distribution 'might be more challenging' per the authors
- You need kinetics validated against labelled sampling-timepoint data — MAPiT/KDE reconstruction does not use label data; the authors only envision such validation

## Inputs

- nx1 vector of pseudotime values from a trajectory-inference algorithm (e.g., Wanderlust via Cyt3, Diffusion Maps/DPT), importable to MATLAB as a .csv file
- nx1 vector of marker signals measured in the same n single cells (e.g., flow-cytometric DNA, RNA, Ki-67, p27, or geminin intensities)

## Outputs

- Joint density p_s(s,y) of pseudotime and marker signal on a regular grid over pseudotime and marker intensity
- MAT-file joint_density_pseudotime_marker.mat for downstream consumption by preMAPiT.m

## How to apply

Import the nx1 vector of pseudotime values (e.g., from Wanderlust via the Cyt3 toolbox, or Diffusion Maps/DPT) and the nx1 vector of marker intensities from the n single cells into MATLAB, e.g., from a .csv file. Call jointDensityPseudotimeY.m with these two vectors as input to estimate the joint density of pseudotime and the marker signal via Gaussian kernel density estimation. Use Silverman's-rule bandwidths for smoothing in both the pseudotime and the marker dimension; choose boundary conditions per application — reflecting boundary conditions at the domain edges for the spheroid (spatial) case, and kernel density estimation with linked boundary conditions (MColbrook/Kernel-Density-Estimation-with-Linked-BCs) for the cell-cycle case. Return the joint density p_s(s,y) on a regular grid over pseudotime and marker intensity (saved as joint_density_pseudotime_marker.mat). Rationale: MAPiT is a measure-preserving transformation, so its transformation function (preMAPiT.m) needs the correctly normalized distribution on the pseudotime scale; entry point of pseudotime values and single-cell data is step 3 of the MAPiT workflow.

## Related tools

- **MAPiT (jointDensityPseudotimeY.m)** (MATLAB function that takes the nx1 pseudotime vector and nx1 marker vector and returns the joint density of pseudotime and marker signal; entry point for pseudotime values and single-cell data in the MAPiT workflow) — https://github.com/karstenkuritz/MAPiT
- **Kernel Density Estimation with Linked BCs** (Supplies kernel density estimation with linked boundary conditions used to obtain the distribution in pseudotime for the cell-cycle example) — https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git
- **Wanderlust (Cyt3 toolbox)** (Produces the pseudotemporal ordering / nx1 pseudotime vector that is one of the two KDE inputs in the examples) — https://github.com/dpeerlab/cyt3
- **MATLAB** (Runtime environment in which the KDE, jointDensityPseudotimeY.m, and the surrounding MAPiT scripts are executed) — https://github.com/karstenkuritz/MAPiT

## Examples

```
data = csvread('pseudotime_marker.csv'); p = jointDensityPseudotimeY(data(:,1), data(:,2));
```

## Evaluation signals

- Normalization check: the estimated joint density integrates to approximately one over its support
- Marginal check: the marginal density of the joint density over pseudotime reproduces the input cell distribution before it is passed to preMAPiT.m
- Output is returned on a regular grid over both pseudotime and marker intensity dimensions
- Boundary-condition choice matches the application: reflecting BCs for the spheroid spatial domain, linked BCs for the cell-cycle domain
- Downstream compatibility: the saved joint_density_pseudotime_marker.mat is accepted by preMAPiT.m without schema errors

## Limitations

- MAPiT/KDE is reliant on the correctness of the cell order provided by the trajectory-inference algorithm
- Sources and sinks along the process (cell death or cell division) are unknown in the presented examples but are critical for obtaining the right real-time distribution and a correct transformation
- Constructing a target distribution for other processes, i.e., the most common application of pseudotime single-cell analysis on stem-cell differentiation, might be more challenging than the two applications shown

## Evidence

- [readme] The function jointDensityPseudotimeY.m takes a nx1 vector of pseudotime values from the pseudotime algorithm: "The function [`jointDensityPseudotimeY.m`](jointDensityPseudotimeY.m) takes a `nx1` vector of
pseudotime values from the pseudotime algorithm"
- [methods] Kernel density estimation with linked boundary conditions used for the cell-cycle distribution in pseudotime: "kernel density estmation with linked boundary conditions (https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git) is used to obtain distribution in pseudotime for the"
- [readme] Entry point of pseudotime values and single cell data is the joint-density step of the MAPiT workflow: "Usage of MAPiT with R or Python based pseudotime analysis methods is straight
 forward. Entry point of pseudotime values and single cell data is step 3."
- [readme] MAPiT workflow step: get joint distribution of pseudotime and markers: "3. Get joint distribution of pseudotime and markers with
[`jointDensityPseudotimeY.m`](jointDensityPseudotimeY.m)"
- [intro] MAPiT relies on the correctness of the pseudotemporal ordering: "The method makes use of pseudotemporal ordering obtained from trajectory inference algorithms and hence is reliant on the correctness of the provided cell order."
