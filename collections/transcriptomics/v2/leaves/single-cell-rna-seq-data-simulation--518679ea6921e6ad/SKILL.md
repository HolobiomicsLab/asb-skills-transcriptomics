---
name: single-cell-rna-seq-data-simulation
description: Use when evaluating a cluster-purity metric (e.g., ROGUE) against sequencing-depth
  differences or cell-number variation, by generating simulated replicate scRNA-seq
  mixtures with known ground-truth purity.
when_to_use_negative:
- Input consists of real, already-annotated cell clusters rather than simulated populations
  with known ground-truth purity — use ROGUE directly via CalculateRogue instead of
  simulating
- The goal is to assess purity of integrated cell populations from different protocols/platforms,
  which the paper states ROGUE does not yet support (proposed future work)
- The task is clustering or rare cell-type detection (e.g., GiniClust, RaceID3) rather
  than benchmarking a purity metric
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ROGUE
  role: Entropy-based statistic (Ratio of Global Unshifted Entropy) used to quantify
    purity of simulated replicate populations
  repo: https://github.com/PaulingLiu/ROGUE
- name: ROGUE (R package)
  role: R implementation providing matr.filter, entropy_fit, DetermineK, and CalculateRogue
    functions applied to each simulated matrix
  repo: https://github.com/PaulingLiu/ROGUE
- name: tidyverse
  role: Dependency package that must be installed before installing ROGUE; used for
    data manipulation in the simulation workflow
- name: devtools
  role: Used to install the ROGUE R package from GitHub
- name: R
  role: Language in which the simulation, ROGUE computation, and silhouette evaluation
    are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-16904-3
    title: An entropy-based metric for assessing the purity of single cell populations
  source_content_hash: sha256:905407a82676919686295af204946a1f91c37609ecf647cc62230a0617b01c87
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/ROGUE
  tools:
  - R
  - ROGUE
  - ROGUE (R package)
  - devtools
  - ggplot2
  - tidyverse
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- single-cell-rna-seq-dataset-simulation
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# single-cell-rna-seq-data-simulation

## Summary

Simulate two-replicate pure single-cell populations with controlled sequencing-depth fold changes (2–100x) and varying cell numbers to benchmark whether the ROGUE entropy-based purity metric remains robust to technical factors.

## When to use

Use when evaluating a cluster-purity metric (e.g., ROGUE) against sequencing-depth differences or cell-number variation, by generating simulated replicate scRNA-seq mixtures with known ground-truth purity.

## When NOT to use

- Input consists of real, already-annotated cell clusters rather than simulated populations with known ground-truth purity — use ROGUE directly via CalculateRogue instead of simulating
- The goal is to assess purity of integrated cell populations from different protocols/platforms, which the paper states ROGUE does not yet support (proposed future work)
- The task is clustering or rare cell-type detection (e.g., GiniClust, RaceID3) rather than benchmarking a purity metric

## Inputs

- Simulated two-replicate scRNA-seq gene-by-cell count matrices with 2- to 100-fold sequencing-depth differences
- Down-sampled versions of simulated populations at varying cell numbers
- Reproduction scripts from the PaulingLiu/ROGUE repository scripts folder

## Outputs

- ROGUE purity values per simulation replicate (CalculateRogue output)
- Entropy-fit parameter S values per replicate and per cell-number level
- Silhouette coefficients paired with ROGUE values
- Consolidated results table written to rogue_simulation_results.csv

## How to apply

Install tidyverse and the ROGUE R package via devtools::install_github("PaulingLiu/ROGUE"), then follow the reproduction scripts in the repository's scripts folder. Simulate two-replicate pure cell populations while varying sequencing-depth fold change from 2 to 100; for each simulation, run matr.filter on the expression matrix, estimate the entropy-fit parameter S with entropy_fit/DetermineK, and compute purity with CalculateRogue. In parallel, compute silhouette coefficients on the same simulated data to provide a comparison metric. Separately, down-sample the number of cells in steps and recompute S at each level, then evaluate Pearson correlations of S across down-sampling levels. The rationale is that a robust purity metric should stay high (~0.99–1) on known-pure simulated populations despite depth and cell-number perturbations, whereas generic metrics like silhouette vary widely.

## Related tools

- **ROGUE** (Entropy-based statistic (Ratio of Global Unshifted Entropy) used to quantify purity of simulated replicate populations) — https://github.com/PaulingLiu/ROGUE
- **ROGUE (R package)** (R implementation providing matr.filter, entropy_fit, DetermineK, and CalculateRogue functions applied to each simulated matrix) — https://github.com/PaulingLiu/ROGUE
- **tidyverse** (Dependency package that must be installed before installing ROGUE; used for data manipulation in the simulation workflow)
- **devtools** (Used to install the ROGUE R package from GitHub)
- **R** (Language in which the simulation, ROGUE computation, and silhouette evaluation are executed)

## Examples

```
install.packages("tidyverse"); devtools::install_github("PaulingLiu/ROGUE"); rogue <- CalculateRogue(matr.filter(sim_matrix, min.cells, min.genes), platform="UMI")
```

## Evaluation signals

- ROGUE values fall in the ~0.99–1 range across the full 2–100 fold sequencing-depth-change range, confirming detected purity on known-pure simulations
- Silhouette coefficients on the same simulated data span 0.25–0.75, demonstrating that ROGUE's stability is not trivially shared by generic clustering metrics
- Pearson correlations of the entropy-fit parameter S across cell down-sampling levels exceed 0.99
- All simulation replicates complete the matr.filter → DetermineK → CalculateRogue pipeline without errors
- The assembled table of ROGUE values, silhouette values, and S-correlation results is written to rogue_simulation_results.csv and matches the manuscript's quantitative results

## Limitations

- The 0.9 purity threshold may not be suitable for low-quality or continuous data, where it must be determined from global ROGUE values
- Current ROGUE does not assess purity of integrated cell populations from different protocols and platforms; this is proposed as future work
- Silhouette values on the same simulated pure populations range widely (0.25–0.75), so silhouette alone cannot be trusted as a depth-robust purity proxy

## Evidence

- [results] ROGUE robustness simulation finding: [evidence span withheld — anchor status: not_found]
- [methods] Package installation from methods: "devtools::install_github("PaulingLiu/ROGUE")"
- [methods] Filter step from methods: "matr.filter(expr, min.cells, min.genes)"
- [methods] ROGUE computation from methods: "CalculateRogue(.x, platform, cutoff, k, features)"
- [readme] Reproduction scripts from readme: "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)."
- [discussion] Threshold caveat from discussion: "But for low-quality data or continuous data, the threshold could be determined by considering the global ROGUE values"
