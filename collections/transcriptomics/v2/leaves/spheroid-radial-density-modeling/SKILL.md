---
name: spheroid-radial-density-modeling
description: Use when transforming single-cell pseudotime from dissociated multicellular
  spheroids into spatial distance-from-surface coordinates, requiring a spherical-shell
  radial cell-density distribution as the MAPiT target.
when_to_use_negative:
- When the real-scale distribution cannot be derived theoretically, e.g. stem cell
  differentiation, where constructing a target distribution is challenging and cell
  death/division rates at every stage may be needed
- When the biological process lacks radial symmetry or a known cell-density distribution
  on the true scale, since MAPiT requires the target density as prior knowledge
- When the cell order from the trajectory inference algorithm itself is in question
  — MAPiT is reliant on the correctness of the provided cell order
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3306
tools:
- name: jointDensityPseudotimeY.m
  role: Computes the joint density of pseudotime and marker signals from nx1 vectors,
    the input to the measure-preserving transformation
  repo: https://github.com/karstenkuritz/MAPiT
- name: preMAPiT.m
  role: Solves tau(s) = P_x^{-1}(P_s(s)) using the radial cell-density distribution
    as target, enforcing strict monotonicity of the map
  repo: https://github.com/karstenkuritz/MAPiT
- name: MAPiT.m
  role: Transforms pseudotime trajectories to the distance-from-spheroid-surface scale
    using the precomputed transformation
  repo: https://github.com/karstenkuritz/MAPiT
- name: MAPiT_Spheroid_Validation.m
  role: 'Cross-checks the reconstruction against microscopy cross-section profiles
    (Paper Fig 4: Spheroid - MAPiT vs. microscopy)'
  repo: https://github.com/karstenkuritz/MAPiT
- name: Kernel Density Estimation with Linked BCs
  role: Estimates the smoothed pseudotime cell-density distribution needed for the
    measure-preserving transformation
  repo: https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git
- name: Wanderlust
  role: Provides the pseudotemporal ordering of cells (part of the Cyt3 toolbox) that
    is transformed to the spatial scale
  repo: https://github.com/dpeerlab/cyt3
- name: Diffusion Maps
  role: Alternative pseudotemporal ordering algorithm (DPT) used to derive pseudotime
    values from single-cell experimental data
  repo: https://www.helmholtz-muenchen.de/icb/research/groups/machine-learning/projects/dpt/index.html
provenance:
  source_task_ids:
  - task_004
  - task_007
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-60400-z
    title: Reconstructing temporal and spatial dynamics from single-cell pseudotime
      using prior knowledge of real scale cell densities
  source_content_hash: sha256:d98e4700dc08c1270b0e21af1797269e8ca3e1de5a08edc25b74c89762fa1398
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/karstenkuritz/MAPiT
  tools:
  - Diffusion Maps
  - Kernel Density Estimation with Linked BCs
  - MAPiT.m
  - MAPiT_Spheroid_Validation.m
  - Wanderlust
  - jointDensityPseudotimeY.m
  - preMAPiT.m
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3306
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# spheroid-radial-density-modeling

## Summary

Constructs the theoretically derived radial cell-density distribution of a spherical multicellular spheroid (e.g. an ~500 µm HCT116 spheroid with a necrotic core at r = 270 µm under the linear growth model) and uses it as the true-scale target distribution for MAPiT's measure-preserving transformation of pseudotime into distance-from-spheroid-surface.

## When to use

Use when transforming single-cell pseudotime from dissociated multicellular spheroids into spatial distance-from-surface coordinates, requiring a spherical-shell radial cell-density distribution as the MAPiT target.

## When NOT to use

- When the real-scale distribution cannot be derived theoretically, e.g. stem cell differentiation, where constructing a target distribution is challenging and cell death/division rates at every stage may be needed
- When the biological process lacks radial symmetry or a known cell-density distribution on the true scale, since MAPiT requires the target density as prior knowledge
- When the cell order from the trajectory inference algorithm itself is in question — MAPiT is reliant on the correctness of the provided cell order

## Inputs

- nx1 vector of pseudotime values from Wanderlust or Diffusion Maps/DPT, e.g. imported from a .csv file
- nx1 marker signal vectors (DNA, RNA, Ki-67, p27, forward scatter) from dissociated HCT116 spheroid flow cytometry
- Prior knowledge of spheroid geometry: diameter (~500 µm) and necrotic core radius (r = 270 µm) under the linear growth model

## Outputs

- Measure-preserving pseudotime-to-distance transformation saved as transformation_map.mat
- MAPiT-derived marker intensity profiles versus distance from the spheroid surface (e.g. mapit_spheroid_profiles.png)
- Leave-one-out cross-validation distance profiles (spheroid_loocv_distance_profiles.png)

