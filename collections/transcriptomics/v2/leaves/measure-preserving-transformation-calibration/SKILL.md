---
name: measure-preserving-transformation-calibration
description: Use when you have an nx1 vector of pseudotime values (from Wanderlust,
  Monocle or Diffusion Pseudotime) plus single-cell marker signals, and a theoretically
  or empirically known cell-density distribution on a true real-time or spatial scale
  that the pseudotime ordering must be mapped onto.
when_to_use_negative:
- No defensible target real-scale distribution exists (e.g. stem cell differentiation
  without measured cell death and division rates at every stage) — the authors note
  constructing such a target distribution 'might be more challenging'
- 'The pseudotime ordering itself is unreliable: MAPiT is reliant on the correctness
  of the provided cell order from the trajectory inference algorithm'
- Sources and sinks (cell death or cell division) along the process are unknown yet
  critical for a correct real-time distribution and transformation
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0092
tools:
- name: MAPiT (preMAPiT.m / MAPiT.m / jointDensityPseudotimeY.m)
  role: Matlab scripts that compute the joint pseudotime-marker density, solve the
    measure-preserving transformation tau(s) = P_x^{-1}(P_s(s)), and apply it to trajectories
  repo: https://github.com/karstenkuritz/MAPiT
- name: Kernel Density Estimation with Linked BCs
  role: Used to obtain the smoothed cell-density distribution in pseudotime required
    by the transformation (e.g. via Cts_samples)
  repo: https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git
- name: Wanderlust (Cyt3 toolbox)
  role: Generates the input pseudotemporal ordering of single cells that MAPiT transforms
    to the true scale
  repo: https://github.com/dpeerlab/cyt3
- name: Diffusion Maps / DPT
  role: Alternative pseudotime algorithm providing the nx1 pseudotime vector as transformation
    input
  repo: https://www.helmholtz-muenchen.de/icb/research/groups/machine-learning/projects/dpt/index.html
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1038/s41598-020-60400-z
    title: Reconstructing temporal and spatial dynamics from single-cell pseudotime
      using prior knowledge of real scale cell densities
  source_content_hash: sha256:709d2568175811b4449a1adba5ed9b723adbcb30a544deb69ad23d27627b2589
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/karstenkuritz/MAPiT
  tools:
  - Diffusion Maps / DPT
  - Kernel Density Estimation with Linked BCs
  - MAPiT (preMAPiT.m / MAPiT.m / jointDensityPseudotimeY.m)
  - Wanderlust (Cyt3 toolbox)
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# measure-preserving-transformation-calibration

## Summary

Calibrates a measure-preserving transformation (MAPiT) that maps arbitrary pseudotime values onto a biologically meaningful true scale (real time or distance-from-spheroid-surface) by requiring prior knowledge of cell distributions on both scales. Implemented as Matlab scripts (preMAPiT.m, MAPiT.m) in the karstenkuritz/MAPiT repository.

## When to use

Use when you have an nx1 vector of pseudotime values (from Wanderlust, Monocle or Diffusion Pseudotime) plus single-cell marker signals, and a theoretically or empirically known cell-density distribution on a true real-time or spatial scale that the pseudotime ordering must be mapped onto.

## When NOT to use

- No defensible target real-scale distribution exists (e.g. stem cell differentiation without measured cell death and division rates at every stage) — the authors note constructing such a target distribution 'might be more challenging'
- The pseudotime ordering itself is unreliable: MAPiT is reliant on the correctness of the provided cell order from the trajectory inference algorithm
- Sources and sinks (cell death or cell division) along the process are unknown yet critical for a correct real-time distribution and transformation

## Inputs

- nx1 vector of pseudotime values from a trajectory-inference algorithm (Wanderlust, Monocle, DPT/Diffusion Maps)
- nx1 vectors of single-cell marker signals (e.g. DNA content, RNA, Ki-67, p27, forward scatter from flow cytometry), importable as a .csv file
- Prior-knowledge target distribution on the true scale (e.g. radial cell-density distribution of a sphere, or exponential age distribution 2*gamma*exp(-gamma*a))

## Outputs

- Strictly monotonically increasing pseudotime-to-true-scale transformation map (pseudotime grid and corresponding real-scale values, saved as transformation_map.mat)
- Marker intensities plotted against the true scale (e.g. Ki-67, p27, DNA content vs. distance from spheroid surface, mapit_spheroid_profiles.png)
- Joint density of pseudotime and markers for downstream transformation

