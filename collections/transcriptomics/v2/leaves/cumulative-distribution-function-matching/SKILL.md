---
name: cumulative-distribution-function-matching
description: Use when you have an nx1 vector of pseudotime values from a trajectory-inference
  algorithm (Wanderlust, Diffusion Maps/DPT, Monocle) and independent prior knowledge
  of the cell density distribution on the desired real scale, and you need to convert
  pseudotime to true time or distance-from-surface.
when_to_use_negative:
- No reliable prior knowledge of the target real-scale distribution exists (e.g. stem-cell
  differentiation, where cell death and division rates at every stage would need to
  be measured) — the paper notes constructing such target distributions can be more
  challenging.
- The cell order from the trajectory-inference algorithm itself is unreliable — MAPiT
  is reliant on the correctness of the provided cell order.
- Data are already on a meaningful real-time or real-space scale and do not require
  rescaling from pseudotime.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3518
tools:
- name: MAPiT
  role: Matlab implementation providing preMAPiT.m (computes the CDF-matching transformation)
    and MAPiT.m (applies the transformation to pseudotime trajectories)
  repo: https://github.com/karstenkuritz/MAPiT
- name: Kernel Density Estimation with Linked BCs
  role: Used to obtain the smoothed pseudotime cell-density distribution feeding the
    CDF computation
  repo: https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git
- name: Wanderlust
  role: Generates the pseudotemporal ordering of cells that is the input to the transformation
    (part of the Cyt3 toolbox)
  repo: https://github.com/dpeerlab/cyt3
- name: MATLAB
  role: Execution environment for the MAPiT scripts and KDE-with-Linked-BCs routines
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-020-60400-z
    title: Reconstructing temporal and spatial dynamics from single-cell pseudotime
      using prior knowledge of real scale cell densities
  source_content_hash: sha256:be18cb7c29df20cc67e44b61520b5e3464311cf8be0a64e5d9e7fa3b54b6717b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/karstenkuritz/MAPiT
  tools:
  - Kernel Density Estimation with Linked BCs
  - MAPiT
  - MATLAB
  - Wanderlust
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# cumulative-distribution-function-matching

## Summary

A measure-preserving transformation (MAPiT) that maps arbitrary pseudotime values onto the true temporal or spatial scale by enforcing equality of the area under the density (cumulative distribution) curves at corresponding points, i.e. tau(s) = P_x^{-1}(P_s(s)). It converts abstract pseudotime orderings from trajectory-inference algorithms into interpretable real-scale values.

## When to use

Use when you have an nx1 vector of pseudotime values from a trajectory-inference algorithm (Wanderlust, Diffusion Maps/DPT, Monocle) and independent prior knowledge of the cell density distribution on the desired real scale, and you need to convert pseudotime to true time or distance-from-surface coordinates.

## When NOT to use

- No reliable prior knowledge of the target real-scale distribution exists (e.g. stem-cell differentiation, where cell death and division rates at every stage would need to be measured) — the paper notes constructing such target distributions can be more challenging.
- The cell order from the trajectory-inference algorithm itself is unreliable — MAPiT is reliant on the correctness of the provided cell order.
- Data are already on a meaningful real-time or real-space scale and do not require rescaling from pseudotime.

## Inputs

- nx1 vector of pseudotime values from a pseudotime algorithm (e.g. Wanderlust or Diffusion Maps), importable from a .csv file
- nx1 vector of marker signals from n single cells (e.g. DNA, geminin, Ki-67 from flow cytometry)
- Prior-knowledge target real-scale cell-density or cumulative distribution (e.g. exponential age distribution 2*gamma*exp(-gamma*a), or radial spheroid cell density vs distance from surface)

## Outputs

- Strictly monotonically increasing transformation map (pseudotime grid and corresponding real-scale values), saved as transformation_map.mat
- Pseudotime trajectories transformed to the real temporal or spatial scale via MAPiT.m

## How to apply

