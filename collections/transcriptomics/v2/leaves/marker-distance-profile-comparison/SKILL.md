---
name: marker-distance-profile-comparison
description: Use when you need to verify that MAPiT-transformed distance profiles
  of flow cytometry markers (e.g. DNA, RNA, Ki-67, p27) are robust to which markers
  are fed to the pseudotime algorithm, or when validating pseudotime-derived spatial
  arrangements against microscopy cross-sections.
when_to_use_negative:
- The target true-scale distribution cannot be derived or measured — e.g. stem cell
  differentiation processes where cell death and division rates at every stage are
  unknown
- The pseudotemporal cell order is not trustworthy, since MAPiT is reliant on the
  correctness of the provided cell order from the trajectory inference algorithm
- The data are already spatially resolved (e.g. imaging cross-sections); profiles
  can be compared directly without a pseudotime-to-space transformation
edam_topics:
- http://edamontology.org/topic_3934
tools:
- name: MAPiT (jointDensityPseudotimeY.m, preMAPiT.m, MAPiT.m)
  role: Computes the joint distribution of pseudotime and marker signals, builds the
    measure-preserving transformation, and converts pseudotime trajectories to distance-from-surface
    profiles for comparison
  repo: https://github.com/karstenkuritz/MAPiT
- name: Wanderlust (Cyt3 toolbox)
  role: Generates leave-one-out pseudotemporal orderings of spheroid cells using marker
    subsets as input
  repo: https://github.com/dpeerlab/cyt3
- name: Diffusion Maps / DPT
  role: Alternative pseudotime algorithm used to confirm that distance profiles are
    independent of the ordering method
  repo: https://www.helmholtz-muenchen.de/icb/research/groups/machine-learning/projects/dpt/index.html
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-60400-z
    title: Reconstructing temporal and spatial dynamics from single-cell pseudotime
      using prior knowledge of real scale cell densities
  source_content_hash: sha256:05b2e2dace0a7a8de5ee254be31b959fb379156f66c5cd2ca1f870d6ea651e23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/karstenkuritz/MAPiT
  tools:
  - Diffusion Maps / DPT
  - MAPiT (jointDensityPseudotimeY.m, preMAPiT.m, MAPiT.m)
  - MAPiT.m
  - Wanderlust (Cyt3 toolbox)
  - jointDensityPseudotimeY.m
  - preMAPiT.m
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3934
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# marker-distance-profile-comparison

## Summary

Validates MAPiT's spatial reconstruction of spheroid marker profiles by comparing distance-from-surface profiles across leave-one-out marker subsets (Wanderlust/DPT pseudotime inputs) against full-marker-set profiles and microscopically quantified Ki-67 spheroid cross-sections. Demonstrates the reconstruction is robust to the choice of markers used as pseudotime-algorithm input.

## When to use

Use when you need to verify that MAPiT-transformed distance profiles of flow cytometry markers (e.g. DNA, RNA, Ki-67, p27) are robust to which markers are fed to the pseudotime algorithm, or when validating pseudotime-derived spatial arrangements against microscopy cross-sections.

## When NOT to use

- The target true-scale distribution cannot be derived or measured — e.g. stem cell differentiation processes where cell death and division rates at every stage are unknown
- The pseudotemporal cell order is not trustworthy, since MAPiT is reliant on the correctness of the provided cell order from the trajectory inference algorithm
- The data are already spatially resolved (e.g. imaging cross-sections); profiles can be compared directly without a pseudotime-to-space transformation

## Inputs

- Single-cell flow cytometric measurements (FCS) of dissociated ~500 µm HCT116 spheroids stained for DNA (cell cycle stage), RNA (transcriptional activity), Ki-67 (proliferation) and p27 (quiescence), plus forward scatter
- nx1 vector of pseudotime values from Wanderlust or Diffusion Maps/DPT for each leave-one-out marker subset
- nx1 vector of marker signals per single cell (importable to MATLAB, e.g. from a .csv file)
- Radially symmetric spheroid cell-density distribution versus distance from the spheroid surface (true-scale distribution)

## Outputs

