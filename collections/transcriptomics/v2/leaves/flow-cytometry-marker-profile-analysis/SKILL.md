---
name: flow-cytometry-marker-profile-analysis
description: Use when a static, unsynchronised flow cytometry dataset of a heterogeneous
  population (e.g. dissociated HCT116 spheroids stained for DNA, RNA, Ki-67, p27)
  must be mapped from pseudotime onto a real spatial (distance-from-surface) or temporal
  (cell age) scale.
when_to_use_negative:
- No defensible true-scale distribution can be defined — e.g. stem cell differentiation,
  where the paper notes constructing a target distribution is challenging and would
  require cell death and division rates at every stage
- The pseudotemporal cell order from the trajectory-inference algorithm is untrustworthy,
  since MAPiT is reliant on the correctness of the provided cell order
- Data are already time-resolved (e.g. single-cell time-lapse microscopy), so no pseudotime-to-real-scale
  transformation is needed
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: MAPiT
  role: MATLAB scripts (jointDensityPseudotimeY.m, preMAPiT.m, MAPiT.m) that perform
    the measure-preserving transformation of pseudotime and marker signals onto real
    temporal/spatial scales
  repo: https://github.com/karstenkuritz/MAPiT
- name: Wanderlust
  role: Pseudotemporal ordering of single-cell flow cytometry data; part of the Cyt3
    toolbox used by the MAPiT examples
  repo: https://github.com/dpeerlab/cyt3
- name: Diffusion Maps
  role: Alternative algorithm to derive pseudotime values from single-cell experimental
    data
  repo: https://www.helmholtz-muenchen.de/icb/research/groups/machine-learning/projects/dpt/index.html
- name: Kernel Density Estimation with Linked BCs
  role: Estimates the cell-age distribution in pseudotime for the cell cycle example
  repo: https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git
- name: MATLAB
  role: Execution environment in which the MAPiT example and validation scripts are
    run
  repo: https://github.com/karstenkuritz/MAPiT
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-60400-z
    title: Reconstructing temporal and spatial dynamics from single-cell pseudotime
      using prior knowledge of real scale cell densities
  source_content_hash: sha256:c5073e99f83a665ba14b9f5872cbcdee124f51b57b449a4a9e84f6ec967bfbfb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/karstenkuritz/MAPiT
  tools:
  - Diffusion Maps
  - Kernel Density Estimation with Linked BCs
  - MAPiT
  - MATLAB
  - Wanderlust
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# flow-cytometry-marker-profile-analysis

## Summary

Recover marker profiles (Ki-67, p27, DNA, RNA, geminin, forward scatter) on true temporal or spatial scales from static flow cytometric snapshots by transforming single-cell pseudotime with MAPiT, using prior knowledge of the real-scale cell-density distribution.

## When to use

Use when a static, unsynchronised flow cytometry dataset of a heterogeneous population (e.g. dissociated HCT116 spheroids stained for DNA, RNA, Ki-67, p27) must be mapped from pseudotime onto a real spatial (distance-from-surface) or temporal (cell age) scale.

## When NOT to use

- No defensible true-scale distribution can be defined — e.g. stem cell differentiation, where the paper notes constructing a target distribution is challenging and would require cell death and division rates at every stage
- The pseudotemporal cell order from the trajectory-inference algorithm is untrustworthy, since MAPiT is reliant on the correctness of the provided cell order
- Data are already time-resolved (e.g. single-cell time-lapse microscopy), so no pseudotime-to-real-scale transformation is needed

## Inputs

- nx1 vector of pseudotime values from Wanderlust or Diffusion Maps (importable as .csv into MATLAB)
- nx1 single-cell marker signal vectors from flow cytometry (DNA content, RNA, Ki-67, p27, forward scatter, mAG-hGeminin(1-110))
- True-scale cell-density distribution (radial density for ~500 µm spheroid with r = 270 µm necrotic core; KDE with linked BCs for cell age)
- HCT116 spheroid or NCI-H460/geminin FCS flow cytometry data (example_spheroid.m / example_cellcycle.m)

## Outputs

- Joint density of pseudotime and marker (from jointDensityPseudotimeY.m)
- Measure-preserving transformation map (preMAPiT.m)
- Marker profiles on the real scale: MAPiT-derived Ki-67/p27/DNA/forward-scatter intensity vs distance from spheroid surface (mapit_spheroid_profiles.png)
- Reconstructed geminin trajectory on the real-time cell-age scale (geminin_MAPiT_trajectory.csv)

