---
name: cell-cycle-age-distribution-modeling
description: Use when flow-cytometric single-cell data from an unsynchronized, exponentially
  growing culture must be placed on real cell-cycle time, and no time-resolved sampling
  exists to calibrate pseudotime directly.
when_to_use_negative:
- When the population is synchronized or otherwise not in exponential growth — the
  exponential age distribution 2*gamma*exp(-gamma*a) is only valid for unsynchronized
  populations
- When the trajectory-inference algorithm's cell order is unreliable — MAPiT is reliant
  on the correctness of the provided cell order
- When the target real-scale distribution cannot be derived (e.g., stem cell differentiation,
  where cell death and division rates at every stage would need to be measured)
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: MATLAB
  role: Implementation environment; MAPiT is a set of Matlab scripts (jointDensityPseudotimeY.m,
    preMAPiT.m, MAPiT.m) adapted per application case
  repo: https://github.com/karstenkuritz/MAPiT
- name: Kernel Density Estimation with Linked BCs
  role: Obtains the smoothed pseudotime cell-density distribution needed for the transformation
    in the cell cycle example
  repo: https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git
- name: Wanderlust
  role: Generates the pseudotemporal ordering of cells that serves as pseudotime input
  repo: https://github.com/dpeerlab/cyt3
- name: Diffusion Maps
  role: Alternative pseudotime algorithm used to derive pseudotime values from single-cell
    experimental data
  repo: https://www.helmholtz-muenchen.de/icb/research/groups/machine-learning/projects/dpt/index.html
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-020-60400-z
    title: Reconstructing temporal and spatial dynamics from single-cell pseudotime
      using prior knowledge of real scale cell densities
  source_content_hash: sha256:2638f65134665108a89912fa70f2457e9c11022d2ceb453fed896ef29a1610b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/karstenkuritz/MAPiT
  tools:
  - Diffusion Maps
  - Kernel Density Estimation with Linked BCs
  - MATLAB
  - Wanderlust
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cell-cycle-age-distribution-modeling

## Summary

Models the age distribution of cells in an unsynchronized cell-cycle population as an exponential density (2*gamma*exp(-gamma*a)) and uses it as the true-scale target for MAPiT's measure-preserving transformation, mapping arbitrary pseudotime values onto real cell-cycle time.

## When to use

Use when flow-cytometric single-cell data from an unsynchronized, exponentially growing culture must be placed on real cell-cycle time, and no time-resolved sampling exists to calibrate pseudotime directly.

## When NOT to use

- When the population is synchronized or otherwise not in exponential growth — the exponential age distribution 2*gamma*exp(-gamma*a) is only valid for unsynchronized populations
- When the trajectory-inference algorithm's cell order is unreliable — MAPiT is reliant on the correctness of the provided cell order
- When the target real-scale distribution cannot be derived (e.g., stem cell differentiation, where cell death and division rates at every stage would need to be measured)

## Inputs

- nx1 vector of pseudotime values from a trajectory-inference algorithm (e.g., Wanderlust or Diffusion Maps), imported as a .csv file
- nx1 vector of marker signals from n single cells (e.g., DNA content and mAG-hGeminin (1-110) from static flow cytometry)
- Growth-rate parameter gamma for the exponential age distribution 2*gamma*exp(-gamma*a)

## Outputs

- Joint density of pseudotime and markers from jointDensityPseudotimeY.m
- Strictly monotonically increasing transformation map (pseudotime grid and corresponding real-scale age values), saved as transformation_map.mat
- Geminin (and other marker) trajectories on the real cell-cycle time scale

## How to apply

Import the nx1 pseudotime vector and nx1 marker-signal vector (e.g., DNA content and mAG-hGeminin (1-110) from a static flow cytometry measurement of NCI-H460/geminin cells) into MATLAB from a .csv file. Compute the joint density of pseudotime and markers with jointDensityPseudotimeY.m, using kernel density estimation with linked boundary conditions (MColbrook KDE-with-Linked-BCs) for the smoothed pseudotime cell-density distribution. Define the target real-scale age distribution from prior knowledge of unsynchronized exponential growth: 2*gamma*exp(-gamma*a), where a is cell age and gamma the growth rate. Compute the CDFs P_s(s) and P_x(x) and solve the strictly monotone map tau(s) = P_x^{-1}(P_s(s)) numerically with preMAPiT.m, then apply it with MAPiT.m. Validate by comparing MAPiT-reconstructed geminin temporal trajectories against geminin kinetics from single-cell time-lapse microscopy, where a boost in geminin intensity at approximately 7 h indicates onset of S-phase.

## Related tools

- **MATLAB** (Implementation environment; MAPiT is a set of Matlab scripts (jointDensityPseudotimeY.m, preMAPiT.m, MAPiT.m) adapted per application case) — https://github.com/karstenkuritz/MAPiT
- **Kernel Density Estimation with Linked BCs** (Obtains the smoothed pseudotime cell-density distribution needed for the transformation in the cell cycle example) — https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git
- **Wanderlust** (Generates the pseudotemporal ordering of cells that serves as pseudotime input) — https://github.com/dpeerlab/cyt3
- **Diffusion Maps** (Alternative pseudotime algorithm used to derive pseudotime values from single-cell experimental data) — https://www.helmholtz-muenchen.de/icb/research/groups/machine-learning/projects/dpt/index.html

## Examples

```
T = load('pseudotime.csv'); Y = load('markers.csv'); rho = jointDensityPseudotimeY(T, Y); tau = preMAPiT(rho, @(a) 2*gamma*exp(-gamma*a)); save('transformation_map.mat','tau');
```

## Evaluation signals

- The transformation tau(s) = P_x^{-1}(P_s(s)) is strictly monotonically increasing over the pseudotime grid
- Applying the map carries the pseudotime CDF onto the target exponential age CDF (qualitative CDF-match check)
- MAPiT-reconstructed geminin temporal trajectory corresponds to geminin kinetics from single-cell time-lapse microscopy, with the S-phase onset geminin boost appearing at approximately 7 h
- Cross-validation with leave-one-out marker subsets as pseudotime algorithm input yields consistent transformed trajectories

## Limitations

- MAPiT is reliant on the correctness of the cell order provided by the trajectory inference algorithm
- Knowing sources and sinks along the process like cell death or cell division is critical for obtaining the right real-time distribution; in the presented examples these were not known
- The geminin intensity spans several orders of magnitude and exceeds the detection range in imaging experiments such that G1 phase cells had signals below the detection limit

## Evidence

- [readme] Define the true-scale target distribution from prior knowledge: "Workflow for analysing single-cell data with MAPiT 1. Generate pseudotemporal ordering of cells with your favorite algorithm 2. Define true-scale distribution"
- [methods] Cell-cycle age distribution as MAPiT target: "kernel density estmation with linked boundary conditions (https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git) is used to obtain distribution in pseudotime for the"
- [results] Validation against microscopy: "Temporal trajectories of geminin obtained with MAPiT corresponded excellently to geminin kinetics obtained by single cell time-lapse microscopy"
- [intro] MAPiT scope and reliance on ordering: "The method makes use of pseudotemporal ordering obtained from trajectory inference algorithms and hence is reliant on the correctness of the provided cell order."
