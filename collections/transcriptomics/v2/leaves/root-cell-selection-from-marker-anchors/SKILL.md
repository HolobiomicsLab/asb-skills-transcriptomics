---
name: root-cell-selection-from-marker-anchors
description: Use when generating a pseudotemporal ordering from flow cytometry single-cell
  data before MAPiT transformation and a trajectory algorithm such as Wanderlust requires
  root cells to define the trajectory start.
when_to_use_negative:
- The trajectory algorithm you use (e.g. DPT/Diffusion Maps variant) does not require
  explicit root cells
- You have already produced a pseudotime ordering and only need the MAPiT transformation
  steps (preMAPiT.m, MAPiT.m)
- The cells are already ordered by a real measured time coordinate (e.g. time-lapse
  sampling timepoints), so pseudotime inference is unnecessary
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Wanderlust
  role: Trajectory-inference algorithm seeded with the selected root cells to derive
    pseudotime values from single-cell cytometry data
  repo: https://github.com/dpeerlab/cyt3
- name: Diffusion Maps / DPT
  role: Alternative pseudotime algorithm applied to the same gated single-cell data
  repo: https://www.helmholtz-muenchen.de/icb/research/groups/machine-learning/projects/dpt/index.html
- name: MATLAB
  role: Environment in which data are gated, subsampled to 10000 cells, and pseudotime
    values are exported as a nx1 CSV for MAPiT
  repo: https://github.com/karstenkuritz/MAPiT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-60400-z
    title: Reconstructing temporal and spatial dynamics from single-cell pseudotime
      using prior knowledge of real scale cell densities
  source_content_hash: sha256:76b674b26db41166ca51a223f6544b22a62cc1c32fb1dc68fdfabf938bbd7830
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dpeerlab/cyt3
  tools:
  - Diffusion Maps / DPT
  - MATLAB
  - Wanderlust
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# Root cell selection from marker anchors

## Summary

Selects anchor/root cells that seed trajectory-inference algorithms (Wanderlust, DPT) so the inferred pseudotime starts at a biologically defined stage, e.g. the DNA-based G1 centre for cell cycle data or high Ki-67 / high RNA cells for spheroid data.

## When to use

Use when generating a pseudotemporal ordering from flow cytometry single-cell data before MAPiT transformation and a trajectory algorithm such as Wanderlust requires root cells to define the trajectory start.

## When NOT to use

- The trajectory algorithm you use (e.g. DPT/Diffusion Maps variant) does not require explicit root cells
- You have already produced a pseudotime ordering and only need the MAPiT transformation steps (preMAPiT.m, MAPiT.m)
- The cells are already ordered by a real measured time coordinate (e.g. time-lapse sampling timepoints), so pseudotime inference is unnecessary

## Inputs

- Gated single-cell flow cytometry measurements (marker signals such as DNA, RNA, Ki-67, p27)
- Root-cell marker criteria (DNA G1 peak centre, or high Ki-67 / high RNA thresholds)
- Subsampled cell population (10000 cells)

## Outputs

- Root cell identifiers for trajectory seeding
- pseudotime_ordering.csv: nx1 vector (n=10000) of pseudotime values, one per sampled cell

## How to apply

After gating the flow cytometry measurements in MATLAB to remove doublets, dead cells and outliers, choose root cells from a marker-defined population that marks the earliest stage of the process: cells at the centre of the DNA-based G1 peak (2N DNA content) for the cell cycle example, or cells with high Ki-67 and high RNA signal for the spheroid example. Randomly subsample 10000 cells from the gated population to reduce the computation time of the trajectory algorithm. Pass the selected root cells to Wanderlust (part of the dpeerlab Cyt3 toolbox) or, alternatively, Diffusion Maps/DPT to derive a pseudotime value per cell. Export the resulting pseudotime values as a single-column (nx1) CSV, pseudotime_ordering.csv, matching the input format of MAPiT's jointDensityPseudotimeY.m.

## Related tools

- **Wanderlust** (Trajectory-inference algorithm seeded with the selected root cells to derive pseudotime values from single-cell cytometry data) — https://github.com/dpeerlab/cyt3
- **Diffusion Maps / DPT** (Alternative pseudotime algorithm applied to the same gated single-cell data) — https://www.helmholtz-muenchen.de/icb/research/groups/machine-learning/projects/dpt/index.html
- **MATLAB** (Environment in which data are gated, subsampled to 10000 cells, and pseudotime values are exported as a nx1 CSV for MAPiT) — https://github.com/karstenkuritz/MAPiT

## Examples

```
pseudotime = wanderlust(subsampled_cells, rootcell_idx); csvwrite('pseudotime_ordering.csv', pseudotime(:))
```

## Evaluation signals

- pseudotime_ordering.csv exists, is importable into MATLAB, and contains exactly one column
- The file contains one pseudotime value per sampled cell (n = 10000 rows), consistent with the nx1 vector input of jointDensityPseudotimeY.m
- Root cells fall in the intended marker-defined population: G1 peak centre by DNA content, or the high Ki-67 / high RNA gate
- Cells ranked earliest by the inferred pseudotime correspond to the selected root population (e.g. lowest pseudotime at G1/2N DNA content)

## Limitations

- MAPiT is reliant on the correctness of the cell order provided by the trajectory inference algorithm, so wrong root-cell selection propagates into the transformed real-scale result
- Random subsampling of 10000 cells trades computation time against coverage of rare subpopulations
- Choosing an anchor population requires prior marker knowledge (DNA peak, Ki-67, RNA); without such markers the root cannot be defined biologically

## Evidence

- [methods] Anchor selection by marker (workflow): [evidence span withheld — anchor status: not_found]
- [methods] Random subsampling: [evidence span withheld — anchor status: not_found]
- [methods] Wanderlust from Cyt3 toolbox: "the examples use Wanderlust (part of the Cyt3 toolbox https://github.com/dpeerlab/cyt3)"
- [results] Pseudotime ordering as MAPiT prerequisite: "We first used the Wanderlust and Diffusion Maps algorithms4,5 to obtain a pseudotemporal ordering."
- [intro] MAPiT dependence on ordering correctness: "The method makes use of pseudotemporal ordering obtained from trajectory inference algorithms and hence is reliant on the correctness of the provided cell order."
- [readme] jointDensityPseudotimeY input format: "The function [`jointDensityPseudotimeY.m`](jointDensityPseudotimeY.m) takes a `nx1` vector of pseudotime values from the pseudotime algorithm"
