---
name: measure-preserving-pseudotime-to-real-scale-transformation
description: Use when snapshot single-cell data have been ordered in pseudotime (e.g.
  by Wanderlust or DPT) and marker trajectories must be interpreted on a real time
  or spatial scale for which a true-scale cell density distribution is available.
when_to_use_negative:
- When no prior knowledge of the true-scale cell density distribution exists — constructing
  a target distribution for processes like stem cell differentiation can be challenging
  and may require measured cell death and division rates at every stage.
- When the cell ordering from the trajectory inference algorithm itself is unreliable
  — MAPiT is reliant on the correctness of the provided cell order and does not fix
  a wrong ordering.
- When sources and sinks along the process (cell death or cell division) are unknown,
  since they are critical for obtaining the correct real-time distribution.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0092
tools:
- name: MAPiT.m
  role: Applies the measure-preserving transformation tau to p_s(s,y), yielding the
    joint density on the real temporal or spatial scale
  repo: https://github.com/karstenkuritz/MAPiT
- name: preMAPiT.m
  role: Computes the measure-preserving transformation tau from the pseudotime-scale
    density and the prior true-scale distribution
  repo: https://github.com/karstenkuritz/MAPiT
- name: jointDensityPseudotimeY.m
  role: Estimates the joint density of pseudotime and marker signals from the nx1
    pseudotime vector and nx1 marker signal vector
  repo: https://github.com/karstenkuritz/MAPiT
- name: Wanderlust
  role: Pseudotime algorithm (part of the Cyt3 toolbox) supplying the nx1 pseudotime
    ordering that MAPiT transforms; also used to test robustness against DPT
  repo: https://github.com/dpeerlab/cyt3
- name: Kernel Density Estimation with Linked BCs
  role: Used to obtain the distribution in pseudotime for the cell cycle example
  repo: https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git
provenance:
  source_task_ids:
  - task_003
  - task_008
  source_papers:
  - doi: 10.1038/s41598-020-60400-z
    title: Reconstructing temporal and spatial dynamics from single-cell pseudotime
      using prior knowledge of real scale cell densities
  source_content_hash: sha256:ea565ba1cd8274d497dff29e303a24254edf6d19ee342235b0568f558c55c6da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/karstenkuritz/MAPiT
  tools:
  - Kernel Density Estimation with Linked BCs
  - MAPiT.m
  - Wanderlust
  - jointDensityPseudotimeY.m
  - preMAPiT.m
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# measure-preserving-pseudotime-to-real-scale-transformation

## Summary

MAPiT is a measure-preserving nonlinear transformation that maps pseudotime-scale joint densities of markers onto a real temporal or spatial scale, using prior knowledge of the true-scale cell density distribution. It conserves the area under the density curve so conditional marker profiles p(y|x) are recovered on the real scale.

## When to use

Use when snapshot single-cell data have been ordered in pseudotime (e.g. by Wanderlust or DPT) and marker trajectories must be interpreted on a real time or spatial scale for which a true-scale cell density distribution is available.

## When NOT to use

- When no prior knowledge of the true-scale cell density distribution exists — constructing a target distribution for processes like stem cell differentiation can be challenging and may require measured cell death and division rates at every stage.
- When the cell ordering from the trajectory inference algorithm itself is unreliable — MAPiT is reliant on the correctness of the provided cell order and does not fix a wrong ordering.
- When sources and sinks along the process (cell death or cell division) are unknown, since they are critical for obtaining the correct real-time distribution.

## Inputs

- nx1 vector of pseudotime values from a trajectory inference algorithm (e.g. Wanderlust from the Cyt3 toolbox, or Diffusion Maps/DPT)
- nx1 vector of marker signals from n single cells (e.g. DNA, RNA, Ki-67, p27, geminin from flow cytometry)
- Prior-knowledge true-scale cell density distribution (e.g. radial cell density versus distance from spheroid surface, or cell cycle stage density)
- Joint density p_s(s,y) computed with jointDensityPseudotimeY.m

## Outputs

- Measure-preserving transformation tau from preMAPiT.m
- Real-scale joint density p_x(x,y), normalized to integrate to 1
- Conditional marker profile p_x(y|x) on the real temporal or spatial scale
- Saved distributions (e.g. MAPiT_transformed_distributions.mat) and comparison figures (e.g. mapit_wanderlust_vs_dpt_comparison.png)

## How to apply

