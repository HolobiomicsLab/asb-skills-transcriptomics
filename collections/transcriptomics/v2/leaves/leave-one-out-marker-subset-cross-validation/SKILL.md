---
name: leave-one-out-marker-subset-cross-validation
description: Use when a MAPiT (or other measure-preserving pseudotime) reconstruction
  must be shown to be robust to the choice of markers fed into the pseudotime algorithm
  — e.g. spheroid flow cytometry data with multiple stains where input-marker bias
  is a concern.
when_to_use_negative:
- The true-scale target distribution cannot be derived theoretically — e.g. stem cell
  differentiation, where cell death and cell division rates at every stage are not
  measured
- The goal is real-time (temporal) rather than spatial reconstruction — the cell-density-versus-distance-from-surface
  distribution used here applies only to the spheroid spatial scale
- A pseudotime ordering whose cell order cannot be trusted is the input — MAPiT is
  reliant on the correctness of the provided cell order, so LOOCV on a bad ordering
  will not rescue it
tools:
- name: MAPiT
  role: Provides jointDensityPseudotimeY.m, preMAPiT.m and MAPiT.m to compute the
    joint density of pseudotime and markers, the measure-preserving transformation,
    and the pseudotime-to-spatial-scale transformation for each marker subset
  repo: https://github.com/karstenkuritz/MAPiT
- name: Wanderlust (Cyt3 toolbox)
  role: One of two pseudotime algorithms run on each leave-one-out marker subset as
    input to the cross-validation
  repo: https://github.com/dpeerlab/cyt3
- name: Diffusion Maps / DPT
  role: Second pseudotime algorithm run on each leave-one-out marker subset so robustness
    conclusions hold across algorithms
  repo: https://www.helmholtz-muenchen.de/icb/research/groups/machine-learning/projects/dpt/index.html
- name: Kernel Density Estimation with Linked BCs
  role: Used to obtain the distribution in pseudospace/pseudotime needed for the density
    estimates in the MAPiT workflow
  repo: https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-60400-z
    title: Reconstructing temporal and spatial dynamics from single-cell pseudotime
      using prior knowledge of real scale cell densities
  source_content_hash: sha256:6f3d5495c0022c3b7924a33593a4cce301e96a4a780518cd5035c2822bf92fd9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/karstenkuritz/MAPiT
  tools:
  - Diffusion Maps / DPT
  - Kernel Density Estimation with Linked BCs
  - MAPiT
  - MAPiT.m
  - Wanderlust (Cyt3 toolbox)
  - jointDensityPseudotimeY.m
  - preMAPiT.m
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# Leave-one-out marker-subset cross-validation

## Summary

A robustness validation scheme for MAPiT spatial reconstruction: pseudotime trajectories are recomputed with each leave-one-out subset of the measured markers (DNA, RNA, Ki-67, p27) as input, transformed to the distance-from-spheroid-surface scale, and the resulting marker distance profiles are checked against the full-marker-set profiles and microscopic Ki-67 cross-sections.

## When to use

Use when a MAPiT (or other measure-preserving pseudotime) reconstruction must be shown to be robust to the choice of markers fed into the pseudotime algorithm — e.g. spheroid flow cytometry data with multiple stains where input-marker bias is a concern.

## When NOT to use

- The true-scale target distribution cannot be derived theoretically — e.g. stem cell differentiation, where cell death and cell division rates at every stage are not measured
- The goal is real-time (temporal) rather than spatial reconstruction — the cell-density-versus-distance-from-surface distribution used here applies only to the spheroid spatial scale
- A pseudotime ordering whose cell order cannot be trusted is the input — MAPiT is reliant on the correctness of the provided cell order, so LOOCV on a bad ordering will not rescue it

## Inputs

- Single-cell flow cytometric measurements of dissociated ~500 µm HCT116 spheroids with markers DNA, RNA, Ki-67 and p27 (plus forward scatter)
- Per-marker-subset pseudotime value vectors (nx1) from Wanderlust and Diffusion Maps/DPT
- Radially symmetric spheroid cell-density distribution versus distance from the spheroid surface (true-scale distribution)
- Microscopically quantified Ki-67 transversal cross-section intensity profiles (n=9) as validation reference

## Outputs

- MAPiT-derived distance-from-surface marker profiles for every leave-one-out marker subset and both pseudotime algorithms
- Consolidated multi-panel figure spheroid_loocv_distance_profiles.png
- Qualitative consistency assessment versus full-marker-set profiles and microscopy-derived Ki-67 profiles