- MAPiT-derived marker intensity profiles as a function of distance from the spheroid surface, one set per leave-one-out marker subset and pseudotime algorithm
- Collated comparison figure (e.g. spheroid_loocv_distance_profiles.png) of all leave-one-out distance profiles vs. the full-marker-set reference and microscopic Ki-67 cross-section profiles

## How to apply

Load single-cell flow cytometric measurements of the 11-day-old HCT116 spheroid (markers DNA, RNA, Ki-67, p27 plus forward scatter) and, for each leave-one-out marker subset, compute a pseudotemporal ordering with Wanderlust (Cyt3 toolbox) and Diffusion Maps/DPT using only that subset. Call jointDensityPseudotimeY.m with the nx1 pseudotime vector and nx1 marker-signal vector, supplying the radially symmetric spheroid cell-density distribution versus distance from the spheroid surface as the true-scale distribution. Run preMAPiT.m to obtain the measure-preserving transformation, then MAPiT.m to transform each pseudotime trajectory to the spatial scale. Verify each subset's MAPiT-derived distance profiles against the correct distance-from-surface profiles from the full marker set and against Ki-67 cross-section profiles quantified microscopically in spheroid sections (n=9 transversal quantifications). Collate all leave-one-out profiles into a single comparison figure (spheroid_loocv_distance_profiles.png).

## Related tools

- **MAPiT (jointDensityPseudotimeY.m, preMAPiT.m, MAPiT.m)** (Computes the joint distribution of pseudotime and marker signals, builds the measure-preserving transformation, and converts pseudotime trajectories to distance-from-surface profiles for comparison) — https://github.com/karstenkuritz/MAPiT
- **Wanderlust (Cyt3 toolbox)** (Generates leave-one-out pseudotemporal orderings of spheroid cells using marker subsets as input) — https://github.com/dpeerlab/cyt3
- **Diffusion Maps / DPT** (Alternative pseudotime algorithm used to confirm that distance profiles are independent of the ordering method) — https://www.helmholtz-muenchen.de/icb/research/groups/machine-learning/projects/dpt/index.html

## Examples

```
run('examples/MAPiT_Spheroid_Validation.m') % MATLAB: reproduce Paper Fig 4 spheroid MAPiT vs. microscopy profiles
```

## Evaluation signals

- MAPiT-derived Ki-67 distance profile closely matches Ki-67 intensity profiles determined microscopically in spheroid cross-sections (n=9 transversal quantifications)
- Leave-one-out marker subsets reproduce distance profiles consistent with the correct profiles obtained with the full marker set, for both Wanderlust and DPT orderings
- DNA profile in the outermost spheroid layers shows the expected bimodality (G1 2N, S, G2/M 4N), while the majority of inner-layer cells are quiescent G1/G0 (2N)
- Profiles are consistent across the two pseudotime algorithms, indicating the result is not an artifact of the ordering method
- The spatial transformation used the radially symmetric spheroid cell-density distribution versus distance from the spheroid surface as the true-scale distribution

## Limitations

- MAPiT is reliant on the correctness of the cell order provided by the trajectory inference algorithm
- Constructing a target real-scale distribution for common applications like stem cell differentiation is challenging; the paper only applied MAPiT to cases where the target distribution could be derived theoretically
- Sources and sinks along the process, like cell death or cell division, are critical for obtaining the right real-time distribution but were not known in the presented examples

## Evidence

- [results] Leave-one-out consistency: "we conducted "leave one out cross validation", where we took subsets of the markers as input to the Wanderlust and DPT algorithms"
- [results] Microscopy validation of Ki-67 profiles: [evidence span withheld — anchor status: not_found]
- [results] True-scale spatial distribution: "the distribution of cells on the spatial scale had to be taken into account, which in the case of radial symmetry of spheroids is the density of cells in relation to the distance from the spheroid"
- [readme] Function entry point: "Entry point of pseudotime values and single cell data is step 3."
- [intro] Dependence on cell order: "The method makes use of pseudotemporal ordering obtained from trajectory inference algorithms and hence is reliant on the correctness of the provided cell order."
- [discussion] Sources and sinks caveat: "knowing sources and sinks along the process like cell death or cell division is critical for obtaining the right real-time distribution and subsequent correct transformation with MAPiT."
