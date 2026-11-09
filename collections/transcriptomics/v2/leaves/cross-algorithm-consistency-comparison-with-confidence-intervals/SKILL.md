---
name: cross-algorithm-consistency-comparison-with-confidence-intervals
description: Use when a downstream transformation or trajectory conclusion must be
  shown to be robust to the choice of pseudotime algorithm, e.g. MAPiT-transformed
  Ki-67, RNA or p27 spatial profiles derived from Wanderlust vs DPT orderings.
when_to_use_negative:
- Only one pseudotime ordering is available, so no cross-algorithm comparison can
  be made
- No prior knowledge of the true-scale cell density distribution exists (MAPiT's transformation
  itself is then undefined)
- The goal is to compare algorithms at the pseudotime level rather than on the transformed
  real scale — MAPiT is reliant on the correctness of the provided cell order and
  does not fix a wrong ordering
edam_operation: http://edamontology.org/operation_2424
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: MAPiT
  role: MATLAB scripts (jointDensityPseudotimeY.m, preMAPiT.m, MAPiT.m) that transform
    each pseudotime ordering to the true spatial/temporal scale for comparison
  repo: https://github.com/karstenkuritz/MAPiT
- name: Wanderlust
  role: First pseudotime algorithm producing one of the two orderings being compared;
    used from the Cyt3 toolbox
  repo: https://github.com/dpeerlab/cyt3
- name: Diffusion Pseudotime (DPT)
  role: Second, independent pseudotime algorithm producing the comparison ordering
  repo: https://www.helmholtz-muenchen.de/icb/research/groups/machine-learning/projects/dpt/index.html
- name: MATLAB
  role: Environment in which the MAPiT transformation and confidence-band plotting
    are executed
  repo: https://github.com/karstenkuritz/MAPiT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-60400-z
    title: Reconstructing temporal and spatial dynamics from single-cell pseudotime
      using prior knowledge of real scale cell densities
  source_content_hash: sha256:08187e30423c01a01b3d8c99cb50873b01e7e4c61eb41dd7c6f286cf9175f5e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/karstenkuritz/MAPiT
  tools:
  - Diffusion Pseudotime (DPT)
  - MAPiT
  - MAPiT.m
  - MATLAB
  - Wanderlust
  - jointDensityPseudotimeY.m
  - preMAPiT.m
  edam_operation: http://edamontology.org/operation_2424
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# Cross-algorithm consistency comparison with confidence intervals

## Summary

A validation strategy that runs two independent pseudotime algorithms (Wanderlust and Diffusion Maps/DPT) on the same single-cell dataset, MAPiT-transforms each ordering to the true spatial or temporal scale, and checks whether marker profiles with 50% confidence bands overlap — demonstrating the result is not an artifact of the trajectory-inference algorithm.

## When to use

Use when a downstream transformation or trajectory conclusion must be shown to be robust to the choice of pseudotime algorithm, e.g. MAPiT-transformed Ki-67, RNA or p27 spatial profiles derived from Wanderlust vs DPT orderings.

## When NOT to use

