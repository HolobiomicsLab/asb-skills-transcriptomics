---
name: time-lapse-microscopy-trajectory-comparison
description: Use when a MAPiT-transformed pseudotime trajectory must be validated
  against ground-truth real-time kinetics, e.g. geminin trajectories reconstructed
  from static flow cytometry that should match single-cell time-lapse microscopy data
  (Paper Fig 2).
when_to_use_negative:
- The trajectory is a pseudotime ordering that has not yet been transformed to the
  real-time scale with MAPiT.m and preMAPiT.m — comparison would be meaningless on
  the pseudotime axis.
- The process has no independent real-time reference measurement (e.g. no time-lapse
  microscopy data and no labelled sampling timepoints); the paper notes such validation
  was only envisioned for label data.
- The analysis target is the spheroid spatial scale, where validation instead requires
  sliced spheroid cross-sections quantified by confocal microscopy (MAPiT_Spheroid_Validation.m),
  not time-lapse kinetics.
tools:
- name: MAPiT
  role: Matlab scripts providing MAPiT.m for the real-time transformation and MAPiT_CellCycle_Validation.m
    for the MAPiT-vs-microscopy comparison (Paper Fig 2)
  repo: https://github.com/karstenkuritz/MAPiT
- name: Wanderlust (Cyt3 toolbox)
  role: Generates the pseudotemporal ordering of cells upstream of the MAPiT transformation
    that is subsequently validated against microscopy
  repo: https://github.com/dpeerlab/cyt3
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-60400-z
    title: Reconstructing temporal and spatial dynamics from single-cell pseudotime
      using prior knowledge of real scale cell densities
  source_content_hash: sha256:7e1ae3c0f515c21defe540d073a469d29413762c43c9ef4c59374bc2af2ef74f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/karstenkuritz/MAPiT
  tools:
  - MAPiT
  - Wanderlust (Cyt3 toolbox)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Time-lapse microscopy trajectory comparison

## Summary

Validation strategy in which a MAPiT-reconstructed marker trajectory (e.g. geminin kinetics from static flow cytometry of unsynchronised NCI-H460/geminin cells) is compared against independently measured single-cell time-lapse microscopy kinetics to confirm the recovered real-time scale.

## When to use

Use when a MAPiT-transformed pseudotime trajectory must be validated against ground-truth real-time kinetics, e.g. geminin trajectories reconstructed from static flow cytometry that should match single-cell time-lapse microscopy data (Paper Fig 2).

## When NOT to use

- The trajectory is a pseudotime ordering that has not yet been transformed to the real-time scale with MAPiT.m and preMAPiT.m — comparison would be meaningless on the pseudotime axis.
- The process has no independent real-time reference measurement (e.g. no time-lapse microscopy data and no labelled sampling timepoints); the paper notes such validation was only envisioned for label data.
- The analysis target is the spheroid spatial scale, where validation instead requires sliced spheroid cross-sections quantified by confocal microscopy (MAPiT_Spheroid_Validation.m), not time-lapse kinetics.

## Inputs

- MAPiT-transformed geminin trajectory on the real time scale (e.g. geminin_MAPiT_trajectory.csv)
- Single-cell time-lapse microscopy geminin kinetics of NCI-H460/geminin cells (ground-truth reference)

## Outputs

- Overlay comparison of MAPiT vs. microscopy geminin trajectories (Paper Fig 2 reproduction)
- Pass/fail validation verdict based on correspondence of the two kinetic curves, including reproduction of the ~7 h S-phase onset boost

## How to apply

After transforming pseudotime to real time with MAPiT.m, export the reconstructed geminin trajectory (e.g. as geminin_MAPiT_trajectory.csv) and run the dedicated validation script MAPiT_CellCycle_Validation.m, which reproduces Paper Fig 2 by overlaying the MAPiT trajectory on the geminin kinetics obtained by single-cell time-lapse microscopy. Accept the reconstruction when the two trajectories correspond excellently across the cell cycle. A key qualitative check is that the MAPiT trajectory reproduces the boost in geminin intensity at approximately 7 h that indicates onset of S-phase. Note that in the imaging experiment geminin intensity exceeded the detection range so that G1-phase cells had signals below the detection limit, which must be tolerated when aligning the curves.

## Related tools

- **MAPiT** (Matlab scripts providing MAPiT.m for the real-time transformation and MAPiT_CellCycle_Validation.m for the MAPiT-vs-microscopy comparison (Paper Fig 2)) — https://github.com/karstenkuritz/MAPiT
- **Wanderlust (Cyt3 toolbox)** (Generates the pseudotemporal ordering of cells upstream of the MAPiT transformation that is subsequently validated against microscopy) — https://github.com/dpeerlab/cyt3

## Examples

```
run('examples/MAPiT_CellCycle_Validation.m')
```

## Evaluation signals

- The MAPiT-transformed geminin trajectory corresponds excellently to the single-cell time-lapse microscopy geminin kinetics when overlaid (Paper Fig 2).
- The reconstructed trajectory reproduces the boost in geminin intensity at approximately 7 h marking onset of S-phase.
- Running MAPiT_CellCycle_Validation.m regenerates the Paper Fig 2 comparison without errors.
- Agreement holds for the cell order derived from either pseudotime algorithm (Wanderlust or Diffusion Maps/DPT), indicating robustness of the validated scale.

## Limitations

- Geminin intensity in the imaging experiment spans several orders of magnitude and exceeds the detection range, so G1-phase cells have signals below the detection limit and cannot be compared directly.
- MAPiT validation relies on the correctness of the cell order provided by the trajectory inference algorithm (Wanderlust/DPT).
- No labelled/sampling-timepoint data were used in the paper to support or validate MAPiT results; the authors only envision such validation as an alternative.

## Evidence

- [results] MAPiT trajectories correspond excellently to microscopy kinetics: "Temporal trajectories of geminin obtained with MAPiT corresponded excellently to geminin kinetics obtained by single cell time-lapse microscopy"
- [results] S-phase boost at ~7 h is the acceptance signal: [evidence span withheld — anchor status: not_found]
- [results] Detection-range limitation in the imaging reference data: "The geminin intensity, spanning several orders of magnitude, exceeds the detection range in the imaging experiment such that G1 phase cells had signals below the detection limit."
- [readme] Dedicated validation script for the cell-cycle example: "- Paper Fig 2: Cell cycle - MAPiT vs. microscopy"
- [readme] Pseudotime tooling used upstream of the validated transformation: "the examples use Wanderlust (part of the Cyt3 toolbox https://github.com/dpeerlab/cyt3) and Diffusion Maps"
