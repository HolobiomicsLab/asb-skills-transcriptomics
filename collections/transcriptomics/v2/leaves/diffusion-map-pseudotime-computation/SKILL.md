---
name: diffusion-map-pseudotime-computation
description: Use when static single-cell flow cytometry data (e.g. DNA, RNA, Ki-67,
  p27 stainings) must be ordered along a developmental or spatial trajectory to serve
  as input to pseudotime-to-real-time transformation.
when_to_use_negative:
- Data are time-series with real sampling timepoints — real temporal labels make pseudotime
  inference unnecessary
- You cannot define or measure the true-scale distribution needed downstream — the
  ordering alone does not recover real-time dynamics
- The cell population is not along a coherent trajectory (e.g. unrelated clusters),
  where trajectory inference assumptions fail
tools:
- name: Wanderlust (Cyt3 toolbox)
  role: Trajectory-inference algorithm used to derive pseudotime values from single-cell
    data, invoked within MATLAB
  repo: https://github.com/dpeerlab/cyt3
- name: Diffusion Maps / DPT
  role: Alternative pseudotime algorithm; comparison of Wanderlust vs DPT orderings
    is used as a validation step
  repo: https://www.helmholtz-muenchen.de/icb/research/groups/machine-learning/projects/dpt/index.html
- name: MATLAB
  role: Environment in which the examples run Wanderlust and produce pseudotime vectors
    for jointDensityPseudotimeY.m
  repo: https://github.com/karstenkuritz/MAPiT
- name: Kernel Density Estimation with Linked BCs
  role: Used after ordering to obtain the density distribution in pseudotime for the
    cell cycle example
  repo: https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-60400-z
    title: Reconstructing temporal and spatial dynamics from single-cell pseudotime
      using prior knowledge of real scale cell densities
  source_content_hash: sha256:bb2d78ad50df071605187159610a127518ead951107d6bacef1b69df751c7286
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dpeerlab/cyt3
  tools:
  - Diffusion Maps
  - Diffusion Maps / DPT
  - Diffusion Pseudotime (DPT)
  - Kernel Density Estimation with Linked BCs
  - MATLAB
  - Wanderlust (Cyt3 toolbox)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# diffusion-map-pseudotime-computation

## Summary

Computes a pseudotemporal ordering of single-cell flow cytometry measurements using trajectory-inference algorithms (Diffusion Maps/DPT or Wanderlust), producing the nx1 pseudotime vector that downstream methods such as MAPiT require.

## When to use

Use when static single-cell flow cytometry data (e.g. DNA, RNA, Ki-67, p27 stainings) must be ordered along a developmental or spatial trajectory to serve as input to pseudotime-to-real-time transformation.

## When NOT to use

- Data are time-series with real sampling timepoints — real temporal labels make pseudotime inference unnecessary
- You cannot define or measure the true-scale distribution needed downstream — the ordering alone does not recover real-time dynamics
- The cell population is not along a coherent trajectory (e.g. unrelated clusters), where trajectory inference assumptions fail

## Inputs

- Gated single-cell flow cytometry measurement matrix (n cells x markers, e.g. DNA, RNA, Ki-67, p27, forward scatter)
- Root/anchor cell identities (e.g. G1-centre cells by DNA content, or high Ki-67/high RNA cells)
- Random subsample of ~10000 gated cells

## Outputs

- nx1 pseudotime vector, one value per sampled cell (n = 10000)
- pseudotime_ordering.csv (single-column nx1 CSV) importable into MATLAB

## How to apply

Load preprocessed flow cytometry measurements into MATLAB and gate out doublets, dead cells and outlier events. Select anchor root cells appropriate to the application (e.g. cells in the DNA-based G1 peak for cell cycle, or high Ki-67/high RNA cells for spheroids), then randomly subsample ~10000 cells to keep trajectory computation tractable. Run Wanderlust from the dpeerlab Cyt3 toolbox, or Diffusion Maps/DPT (Helmholtz ICB implementation), to assign a pseudotime value to each of the 10000 cells. Export the pseudotime values as a single-column (nx1) CSV, e.g. pseudotime_ordering.csv, matching the input format of MAPiT's jointDensityPseudotimeY.m. Note that MAPiT's transformation is measure-preserving but entirely reliant on the correctness of this ordering, so cross-validate using marker subsets (leave-one-out) and compare Wanderlust vs DPT outputs.

## Related tools

- **Wanderlust (Cyt3 toolbox)** (Trajectory-inference algorithm used to derive pseudotime values from single-cell data, invoked within MATLAB) — https://github.com/dpeerlab/cyt3
- **Diffusion Maps / DPT** (Alternative pseudotime algorithm; comparison of Wanderlust vs DPT orderings is used as a validation step) — https://www.helmholtz-muenchen.de/icb/research/groups/machine-learning/projects/dpt/index.html
- **MATLAB** (Environment in which the examples run Wanderlust and produce pseudotime vectors for jointDensityPseudotimeY.m) — https://github.com/karstenkuritz/MAPiT
- **Kernel Density Estimation with Linked BCs** (Used after ordering to obtain the density distribution in pseudotime for the cell cycle example) — https://github.com/MColbrook/Kernel-Density-Estimation-with-Linked-BCs.git

## Examples

```
pseudotime = csvread('pseudotime_ordering.csv'); jointDensityPseudotimeY(pseudotime, marker)
```

## Evaluation signals

- pseudotime_ordering.csv exists, imports into MATLAB, and contains exactly one column with one pseudotime value per cell (n = 10000 rows, nx1 format)
- Leave-one-out cross validation with marker subsets yields consistent orderings from Wanderlust and DPT
- Positions of known cell-cycle states in pseudotime are consistent with DNA content (G1 2N vs S/G2-M 4N bimodality)
- Ordering is comparable between the Wanderlust and DPT algorithms on the same subsampled cells

## Limitations

- Pseudotime trajectories may deviate substantially from the real-time trajectory, and the downstream transformation (MAPiT) is reliant on the correctness of the provided cell order
- Only ~10000 subsampled cells are used, so ordering accuracy depends on the subsample; trajectories are validated only for the two application cases (cell cycle, spheroids) where the target distribution could be derived theoretically
- Sources and sinks (cell death or division) along the process are not known in the presented examples but are critical for a correct real-time transformation

## Evidence

- [results] Wanderlust and Diffusion Maps derive pseudotime: "We first used the Wanderlust and Diffusion Maps algorithms4,5 to obtain a pseudotemporal ordering."
- [readme] Cyt3 toolbox and Diffusion Maps usage: "the examples use Wanderlust (part of the Cyt3 toolbox https://github.com/dpeerlab/cyt3) and Diffusion Maps"
- [methods] nx1 vector input format: "The function jointDensityPseudotimeY.m takes a nx1 vector of pseudotime values from the pseudotime algorithm"
- [intro] Reliance on ordering correctness: "The method makes use of pseudotemporal ordering obtained from trajectory inference algorithms and hence is reliant on the correctness of the provided cell order."
- [results] Leave-one-out cross validation: "we conducted "leave one out cross validation", where we took subsets of the markers as input to the Wanderlust and DPT algorithms"
