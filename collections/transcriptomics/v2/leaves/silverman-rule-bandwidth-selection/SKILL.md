---
name: silverman-rule-bandwidth-selection
description: Use when smoothing nx1 pseudotime and nx1 marker-signal vectors from
  n single cells into a joint density (e.g., via jointDensityPseudotimeY.m for MAPiT)
  and Gaussian kernel bandwidths must be set per dimension.
when_to_use_negative:
- The input is already a histogram or estimated density rather than raw nx1 pseudotime
  and marker-sample vectors — bandwidth selection applies to sample-based Gaussian
  KDE.
- The cell order from the trajectory-inference algorithm cannot be trusted, since
  MAPiT (which consumes the smoothed density) is reliant on the correctness of the
  provided cell order.
- The application domain edges matter but neither reflecting nor linked boundary conditions
  fit the process (naive Gaussian KDE distorts density near boundaries).
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: jointDensityPseudotimeY.m (MAPiT)
  role: Performs the Gaussian kernel density estimation of pseudotime and marker signal
    with Silverman's-rule bandwidths, producing the joint density passed to preMAPiT.m
  repo: https://github.com/karstenkuritz/MAPiT
- name: Kernel Density Estimation with Linked BCs
  role: Supplies linked-boundary-condition KDE (e.g., Cts_samples) used to obtain
    the distribution in pseudotime for the cell-cycle example
  repo: https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git
- name: Wanderlust (Cyt3 toolbox)
  role: Generates the nx1 pseudotime values that are the input samples to the bandwidth-smoothed
    joint density estimation
  repo: https://github.com/dpeerlab/cyt3
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-60400-z
    title: Reconstructing temporal and spatial dynamics from single-cell pseudotime
      using prior knowledge of real scale cell densities
  source_content_hash: sha256:9fe64d37d979b97a2f1aae5c51ee52706423b9b56c8d2000cd97d50d9bcc5c12
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/karstenkuritz/MAPiT
  tools:
  - Kernel Density Estimation with Linked BCs
  - Wanderlust (Cyt3 toolbox)
  - jointDensityPseudotimeY.m (MAPiT)
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Silverman's-rule bandwidth selection for Gaussian kernel density estimation

## Summary

Chooses Gaussian-kernel smoothing bandwidths by Silverman's rule of thumb, applied independently in the pseudotime and marker dimensions, when estimating the joint density p_s(s,y) from single-cell pseudotime and marker intensities. This joint density is the input required by the MAPiT measure-preserving transformation into the real scale.

## When to use

Use when smoothing nx1 pseudotime and nx1 marker-signal vectors from n single cells into a joint density (e.g., via jointDensityPseudotimeY.m for MAPiT) and Gaussian kernel bandwidths must be set per dimension.

## When NOT to use

- The input is already a histogram or estimated density rather than raw nx1 pseudotime and marker-sample vectors — bandwidth selection applies to sample-based Gaussian KDE.
- The cell order from the trajectory-inference algorithm cannot be trusted, since MAPiT (which consumes the smoothed density) is reliant on the correctness of the provided cell order.
- The application domain edges matter but neither reflecting nor linked boundary conditions fit the process (naive Gaussian KDE distorts density near boundaries).

## Inputs

- nx1 vector of pseudotime values from a trajectory-inference algorithm (e.g., Wanderlust via Cyt3 toolbox or Diffusion Maps)
- nx1 vector of a marker signal from n single cells, imported into MATLAB e.g. from a .csv file

## Outputs

- Joint density p_s(s,y) of pseudotime and marker intensity on a regular grid over both dimensions
- Saved MATLAB file joint_density_pseudotime_marker.mat
- Marginal density over pseudotime (input cell distribution on the pseudotime scale) for validation

## How to apply

Import the nx1 pseudotime vector (from Wanderlust via Cyt3 or Diffusion Maps) and the nx1 marker-signal vector from n single cells into MATLAB, e.g. from a .csv file, and call jointDensityPseudotimeY.m. Estimate the joint density with Gaussian kernel density estimation, applying Silverman's-rule bandwidths separately in both the pseudotime and the marker dimension (for Gaussian kernels, h = 0.9·min(σ, IQR/1.34)·n^(−1/5) per variable). Choose boundary handling by application: reflecting boundary conditions at domain edges for the spheroid (spatial) case, and kernel density estimation with linked boundary conditions (MColbrook/Kernel-Density-Estimation-with-Linked-BCs, e.g. Cts_samples) for the cell-cycle case. Return the joint density p_s(s,y) on a regular grid over pseudotime and marker intensity and save it as joint_density_pseudotime_marker.mat. Verify normalization (integral ≈ 1 over the support) and that the pseudotime marginal reproduces the input cell distribution before passing the density to preMAPiT.m.

## Related tools

- **jointDensityPseudotimeY.m (MAPiT)** (Performs the Gaussian kernel density estimation of pseudotime and marker signal with Silverman's-rule bandwidths, producing the joint density passed to preMAPiT.m) — https://github.com/karstenkuritz/MAPiT
- **Kernel Density Estimation with Linked BCs** (Supplies linked-boundary-condition KDE (e.g., Cts_samples) used to obtain the distribution in pseudotime for the cell-cycle example) — https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git
- **Wanderlust (Cyt3 toolbox)** (Generates the nx1 pseudotime values that are the input samples to the bandwidth-smoothed joint density estimation) — https://github.com/dpeerlab/cyt3

## Examples

```
load pseudotime_marker.csv; joint_density = jointDensityPseudotimeY(pseudotime, y);
```

## Evaluation signals

- The estimated joint density integrates to approximately one over its support (normalization check).
- The marginal density over pseudotime reproduces the input cell distribution.
- Bandwidths are of order n^(−1/5) per dimension and are set independently for pseudotime and marker axes.
- Output is a regular grid over both pseudotime and marker intensity, saved as joint_density_pseudotime_marker.mat.
- The smoothed density is accepted by preMAPiT.m downstream without boundary artifacts at the domain edges.

## Limitations

- Silverman's rule assumes roughly Gaussian-shaped data; it can oversmooth sharp or multimodal marker structure in the joint density.
- Boundary handling is application-specific: reflecting BCs for the spheroid (spatial) case versus linked BCs for the cell-cycle case; using the wrong one distorts the density near edges.
- The resulting density is only as good as the pseudotemporal ordering — the method is reliant on the correctness of the provided cell order.
- For processes like stem cell differentiation, constructing the target real-scale distribution the density feeds into might be more challenging.

## Evidence

- [methods] KDE function signature: "The function [`jointDensityPseudotimeY.m`](jointDensityPseudotimeY.m) takes a `nx1` vector of pseudotime values from the pseudotime algorithm"
- [readme] Linked-BC KDE for cell cycle: "kernel density estmation with linked boundary conditions (https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git) is used to obtain distribution in pseudotime"
- [readme] Workflow step for joint density: "Get joint distribution of pseudotime and markers with"
- [intro] Dependence on cell ordering: "The method makes use of pseudotemporal ordering obtained from trajectory inference algorithms and hence is reliant on the correctness of the provided cell order."
- [discussion] Challenge of target distributions: "Constructing a target distribution for other processes i.e. the most common application of pseudotime single-cell analysis on stem cell differentiation, might be more challenging."