- Only one pseudotime ordering is available, so no cross-algorithm comparison can be made
- No prior knowledge of the true-scale cell density distribution exists (MAPiT's transformation itself is then undefined)
- The goal is to compare algorithms at the pseudotime level rather than on the transformed real scale — MAPiT is reliant on the correctness of the provided cell order and does not fix a wrong ordering

## Inputs

- nx1 pseudotime vector from Wanderlust (Cyt3 toolbox) for n single cells
- nx1 pseudotime vector from Diffusion Maps/DPT for the same n cells
- nx1 marker signal vectors (e.g. Ki-67, RNA, p27) from flow cytometric measurements
- Prior knowledge true-scale cell density distribution (e.g. radial cell density as a function of distance from spheroid surface)

## Outputs

- MAPiT-transformed marker profiles on the common real scale for each algorithm
- Comparison plot with overlapping 50% confidence bands (e.g. mapit_wanderlust_vs_dpt_comparison.png)
- Qualitative robustness verdict: overlapping vs non-overlapping confidence intervals

## How to apply

Run Wanderlust (from the Cyt3 toolbox) and Diffusion Maps/DPT on the same flow-cytometric single-cell dataset (e.g. dissociated HCT116 spheroids stained for DNA, RNA, Ki-67 and p27) to obtain two independent nx1 pseudotime vectors. Import each pseudotime vector together with the nx1 marker signal vector into MATLAB and estimate the joint density with jointDensityPseudotimeY.m. Compute the measure-preserving transformation with preMAPiT.m using prior knowledge of the true-scale cell density distribution (e.g. radial cell density vs distance from spheroid surface), then transform both trajectories to the real scale with MAPiT.m. Finally, plot the transformed marker profiles from both orderings on the common real scale with 50% confidence bands. Rationale: pseudotime orderings from different algorithms may deviate, but if MAPiT-derived true-scale profiles agree, the biological conclusion does not depend on the ordering algorithm.

## Related tools

- **MAPiT** (MATLAB scripts (jointDensityPseudotimeY.m, preMAPiT.m, MAPiT.m) that transform each pseudotime ordering to the true spatial/temporal scale for comparison) — https://github.com/karstenkuritz/MAPiT
- **Wanderlust** (First pseudotime algorithm producing one of the two orderings being compared; used from the Cyt3 toolbox) — https://github.com/dpeerlab/cyt3
- **Diffusion Pseudotime (DPT)** (Second, independent pseudotime algorithm producing the comparison ordering) — https://www.helmholtz-muenchen.de/icb/research/groups/machine-learning/projects/dpt/index.html
- **MATLAB** (Environment in which the MAPiT transformation and confidence-band plotting are executed) — https://github.com/karstenkuritz/MAPiT

## Examples

```
run MAPiT.m with pseudotime vectors from both Wanderlust and DPT via jointDensityPseudotimeY.m and preMAPiT.m, then overlay the transformed profiles with 50% confidence bands (see /examples/example_spheroid.m in github.com/karstenkuritz/MAPiT)
```

## Evaluation signals

- The 50% confidence intervals of the true-scale marker profiles (Ki-67, RNA, p27) from the Wanderlust and DPT orderings overlap, as reported for the spheroid data
- Both orderings are computed from the identical single-cell dataset (same n cells) so differences are attributable to the algorithm only
- Both transformed profiles are plotted on the same real scale (e.g. distance from spheroid surface) rather than on their native pseudotime axes
- Each ordering passes through the same MAPiT pipeline (jointDensityPseudotimeY.m → preMAPiT.m → MAPiT.m) with the same true-scale density prior

## Limitations

- MAPiT is reliant on the correctness of the cell order provided by the trajectory inference algorithm; agreement between two algorithms does not guarantee either ordering is biologically correct
- Constructing the target real-scale distribution for processes like stem cell differentiation is challenging; the paper only applied MAPiT where the target distribution could be derived theoretically
- The comparison can only be as good as the density estimation; kernel density estimation with linked boundary conditions is needed to obtain the distribution in pseudotime for the cell cycle example

## Evidence

- [results] Transforming the DPT and Wanderlust axes to a distance scale with MAPiT resulted in almost identical spatial profiles of Ki-67, RNA and p27 (with overlapping 50% confidence intervals): [evidence span withheld — anchor status: not_found]
- [intro] The method makes use of pseudotemporal ordering obtained from trajectory inference algorithms and hence is reliant on the correctness of the provided cell order.: "The method makes use of pseudotemporal ordering obtained from trajectory inference algorithms and hence is reliant on the correctness of the provided cell order."
- [readme] the examples use Wanderlust (part of the Cyt3 toolbox https://github.com/dpeerlab/cyt3): "the examples use Wanderlust (part of the Cyt3 toolbox https://github.com/dpeerlab/cyt3)"
- [readme] The function jointDensityPseudotimeY.m takes a nx1 vector of pseudotime values from the pseudotime algorithm: "The function [`jointDensityPseudotimeY.m`](jointDensityPseudotimeY.m) takes a `nx1` vector of pseudotime values from the pseudotime algorithm"
- [results] the distribution of cells on the spatial scale had to be taken into account, which in the case of radial symmetry of spheroids is the density of cells in relation to the distance from the spheroid: "the distribution of cells on the spatial scale had to be taken into account, which in the case of radial symmetry of spheroids is the density of cells in relation to the distance from the spheroid"