## How to apply

Generate a pseudotemporal ordering of the n cells with Wanderlust (Cyt3 toolbox) or Diffusion Maps, then import the nx1 pseudotime vector and nx1 marker vectors (e.g. Ki-67, p27, DNA content, mAG-hGeminin) into MATLAB as .csv. Compute the joint density of pseudotime and each marker with jointDensityPseudotimeY.m, and define the true-scale distribution: for an ~500 µm HCT116 spheroid with necrotic core of r = 270 µm use the radial cell-density distribution of a sphere; for the cell-cycle example use kernel density estimation with linked boundary conditions (MColbrook implementation) to get the cell-age distribution. Run preMAPiT.m to obtain the measure-preserving transformation, then MAPiT.m to project pseudotime trajectories onto the real scale. Plot the transformed marker intensities versus distance from the spheroid surface (mapit_spheroid_profiles.png) or versus real time (geminin_MAPiT_trajectory.csv), and cross-check with MAPiT_Spheroid_Validation.m or MAPiT_CellCycle_Validation.m.

## Related tools

- **MAPiT** (MATLAB scripts (jointDensityPseudotimeY.m, preMAPiT.m, MAPiT.m) that perform the measure-preserving transformation of pseudotime and marker signals onto real temporal/spatial scales) — https://github.com/karstenkuritz/MAPiT
- **Wanderlust** (Pseudotemporal ordering of single-cell flow cytometry data; part of the Cyt3 toolbox used by the MAPiT examples) — https://github.com/dpeerlab/cyt3
- **Diffusion Maps** (Alternative algorithm to derive pseudotime values from single-cell experimental data) — https://www.helmholtz-muenchen.de/icb/research/groups/machine-learning/projects/dpt/index.html
- **Kernel Density Estimation with Linked BCs** (Estimates the cell-age distribution in pseudotime for the cell cycle example) — https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git
- **MATLAB** (Execution environment in which the MAPiT example and validation scripts are run) — https://github.com/karstenkuritz/MAPiT

## Examples

```
run('examples/example_spheroid.m')
```

## Evaluation signals

- MAPiT-derived Ki-67 spatial distribution closely matches Ki-67 intensity profiles quantified microscopically in spheroid cross-sections (n = 9 transversal quantifications)
- MAPiT-transformed geminin trajectory corresponds excellently to time-lapse microscopy kinetics, reproducing the boost in geminin intensity at approximately 7 h indicating S-phase onset
- Recovered spatial structure shows a proliferative outer layer with bimodal 2N/4N DNA content surrounding quiescent inner layers in 2N G1/G0
- Leave-one-out cross validation with marker subsets as Wanderlust/DPT input yields consistent profiles, confirming robustness to marker choice
- Transformation is measure-preserving: the joint density on pseudotime and the true-scale distribution must map cell counts consistently across scales

## Limitations

- MAPiT relies on the correctness of the cell order provided by the trajectory inference algorithm
- Knowing sources and sinks along the process, like cell death or cell division, is critical for obtaining the right real-time distribution and correct transformation; these were not known in the presented examples
- No labelled/sampling-timepoint data were used to support or validate MAPiT results; the authors only envision such validation
- MAPiT has no dependencies to other third-party Matlab interfaces/toolboxes, but the examples require Wanderlust (Cyt3), Diffusion Maps and KDE-with-linked-BCs code to be available

## Evidence

- [results] Marker staining rationale for spheroid cytometry: "cells were in addition stained for DNA as measure for cell cycle stage, RNA as indicator for transcriptional activity, Ki-67 as marker for proliferation and p27 as marker for quiescence"
- [results] Spheroid Ki-67 validation against microscopy: [evidence span withheld — anchor status: not_found]
- [results] Geminin trajectory validation: "Temporal trajectories of geminin obtained with MAPiT corresponded excellently to geminin kinetics obtained by single cell time-lapse microscopy"
- [readme] Pseudotime algorithms used by the examples: "the examples use Wanderlust (part of the Cyt3 toolbox https://github.com/dpeerlab/cyt3) and Diffusion Maps"
- [intro] Dependency on correct cell order: "The method makes use of pseudotemporal ordering obtained from trajectory inference algorithms and hence is reliant on the correctness of the provided cell order."
- [discussion] Need for known sources and sinks: "knowing sources and sinks along the process like cell death or cell division is critical for obtaining the right real-time distribution and subsequent correct transformation with MAPiT."
