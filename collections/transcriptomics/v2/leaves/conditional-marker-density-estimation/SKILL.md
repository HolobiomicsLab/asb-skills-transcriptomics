---
name: conditional-marker-density-estimation
description: Use when you have an nx1 pseudotime vector and matched nx1 marker signals
  from single-cell snapshot data and need marker intensity distributions as a function
  of pseudotime or a transformed real time/space scale.
when_to_use_negative:
- The pseudotime ordering from the trajectory inference algorithm is untrustworthy,
  since MAPiT relies on the correctness of the provided cell order
- No knowledge of the real-scale cell density distribution exists for the process
  of interest (e.g. stem cell differentiation, where target distributions are hard
  to construct and cell death/division rates are unknown)
- The analysis goal is trajectory inference itself rather than rescaling an existing
  pseudotemporal ordering to a meaningful scale
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: MAPiT.m
  role: Applies the measure-preserving transformation tau to the joint pseudotime-marker
    density to obtain the joint density on the real scale, from which p(y|x) is derived
  repo: https://github.com/karstenkuritz/MAPiT
- name: jointDensityPseudotimeY.m
  role: Takes the nx1 pseudotime vector and nx1 marker signal vector as input and
    computes the joint density of pseudotime and marker signal
  repo: https://github.com/karstenkuritz/MAPiT
- name: preMAPiT.m
  role: Provides the measure-preserving transformation tau from the true-scale distribution
    prior to transforming the joint density
  repo: https://github.com/karstenkuritz/MAPiT
- name: MATLAB
  role: Runtime environment in which the MAPiT scripts are adapted and called for
    each application case
  repo: https://github.com/karstenkuritz/MAPiT
- name: Kernel Density Estimation with Linked BCs
  role: Estimates the cell distribution in pseudotime for the cell cycle example prior
    to MAPiT transformation
  repo: https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git
- name: Wanderlust
  role: Generates the pseudotemporal ordering of cells that serves as the pseudotime
    input to the joint density estimation
  repo: https://github.com/dpeerlab/cyt3
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-020-60400-z
    title: Reconstructing temporal and spatial dynamics from single-cell pseudotime
      using prior knowledge of real scale cell densities
  source_content_hash: sha256:28d9cc672dde78f4eccce37c009854fdcf3c240c0effca174bcf40d57012c086
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/karstenkuritz/MAPiT
  tools:
  - Kernel Density Estimation with Linked BCs
  - MAPiT.m
  - MATLAB
  - Wanderlust
  - jointDensityPseudotimeY.m
  - preMAPiT.m
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# conditional-marker-density-estimation

## Summary

Derives conditional marker profiles p(y|x) from the joint density of pseudotime (or true-scale) and marker signal, so that marker trajectories (e.g. geminin, Ki-67, DNA content) can be read as functions of temporal or spatial position from snapshot single-cell data.

## When to use

Use when you have an nx1 pseudotime vector and matched nx1 marker signals from single-cell snapshot data and need marker intensity distributions as a function of pseudotime or a transformed real time/space scale.

## When NOT to use

- The pseudotime ordering from the trajectory inference algorithm is untrustworthy, since MAPiT relies on the correctness of the provided cell order
- No knowledge of the real-scale cell density distribution exists for the process of interest (e.g. stem cell differentiation, where target distributions are hard to construct and cell death/division rates are unknown)
- The analysis goal is trajectory inference itself rather than rescaling an existing pseudotemporal ordering to a meaningful scale

## Inputs

- nx1 vector of pseudotime values from a trajectory inference algorithm (e.g. Wanderlust from the Cyt3 toolbox, Diffusion Maps/DPT), importable as .csv
- nx1 vector of marker signals from n single cells (e.g. DNA, RNA, Ki-67, p27, mAG-hGeminin(1-110) intensities from flow cytometry)
- True-scale cell density distribution (e.g. radial cell-density distribution of spheroids) for MAPiT transformation
- Matlab session with MAPiT scripts

## Outputs

- Joint density of pseudotime and marker p_s(s,y)
- MAPiT-transformed joint density p_x(x,y) on the real temporal/spatial scale
- Conditional marker profile p_x(y|x) (e.g. geminin intensity vs. real time; Ki-67 intensity vs. distance from spheroid surface)