## How to apply

For radially symmetric spheroids, derive the true-scale distribution of cells versus distance from the spheroid surface analytically, assuming a linear growth model (e.g. ~500 µm diameter HCT116 spheroid with necrotic core at r = 270 µm). Use this spherical-shell radial density as the target real-scale distribution in step 2 of the MAPiT workflow, alongside the pseudotime cell-density distribution obtained by kernel density estimation with linked boundary conditions. Compute the measure-preserving map tau(s) = P_x^{-1}(P_s(s)) with preMAPiT.m, enforcing strict monotonicity, then apply MAPiT.m to transform pseudotime trajectories to the spatial scale. Joint densities of pseudotime and markers (DNA, RNA, Ki-67, p27) are computed with jointDensityPseudotimeY.m from nx1 vectors imported from a .csv file. The resulting distance-from-surface profiles should reproduce the proliferative outer layer (bimodal 2N/4N DNA) and quiescent inner 2N G1/G0 layers, and are validated against microscopically quantified Ki-67 cross-section profiles (n = 9 transversal quantifications).

## Related tools

- **jointDensityPseudotimeY.m** (Computes the joint density of pseudotime and marker signals from nx1 vectors, the input to the measure-preserving transformation) — https://github.com/karstenkuritz/MAPiT
- **preMAPiT.m** (Solves tau(s) = P_x^{-1}(P_s(s)) using the radial cell-density distribution as target, enforcing strict monotonicity of the map) — https://github.com/karstenkuritz/MAPiT
- **MAPiT.m** (Transforms pseudotime trajectories to the distance-from-spheroid-surface scale using the precomputed transformation) — https://github.com/karstenkuritz/MAPiT
- **MAPiT_Spheroid_Validation.m** (Cross-checks the reconstruction against microscopy cross-section profiles (Paper Fig 4: Spheroid - MAPiT vs. microscopy)) — https://github.com/karstenkuritz/MAPiT
- **Kernel Density Estimation with Linked BCs** (Estimates the smoothed pseudotime cell-density distribution needed for the measure-preserving transformation) — https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git
- **Wanderlust** (Provides the pseudotemporal ordering of cells (part of the Cyt3 toolbox) that is transformed to the spatial scale) — https://github.com/dpeerlab/cyt3
- **Diffusion Maps** (Alternative pseudotemporal ordering algorithm (DPT) used to derive pseudotime values from single-cell experimental data) — https://www.helmholtz-muenchen.de/icb/research/groups/machine-learning/projects/dpt/index.html

## Examples

```
run('examples/example_spheroid.m')  % MATLAB: load HCT116 flow cytometry, jointDensityPseudotimeY.m, preMAPiT.m with radial cell-density distribution, MAPiT.m
```

## Evaluation signals

- The transformation map is strictly monotonically increasing and maps the pseudotime CDF onto the target real-scale CDF (qualitative CDF-match check)
- MAPiT-derived Ki-67 spatial distribution closely matches Ki-67 intensity profiles quantified microscopically in spheroid cross-sections (n = 9 transversal quantifications)
- DNA content of the outermost layers shows the expected bimodality (G1 2N, S, G2/M 4N) while the majority of inner-layer cells are quiescent 2N G1/G0
- Leave-one-out marker-subset cross validation yields distance profiles consistent with the full-marker-set profiles, showing robustness to pseudotime algorithm input choice
- Distance profiles are consistent across both Wanderlust and Diffusion Maps/DPT pseudotemporal orderings

## Limitations

- MAPiT is reliant on the correctness of the cell order provided by the trajectory inference algorithm
- Constructing a target distribution for other processes (e.g. stem cell differentiation) might be more challenging; cell death and cell division rates at every stage may be needed
- Sources and sinks along the process (cell death or cell division) are not known in the presented examples but are critical for a correct real-time transformation
- No labelled/sampling-timepoint data were used to validate MAPiT results; the authors only envision such validation

## Evidence

- [results] Radial density as true-scale distribution: "the distribution of cells on the spatial scale had to be taken into account, which in the case of radial symmetry of spheroids is the density of cells in relation to the distance from the spheroid"
- [readme] Joint density function input: "The function [`jointDensityPseudotimeY.m`](jointDensityPseudotimeY.m) takes a `nx1` vector of pseudotime values from the pseudotime algorithm"
- [results] Ki-67 validation: [evidence span withheld — anchor status: not_found]
- [methods] Monotonicity requirement: [evidence span withheld — anchor status: not_found]
- [results] Leave-one-out robustness: [evidence span withheld — anchor status: not_found]
