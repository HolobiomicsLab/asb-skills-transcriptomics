---
name: joint-density-change-of-variables
description: Use when a pseudotime-ordered single-cell dataset needs marker trajectories
  expressed on a real time or spatial scale, and a prior cell-density distribution
  on that true scale is available.
when_to_use_negative:
- No prior knowledge of the real-scale cell-density distribution is available (e.g.
  stem cell differentiation, where the paper notes constructing a target distribution
  is challenging)
- Sources and sinks such as cell death or cell division rates along the process are
  unknown, since these are critical for obtaining the correct real-time distribution
- The pseudotime ordering from the trajectory inference algorithm is unreliable, as
  MAPiT is reliant on the correctness of the provided cell order
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: MAPiT.m
  role: Applies the measure-preserving transformation tau to p_s(s,y) to yield the
    joint density on the real scale
  repo: https://github.com/karstenkuritz/MAPiT
- name: preMAPiT.m
  role: Computes the measure-preserving transformation tau from the pseudotime density
    and the true-scale distribution
  repo: https://github.com/karstenkuritz/MAPiT
- name: jointDensityPseudotimeY.m
  role: Computes the joint density of pseudotime and marker signal from nx1 pseudotime
    and marker vectors
  repo: https://github.com/karstenkuritz/MAPiT
- name: MATLAB
  role: Runtime environment in which the MAPiT scripts are adapted and called for
    each application case
- name: Kernel Density Estimation with Linked BCs
  role: Used to obtain the density distribution in pseudotime for the cell cycle example
  repo: https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-020-60400-z
    title: Reconstructing temporal and spatial dynamics from single-cell pseudotime
      using prior knowledge of real scale cell densities
  source_content_hash: sha256:a715b1e52b7c4e162749a6a2c4ba7083fa9838fc1405da427ca13fc559dfdf8d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/karstenkuritz/MAPiT
  tools:
  - Kernel Density Estimation with Linked BCs
  - MAPiT.m
  - MATLAB
  - jointDensityPseudotimeY.m
  - preMAPiT.m
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# joint-density-change-of-variables

## Summary

MAPiT applies a measure-preserving nonlinear change of variables (transformation tau from preMAPiT.m) to the joint density p_s(s,y) computed on the pseudotime scale, yielding p_x(x,y) on the real temporal or spatial scale so that conditional marker profiles p_x(y|x) can be recovered from snapshot single-cell data.

## When to use

Use when a pseudotime-ordered single-cell dataset needs marker trajectories expressed on a real time or spatial scale, and a prior cell-density distribution on that true scale is available.

## When NOT to use

- No prior knowledge of the real-scale cell-density distribution is available (e.g. stem cell differentiation, where the paper notes constructing a target distribution is challenging)
- Sources and sinks such as cell death or cell division rates along the process are unknown, since these are critical for obtaining the correct real-time distribution
- The pseudotime ordering from the trajectory inference algorithm is unreliable, as MAPiT is reliant on the correctness of the provided cell order

## Inputs

- nx1 vector of pseudotime values from a trajectory inference algorithm (e.g. Wanderlust, Diffusion Pseudotime)
- nx1 vector of marker signal values from n single cells (e.g. DNA, geminin, Ki-67 intensities from flow cytometry)
- Predefined true-scale cell-density distribution (e.g. radial cell density versus distance from spheroid surface)
- Measure-preserving transformation tau computed by preMAPiT.m

## Outputs

- Transformed joint density p_x(x,y) on the real temporal or spatial scale
- Conditional marker profile p_x(y|x) on the real scale
- Saved distributions in MAPiT_transformed_distributions.mat

## How to apply

In MATLAB, first compute the joint density of pseudotime and marker signal with jointDensityPseudotimeY.m, which takes an nx1 pseudotime vector from the trajectory algorithm (e.g. Wanderlust, DPT) and an nx1 marker signal vector from n single cells. Obtain the measure-preserving transformation tau with preMAPiT.m, using the defined true-scale distribution (e.g. the radial cell-density distribution of spheroids for spatial analysis). Run MAPiT.m to apply tau to p_s(s,y), producing the joint density p_x(x,y) on the real scale; normalize it so it integrates to 1. Then derive the conditional marker profile p_x(y|x) from the transformed joint density and save both to MAPiT_transformed_distributions.mat. The transformation is measure-preserving, conserving the area under the density curve, which is why p(y|x) profiles are recovered correctly on the real scale.

## Related tools

- **MAPiT.m** (Applies the measure-preserving transformation tau to p_s(s,y) to yield the joint density on the real scale) — https://github.com/karstenkuritz/MAPiT
- **preMAPiT.m** (Computes the measure-preserving transformation tau from the pseudotime density and the true-scale distribution) — https://github.com/karstenkuritz/MAPiT
- **jointDensityPseudotimeY.m** (Computes the joint density of pseudotime and marker signal from nx1 pseudotime and marker vectors) — https://github.com/karstenkuritz/MAPiT
- **MATLAB** (Runtime environment in which the MAPiT scripts are adapted and called for each application case)
- **Kernel Density Estimation with Linked BCs** (Used to obtain the density distribution in pseudotime for the cell cycle example) — https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git

## Examples

```
run MAPiT.m % applies tau from preMAPiT.m to p_s(s,y) and saves p_x(x,y) and p_x(y|x) to MAPiT_transformed_distributions.mat
```

## Evaluation signals

- Transformed densities are non-negative everywhere
- The transformed joint density p_x(x,y) integrates to 1 on the real temporal or spatial scale
- MAPiT.m executes without error in the MATLAB session
- MAPiT-derived marker trajectories correspond to independent validation measurements, e.g. geminin kinetics matched single-cell time-lapse microscopy and Ki-67 profiles matched spheroid cross-section confocal quantifications

## Limitations

- MAPiT is reliant on the correctness of the cell order provided by the trajectory inference algorithm
- Constructing a target real-scale distribution for processes like stem cell differentiation is challenging; the paper only applied MAPiT to examples where the target distribution could be derived theoretically
- Cell death and cell division rates (sources and sinks) at every stage of the process may need to be measured for a correct real-time transformation but were not provided in the presented examples
- No labelled/sampling-timepoint data were used to support or validate MAPiT results in the paper

## Evidence

- [intro] MAPiT operates as a measure-preserving nonlinear transformation conserving the area under the density curve: "Here, we developed a measure-preserving transformation of pseudotime into real-time, a MAP of pseudotime into Time, in short MAPiT16."
- [readme] jointDensityPseudotimeY.m input contract: "The function [`jointDensityPseudotimeY.m`](jointDensityPseudotimeY.m) takes a `nx1` vector of pseudotime values from the pseudotime algorithm"
- [readme] Workflow step ordering from README: "3. Get joint distribution of pseudotime and markers with
[`jointDensityPseudotimeY.m`](jointDensityPseudotimeY.m)
4. Get transformation with [`preMAPiT.m`](preMAPiT.m)
5. Transform pseudotime"
- [results] True-scale distribution requirement for spheroid spatial analysis: "the distribution of cells on the spatial scale had to be taken into account, which in the case of radial symmetry of spheroids is the density of cells in relation to the distance from the spheroid"
- [results] Validation against microscopy: "Temporal trajectories of geminin obtained with MAPiT corresponded excellently to geminin kinetics obtained by single cell time-lapse microscopy"