## How to apply

Import the nx1 pseudotime vector and nx1 marker signal vector into MATLAB (e.g. from a .csv file) and compute the joint density p_s(s,y) with jointDensityPseudotimeY.m. For the cell cycle example, estimate the pseudotime distribution with kernel density estimation with linked boundary conditions. If a real temporal or spatial scale is required, obtain the measure-preserving transformation tau with preMAPiT.m from a known true-scale cell density, then apply MAPiT.m to yield the joint density p_x(x,y) on the real scale. Normalize the joint density so it integrates to 1, then derive the conditional marker profile p_x(y|x) = p_x(x,y)/p(x). Compare the reconstructed marker profile (e.g. geminin kinetics, Ki-67 spatial distribution) against independent measurements such as time-lapse microscopy or spheroid cross-section intensities.

## Related tools

- **MAPiT.m** (Applies the measure-preserving transformation tau to the joint pseudotime-marker density to obtain the joint density on the real scale, from which p(y|x) is derived) — https://github.com/karstenkuritz/MAPiT
- **jointDensityPseudotimeY.m** (Takes the nx1 pseudotime vector and nx1 marker signal vector as input and computes the joint density of pseudotime and marker signal) — https://github.com/karstenkuritz/MAPiT
- **preMAPiT.m** (Provides the measure-preserving transformation tau from the true-scale distribution prior to transforming the joint density) — https://github.com/karstenkuritz/MAPiT
- **MATLAB** (Runtime environment in which the MAPiT scripts are adapted and called for each application case) — https://github.com/karstenkuritz/MAPiT
- **Kernel Density Estimation with Linked BCs** (Estimates the cell distribution in pseudotime for the cell cycle example prior to MAPiT transformation) — https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git
- **Wanderlust** (Generates the pseudotemporal ordering of cells that serves as the pseudotime input to the joint density estimation) — https://github.com/dpeerlab/cyt3

## Examples

```
run('jointDensityPseudotimeY.m'); tau = preMAPiT(...); p_xy = MAPiT(...);
```

## Evaluation signals

- The transformed joint density p_x(x,y) is non-negative and integrates to 1 on the real scale (a valid probability density)
- MAPiT-reconstructed geminin temporal trajectories match geminin kinetics from single-cell time-lapse microscopy, with a geminin intensity boost at approximately 7 h indicating S-phase onset
- MAPiT-derived Ki-67 distribution in an 11-day-old HCT116 spheroid closely matches Ki-67 intensity profiles from n=9 transversal quantifications of spheroid cross-sections
- Leave-one-out cross validation with marker subsets as pseudotime algorithm input yields consistent reconstructed profiles
- MAPiT.m and jointDensityPseudotimeY.m execute without error on the imported nx1 pseudotime and marker vectors

## Limitations

- MAPiT is reliant on the correctness of the cell order provided by the trajectory inference algorithm
- Constructing a target real-scale distribution for common applications like stem cell differentiation is challenging; the paper only applied MAPiT to two examples where the target distribution could be derived theoretically
- Knowing sources and sinks along the process (cell death or cell division) is critical for obtaining the right real-time distribution and subsequent correct transformation, but was not known in the presented examples
- No labelled/sampling-timepoint data were used to validate MAPiT results; the authors only envision such validation

## Evidence

- [readme] README workflow step 3: "The function [`jointDensityPseudotimeY.m`](jointDensityPseudotimeY.m) takes a `nx1` vector of pseudotime values from the pseudotime algorithm and a `nx1` vector of a marker signals from `n` single"
- [intro] MAPiT as measure-preserving transformation: "Here, we developed a measure-preserving transformation of pseudotime into real-time, a MAP of pseudotime into Time, in short MAPiT"
- [results] Geminin validation vs microscopy: "Temporal trajectories of geminin obtained with MAPiT corresponded excellently to geminin kinetics obtained by single cell time-lapse microscopy"
- [intro] Cell order dependency: "The method makes use of pseudotemporal ordering obtained from trajectory inference algorithms and hence is reliant on the correctness of the provided cell order."
- [discussion] Target distribution difficulty: "Constructing a target distribution for other processes i.e. the most common application of pseudotime single-cell analysis on stem cell differentiation, might be more challenging."
