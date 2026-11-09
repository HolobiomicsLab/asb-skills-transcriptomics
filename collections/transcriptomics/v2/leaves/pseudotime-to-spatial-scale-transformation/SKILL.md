---
name: pseudotime-to-spatial-scale-transformation
description: Use when single-cell flow cytometry data of radially symmetric multicellular
  spheroids have been ordered in pseudotime and marker profiles (Ki-67, p27, DNA,
  RNA) must be recovered versus distance from the spheroid surface.
when_to_use_negative:
- When no theoretically or empirically justified true-scale distribution can be constructed,
  e.g. stem cell differentiation, where target cell densities and death/division rates
  at every stage are unknown
- When the pseudotime ordering from the trajectory inference algorithm is unreliable,
  since MAPiT inherits errors in cell order
- When the biological system lacks radial symmetry or a known cell-density profile
  that can serve as the transformation target
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: MAPiT
  role: MATLAB scripts (jointDensityPseudotimeY.m, preMAPiT.m, MAPiT.m) performing
    the measure-preserving transformation of pseudotime to the spatial scale
  repo: https://github.com/karstenkuritz/MAPiT
- name: Wanderlust
  role: Produces the pseudotemporal ordering of spheroid single cells that MAPiT transforms;
    part of the Cyt3 toolbox
  repo: https://github.com/dpeerlab/cyt3
- name: Diffusion Maps / DPT
  role: Alternative pseudotime algorithm providing the nx1 pseudotime vector input
    to MAPiT
  repo: https://www.helmholtz-muenchen.de/icb/research/groups/machine-learning/projects/dpt/index.html
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-60400-z
    title: Reconstructing temporal and spatial dynamics from single-cell pseudotime
      using prior knowledge of real scale cell densities
  source_content_hash: sha256:f82b4c37a9fbca68541e8903a113c49a87b19ef626d24a4279f35ec7bd0fd1d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/karstenkuritz/MAPiT
  tools:
  - Diffusion Maps / DPT
  - MAPiT
  - MAPiT.m
  - Wanderlust
  - jointDensityPseudotimeY.m
  - preMAPiT.m
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# pseudotime-to-spatial-scale-transformation

## Summary

MAPiT is a measure-preserving transformation (MATLAB implementation: jointDensityPseudotimeY.m, preMAPiT.m, MAPiT.m) that converts pseudotime trajectories from Wanderlust or Diffusion Maps into real spatial scales, e.g. distance from the spheroid surface, using prior knowledge of the true-scale cell-density distribution.

## When to use

Use when single-cell flow cytometry data of radially symmetric multicellular spheroids have been ordered in pseudotime and marker profiles (Ki-67, p27, DNA, RNA) must be recovered versus distance from the spheroid surface.

## When NOT to use

- When no theoretically or empirically justified true-scale distribution can be constructed, e.g. stem cell differentiation, where target cell densities and death/division rates at every stage are unknown
- When the pseudotime ordering from the trajectory inference algorithm is unreliable, since MAPiT inherits errors in cell order
- When the biological system lacks radial symmetry or a known cell-density profile that can serve as the transformation target

## Inputs

- nx1 vector of pseudotime values from Wanderlust or Diffusion Maps (e.g. imported from a .csv file into MATLAB)
- nx1 marker signal vectors from single-cell flow cytometry of dissociated HCT116 spheroids (DNA, RNA, Ki-67, p27, forward scatter)
- True-scale radial cell-density distribution of a sphere (distance from spheroid surface) derived from spheroid diameter and necrotic core radius

## Outputs

- Marker intensity profiles (Ki-67, p27, DNA content, forward scatter) versus distance from the spheroid surface (e.g. mapit_spheroid_profiles.png)
- Reconstructed spatial positions of single cells within the intact spheroid prior to dissociation

## How to apply

Generate a pseudotemporal ordering of cells with Wanderlust (Cyt3 toolbox) or Diffusion Maps/DPT, then compute the joint density of pseudotime and each marker signal from nx1 vectors using jointDensityPseudotimeY.m. Define the true-scale distribution as the radial cell-density distribution of a sphere (e.g. an approximately 500 µm HCT116 spheroid with a necrotic core of r = 270 µm under the linear growth model) and obtain the measure-preserving transformation with preMAPiT.m. Transform the pseudotime trajectories to the distance-from-surface scale with MAPiT.m and plot marker intensities against distance from the spheroid surface. The rationale is that pseudotime values alone carry no physical scale; MAPiT enforces that the transformed cell positions reproduce the known radial cell density, mapping marker joint densities onto the spatial scale. Correctness of the input cell order is a prerequisite, since the method is reliant on the correctness of the ordering provided by the trajectory inference algorithm.

## Related tools

- **MAPiT** (MATLAB scripts (jointDensityPseudotimeY.m, preMAPiT.m, MAPiT.m) performing the measure-preserving transformation of pseudotime to the spatial scale) — https://github.com/karstenkuritz/MAPiT
- **Wanderlust** (Produces the pseudotemporal ordering of spheroid single cells that MAPiT transforms; part of the Cyt3 toolbox) — https://github.com/dpeerlab/cyt3
- **Diffusion Maps / DPT** (Alternative pseudotime algorithm providing the nx1 pseudotime vector input to MAPiT) — https://www.helmholtz-muenchen.de/icb/research/groups/machine-learning/projects/dpt/index.html

## Examples

```
In MATLAB: run example_spheroid.m from github.com/karstenkuritz/MAPiT (jointDensityPseudotimeY.m -> preMAPiT.m -> MAPiT.m)
```

## Evaluation signals

- MAPiT-derived Ki-67 distribution from 11-day-old HCT116 spheroid flow cytometry closely matches Ki-67 intensity profiles quantified microscopically in spheroid cross-sections (n = 9 transversal quantifications)
- Reconstructed profiles show the expected biology: a proliferative outer layer with bimodal 2N/4N DNA content surrounding quiescent inner layers in 2N G1/G0
- Leave-one-out cross validation: distance profiles obtained with any marker subset as pseudotime input remain consistent with profiles from the full marker set
- Comparison of pseudotime orderings from Wanderlust vs. DPT yields consistent MAPiT spatial reconstructions

## Limitations

- MAPiT relies on the correctness of the cell order provided by the trajectory inference algorithm
- Constructing a target real-scale distribution for applications like stem cell differentiation is challenging; the paper only applied MAPiT where the target distribution could be derived theoretically
- Sources and sinks (cell death or cell division) along the process are not known in the presented examples but are critical for a correct real-time distribution and transformation

## Evidence

- [results] MAPiT recovered spatial positions from flow cytometry: [evidence span withheld — anchor status: not_found]
- [readme] Method workflow from README: "3. Get joint distribution of pseudotime and markers with [`jointDensityPseudotimeY.m`](jointDensityPseudotimeY.m) 4. Get transformation with [`preMAPiT.m`](preMAPiT.m)"
- [results] Radial symmetry and density distribution: "the distribution of cells on the spatial scale had to be taken into account, which in the case of radial symmetry of spheroids is the density of cells in relation to the distance from the spheroid"
- [intro] Reliance on correct cell ordering: "The method makes use of pseudotemporal ordering obtained from trajectory inference algorithms and hence is reliant on the correctness of the provided cell order."
- [readme] Input format for MAPiT functions: "The function [`jointDensityPseudotimeY.m`](jointDensityPseudotimeY.m) takes a `nx1` vector of pseudotime values from the pseudotime algorithm"