## How to apply

First compute the joint density of pseudotime and each marker from the nx1 pseudotime and marker vectors with jointDensityPseudotimeY.m, using kernel density estimation with linked boundary conditions (MColbrook KDE-with-Linked-BCs) where a smoothed pseudotime cell-density distribution is needed. Next define the true-scale target distribution from prior knowledge — e.g. the exponential age distribution 2*gamma*exp(-gamma*a) for unsynchronized cell cycle progression, or the spherical-shell radial cell-density distribution for a ~500 µm diameter spheroid with a necrotic core (r = 270 µm) under a linear growth model. Compute the cumulative distribution functions P_s(s) and P_x(x) of the pseudotime and target-scale densities, then solve tau(s) = P_x^{-1}(P_s(s)) numerically with preMAPiT.m, enforcing strict monotonicity of the resulting map; this measure-preserving construction enforces equality of the area under the density curves at corresponding points. Finally apply the saved transformation (e.g. transformation_map.mat) to pseudotime trajectories with MAPiT.m to place each cell on the true spatial or temporal scale.

## Related tools

- **MAPiT (preMAPiT.m / MAPiT.m / jointDensityPseudotimeY.m)** (Matlab scripts that compute the joint pseudotime-marker density, solve the measure-preserving transformation tau(s) = P_x^{-1}(P_s(s)), and apply it to trajectories) — https://github.com/karstenkuritz/MAPiT
- **Kernel Density Estimation with Linked BCs** (Used to obtain the smoothed cell-density distribution in pseudotime required by the transformation (e.g. via Cts_samples)) — https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git
- **Wanderlust (Cyt3 toolbox)** (Generates the input pseudotemporal ordering of single cells that MAPiT transforms to the true scale) — https://github.com/dpeerlab/cyt3
- **Diffusion Maps / DPT** (Alternative pseudotime algorithm providing the nx1 pseudotime vector as transformation input) — https://www.helmholtz-muenchen.de/icb/research/groups/machine-learning/projects/dpt/index.html

## Examples

```
run example_spheroid.m  % MATLAB: loads HCT116 spheroid flow data + pseudotime, computes jointDensityPseudotimeY, preMAPiT (radial cell-density target), MAPiT.m
```

## Evaluation signals

- The transformation map tau(s) is strictly monotonically increasing
- Applying the map transforms the pseudotime CDF P_s(s) onto the target real-scale CDF P_x(x) (qualitative CDF-match check; no numeric threshold reported in the paper)
- For the spheroid case, the MAPiT-derived Ki-67 spatial distribution closely matches Ki-67 intensity profiles quantified microscopically in spheroid cross-sections (n = 9 transversal quantifications)
- For the cell cycle case, reconstructed geminin kinetics correspond to single-cell time-lapse microscopy, with a geminin intensity boost at approximately 7 h marking S-phase onset
- Leave-one-out cross validation over marker subsets (feeding subsets of DNA, RNA, Ki-67, p27 to Wanderlust/DPT) yields consistent spatial profiles

## Limitations

- MAPiT is reliant on the correctness of the pseudotemporal cell order provided by the trajectory inference algorithm
- Constructing a target real-scale distribution for common applications like stem cell differentiation is challenging; the paper only derived target distributions theoretically for two examples
- For differentiation processes, cell death and cell division rates at every stage may need to be measured but are not provided; unknown sources and sinks are critical for obtaining the correct real-time distribution
- No labelled/sampling-timepoint data were used to support or validate MAPiT results; the authors only envision such validation
- In the imaging experiment, geminin intensity exceeded the detection range so G1 phase cells had signals below the detection limit

## Evidence

- [results] measure-preserving transformation definition: [evidence span withheld — anchor status: not_found]
- [methods] workflow step: get transformation: "4. Get transformation with [`preMAPiT.m`](preMAPiT.m)"
- [readme] joint density input spec: "The function [`jointDensityPseudotimeY.m`](jointDensityPseudotimeY.m) takes a `nx1` vector of pseudotime values from the pseudotime algorithm"
- [readme] KDE dependency: "kernel density estmation with linked boundary conditions (https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git) is used to obtain distribution in pseudotime"
- [intro] order dependency limitation: "The method makes use of pseudotemporal ordering obtained from trajectory inference algorithms and hence is reliant on the correctness of the provided cell order."
- [results] spheroid validation: [evidence span withheld — anchor status: not_found]