Load the nx1 pseudotime vector and nx1 marker-signal vector from n single cells into MATLAB (e.g. from a .csv file). Compute the joint density of pseudotime and markers with jointDensityPseudotimeY.m, using kernel density estimation with linked boundary conditions (MColbrook KDE-with-Linked-BCs) to obtain a smoothed pseudotime cell-density distribution. Define the target real-scale distribution from prior knowledge — e.g. the exponential age distribution 2*gamma*exp(-gamma*a) for the unsynchronized cell cycle example, or the radial spherical-shell cell-density distribution for the spheroid example. Compute the CDFs P_s(s) and P_x(x) of both distributions and solve tau(s) = P_x^{-1}(P_s(s)) numerically with preMAPiT.m, enforcing strict monotonicity of the map; apply the saved map with MAPiT.m. The rationale is that matching cumulative distributions guarantees the transformed pseudotime reproduces the known cell density on the real scale (measure preservation).

## Related tools

- **MAPiT** (Matlab implementation providing preMAPiT.m (computes the CDF-matching transformation) and MAPiT.m (applies the transformation to pseudotime trajectories)) — https://github.com/karstenkuritz/MAPiT
- **Kernel Density Estimation with Linked BCs** (Used to obtain the smoothed pseudotime cell-density distribution feeding the CDF computation) — https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git
- **Wanderlust** (Generates the pseudotemporal ordering of cells that is the input to the transformation (part of the Cyt3 toolbox)) — https://github.com/dpeerlab/cyt3
- **MATLAB** (Execution environment for the MAPiT scripts and KDE-with-Linked-BCs routines)

## Examples

```
load pseudotime.csv; Y = load marker_signals.csv; jointDensity = jointDensityPseudotimeY(pseudotime, Y); transformation = preMAPiT.m; newScale = MAPiT.m(pseudotime, 'transformation_map.mat');
```

## Evaluation signals

- Verify the resulting transformation map tau(s) is strictly monotonically increasing.
- Check qualitatively that applying the map transforms the pseudotime CDF P_s(s) onto the target real-scale CDF P_x(x) (CDF-match check; the paper reports no numeric threshold).
- For the cell cycle example, compare MAPiT-reconstructed geminin kinetics against geminin kinetics from single-cell time-lapse microscopy (expected: excellent correspondence, geminin boost at ~7 h marking S-phase onset).
- For the spheroid example, compare MAPiT-derived Ki-67 distribution against Ki-67 intensity profiles from spheroid cross-sections quantified by confocal microscopy (n=9 transversal quantifications).
- Confirm the pseudotime and marker vectors are both nx1 and derived from the same n single cells.

## Limitations

- MAPiT relies on the correctness of the cell order provided by the trajectory-inference algorithm.
- Constructing a target real-scale distribution for common applications like stem-cell differentiation is challenging; cell death and cell division rates at every stage of the process may need to be measured but were not provided.
- Sources and sinks along the process (cell death or cell division) are critical for a correct real-time transformation but were unknown in the presented examples.
- No labelled/sampling-timepoint data were used to support or validate MAPiT results in the paper; such validation was only envisioned.

## Evidence

- [intro] Here, we developed a measure-preserving transformation of pseudotime into real-time, a MAP of pseudotime into Time, in short MAPiT16.: "Here, we developed a measure-preserving transformation of pseudotime into real-time, a MAP of pseudotime into Time, in short MAPiT16."
- [readme] kernel density estmation with linked boundary conditions ... is used to obtain distribution in pseudotime: "kernel density estmation with linked boundary conditions (https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git) is used to obtain distribution in pseudotime"
- [methods] The function jointDensityPseudotimeY.m takes a nx1 vector of pseudotime values from the pseudotime algorithm: "The function [`jointDensityPseudotimeY.m`](jointDensityPseudotimeY.m) takes a `nx1` vector of pseudotime values from the pseudotime algorithm"
- [methods] Get transformation with preMAPiT.m: "4. Get transformation with [`preMAPiT.m`](preMAPiT.m)"
- [intro] The method makes use of pseudotemporal ordering obtained from trajectory inference algorithms and hence is reliant on the correctness of the provided cell order.: "The method makes use of pseudotemporal ordering obtained from trajectory inference algorithms and hence is reliant on the correctness of the provided cell order."
- [results] MAPiT-reconstructed geminin temporal trajectories corresponded well to geminin kinetics from single-cell time-lapse microscopy: "Temporal trajectories of geminin obtained with MAPiT corresponded excellently to geminin kinetics obtained by single cell time-lapse microscopy"