Generate a pseudotemporal ordering of cells with a trajectory-inference algorithm, then define the true-scale distribution (e.g. radial cell density versus distance from spheroid surface, or uniform cell density over the cell cycle). In MATLAB, compute the joint density of pseudotime and marker signals from the nx1 pseudotime vector and nx1 marker vector with jointDensityPseudotimeY.m. Obtain the measure-preserving transformation with preMAPiT.m using the prior true-scale density, then apply MAPiT.m to transform the joint density p_s(s,y) to the real-scale joint density p_x(x,y). Normalize p_x(x,y) so it integrates to 1, and derive conditional marker profiles p_x(y|x). Robustness to the pseudotime algorithm can be checked by transforming both Wanderlust and DPT orderings and verifying overlapping 50% confidence bands of the resulting profiles.

## Related tools

- **MAPiT.m** (Applies the measure-preserving transformation tau to p_s(s,y), yielding the joint density on the real temporal or spatial scale) — https://github.com/karstenkuritz/MAPiT
- **preMAPiT.m** (Computes the measure-preserving transformation tau from the pseudotime-scale density and the prior true-scale distribution) — https://github.com/karstenkuritz/MAPiT
- **jointDensityPseudotimeY.m** (Estimates the joint density of pseudotime and marker signals from the nx1 pseudotime vector and nx1 marker signal vector) — https://github.com/karstenkuritz/MAPiT
- **Wanderlust** (Pseudotime algorithm (part of the Cyt3 toolbox) supplying the nx1 pseudotime ordering that MAPiT transforms; also used to test robustness against DPT) — https://github.com/dpeerlab/cyt3
- **Kernel Density Estimation with Linked BCs** (Used to obtain the distribution in pseudotime for the cell cycle example) — https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git

## Examples

```
p_s = jointDensityPseudotimeY(pseudotime, marker); tau = preMAPiT(p_s, true_scale_density); p_x = MAPiT(p_s, tau);
```

## Evaluation signals

- Transformed density p_x(x,y) is non-negative everywhere and integrates to 1 on the real scale (valid probability density).
- MAPiT.m executes without error and the transformation conserves the area under the density curve (measure-preserving property).
- MAPiT-transformed Wanderlust and DPT profiles of markers such as Ki-67, RNA and p27 have overlapping 50% confidence intervals, showing independence of the pseudotime algorithm.
- For validation cases, MAPiT-derived profiles match independent measurements — e.g. Ki-67 distribution in an 11-day-old HCT116 spheroid closely matches microscopic Ki-67 intensity profiles in spheroid cross-sections (n=9 transversal quantifications), and geminin trajectories correspond to single-cell time-lapse microscopy kinetics with a geminin boost at approximately 7 h.

## Limitations

- MAPiT is reliant on the correctness of the cell order provided by the trajectory inference algorithm.
- Constructing a target real-scale distribution for applications such as stem cell differentiation is challenging; the paper only applied MAPiT to cases where the target distribution could be derived theoretically.
- For differentiation processes, cell death and cell division rates at every stage may need to be measured but are not provided.
- No labelled/sampling-timepoint data were used to validate MAPiT results; the authors only envision such validation.

## Evidence

- [abstract] MAPiT operates as measure-preserving transformation: "Here we describe MAPiT, an universal transformation method that recovers real-time dynamics of cellular processes from pseudotime scales by utilising knowledge of the distributions on the real scales."
- [readme] Workflow steps in README: "3. Get joint distribution of pseudotime and markers with [`jointDensityPseudotimeY.m`](jointDensityPseudotimeY.m) 4. Get transformation with [`preMAPiT.m`](preMAPiT.m) 5. Transform pseudotime"
- [readme] jointDensityPseudotimeY.m inputs: "The function [`jointDensityPseudotimeY.m`](jointDensityPseudotimeY.m) takes a `nx1` vector of pseudotime values from the pseudotime algorithm and a `nx1` vector of a marker signals from `n` single"
- [results] Robustness to pseudotime algorithm: "the distribution of cells on the spatial scale had to be taken into account, which in the case of radial symmetry of spheroids is the density of cells in relation to the distance from the spheroid"
- [intro] Reliance on ordering: "The method makes use of pseudotemporal ordering obtained from trajectory inference algorithms and hence is reliant on the correctness of the provided cell order."
- [discussion] Limitation on target distributions: "Constructing a target distribution for other processes i.e. the most common application of pseudotime single-cell analysis on stem cell differentiation, might be more challenging."