## How to apply

Load the single-cell flow cytometric measurements (for the reference case, 11-day-old HCT116 spheroids stained for DNA, RNA, Ki-67 and p27 plus forward scatter). For each leave-one-out marker subset, compute a pseudotemporal ordering using only that subset as input, with both the Wanderlust algorithm (from the Cyt3 toolbox) and Diffusion Maps/DPT so the result can be checked across algorithms. Feed each subset's pseudotime vector and marker signals into jointDensityPseudotimeY.m, using the radially symmetric spheroid cell-density distribution versus distance from the spheroid surface as the true-scale distribution, then obtain the measure-preserving transformation with preMAPiT.m and transform each pseudotime trajectory with MAPiT.m. Verify that the MAPiT-derived distance-from-surface profiles of every marker in each subset are consistent with the profiles obtained using the full marker set, and that the Ki-67 profile matches the microscopically quantified transversal cross-section profiles (n=9). Collate all leave-one-out profiles into a single figure (spheroid_loocv_distance_profiles.png) as the deliverable.

## Related tools

- **MAPiT** (Provides jointDensityPseudotimeY.m, preMAPiT.m and MAPiT.m to compute the joint density of pseudotime and markers, the measure-preserving transformation, and the pseudotime-to-spatial-scale transformation for each marker subset) — https://github.com/karstenkuritz/MAPiT
- **Wanderlust (Cyt3 toolbox)** (One of two pseudotime algorithms run on each leave-one-out marker subset as input to the cross-validation) — https://github.com/dpeerlab/cyt3
- **Diffusion Maps / DPT** (Second pseudotime algorithm run on each leave-one-out marker subset so robustness conclusions hold across algorithms) — https://www.helmholtz-muenchen.de/icb/research/groups/machine-learning/projects/dpt/index.html
- **Kernel Density Estimation with Linked BCs** (Used to obtain the distribution in pseudospace/pseudotime needed for the density estimates in the MAPiT workflow) — https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git

## Examples

```
run('examples/example_spheroid.m')  % MATLAB: spheroid analysis example (extend to leave-one-out marker subsets, then MAPiT.m transforms each pseudotime trajectory to the distance-from-surface scale)
```

## Evaluation signals

- MAPiT-derived distance profiles of every marker in every leave-one-out subset are consistent with the correct distance-from-surface profiles obtained with the full marker set
- The MAPiT Ki-67 profile closely matches Ki-67 intensity profiles determined microscopically in spheroid cross-sections (n=9 transversal quantifications)
- Agreement holds for both pseudotime algorithms (Wanderlust and DPT), not just one
- Expected biology is preserved: DNA content bimodal (G1 2N, S, G2/M 4N) in outermost spheroid layers with majority quiescent G1/G0 2N cells in inner layers
- All subset distance profiles are collated into the single deliverable figure spheroid_loocv_distance_profiles.png

## Limitations

- MAPiT and hence the cross-validation are reliant on the correctness of the pseudotemporal ordering provided by the trajectory inference algorithm
- The paper only validated against microscopy for Ki-67; no labelled/sampling-timepoint data were used to support or validate MAPiT results
- Constructing a target real-scale distribution for applications like stem cell differentiation is challenging; the approach was only applied where the target distribution could be derived theoretically (radial spheroid symmetry, cell cycle)
- Sources and sinks (cell death or cell division) along the process are not known in the presented examples but are critical for a correct real-time transformation

## Evidence

- [results] LOOCV finding: "we conducted "leave one out cross validation", where we took subsets of the markers as input to the Wanderlust and DPT algorithms"
- [results] Marker panel and stains: "cells were in addition stained for DNA as measure for cell cycle stage, RNA as indicator for transcriptional activity, Ki-67 as marker for proliferation and p27 as marker for quiescence"
- [results] True-scale spatial distribution: "the distribution of cells on the spatial scale had to be taken into account, which in the case of radial symmetry of spheroids is the density of cells in relation to the distance from the spheroid"
- [results] Microscopy validation reference: [evidence span withheld — anchor status: not_found]
- [readme] jointDensityPseudotimeY input contract: "Entry point of pseudotime values and single cell data is step 3. The function [`jointDensityPseudotimeY.m`](jointDensityPseudotimeY.m) takes a `nx1` vector of pseudotime values"
- [readme] Wanderlust dependency: "the examples use Wanderlust (part of the Cyt3 toolbox https://github.com/dpeerlab/cyt3) and Diffusion Maps"
