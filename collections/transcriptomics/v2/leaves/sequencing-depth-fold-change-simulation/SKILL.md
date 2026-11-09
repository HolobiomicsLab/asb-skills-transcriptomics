---
name: sequencing-depth-fold-change-simulation
description: Use when validating that a single-cell cluster-purity metric (e.g., ROGUE)
  is insensitive to sequencing-depth fold changes between replicates or to variation
  in cell number, by simulating replicate mixtures and comparing ROGUE/S against silhouette.
when_to_use_negative:
- The goal is to compare clustering algorithms (e.g., GiniClust, GiniClust2, RaceID3)
  rather than to validate the robustness of a purity metric to technical depth differences
- The data are integrated cell populations from different protocols and platforms
  — the article explicitly notes ROGUE does not yet assess purity of such integrated
  populations (proposed future work)
- The input is already a real experimental dataset where the 0.9 purity threshold
  or a data-derived global ROGUE threshold should be applied directly instead of running
  simulations
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ROGUE (R package)
  role: Provides matr.filter, entropy_fit, DetermineK, and CalculateRogue used to
    filter matrices, estimate S, and compute purity on the simulated replicate mixtures
  repo: https://github.com/PaulingLiu/ROGUE
- name: tidyverse
  role: Dependency package installed before ROGUE for data manipulation in the simulation
    scripts
  repo: https://github.com/PaulingLiu/ROGUE
- name: devtools
  role: Used to install ROGUE from GitHub via devtools::install_github
  repo: https://github.com/PaulingLiu/ROGUE
- name: R
  role: Language environment in which the simulation, down-sampling, and ROGUE/silhouette
    computations are run
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-16904-3
    title: An entropy-based metric for assessing the purity of single cell populations
  source_content_hash: sha256:7d2a09b2b518f8589aadcade47f258723ac941f7f35aaefafa6f90f50d170fb2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/ROGUE
  tools:
  - R
  - ROGUE (R package)
  - devtools
  - tidyverse
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# sequencing-depth-fold-change-simulation

## Summary

Simulate two-replicate pure single-cell populations with 2- to 100-fold sequencing-depth differences and cell-number down-sampling, then verify with the ROGUE entropy statistic that the purity metric stays robust (~0.99-1) while a conventional metric like silhouette varies (0.25-0.75).

## When to use

Use when validating that a single-cell cluster-purity metric (e.g., ROGUE) is insensitive to sequencing-depth fold changes between replicates or to variation in cell number, by simulating replicate mixtures and comparing ROGUE/S against silhouette.

## When NOT to use

- The goal is to compare clustering algorithms (e.g., GiniClust, GiniClust2, RaceID3) rather than to validate the robustness of a purity metric to technical depth differences
- The data are integrated cell populations from different protocols and platforms — the article explicitly notes ROGUE does not yet assess purity of such integrated populations (proposed future work)
- The input is already a real experimental dataset where the 0.9 purity threshold or a data-derived global ROGUE threshold should be applied directly instead of running simulations

## Inputs

- Simulated gene-by-cell expression matrices for two-replicate pure cell populations
- Sequencing-depth fold-change levels ranging from 2 to 100
- Cell numbers per replicate for down-sampling series
- Reproduction scripts from the PaulingLiu/ROGUE repository scripts folder

## Outputs

- ROGUE purity values per simulated replicate (expected ~0.99-1 across 2-100 fold-depth changes)
- Silhouette coefficients per simulation (expected range 0.25-0.75)
- Pearson correlations of entropy-fit parameter S across cell down-sampling levels (>0.99)
- Consolidated results table written to rogue_simulation_results.csv

## How to apply

Install tidyverse and the ROGUE R package from GitHub, then follow the manuscript reproduction scripts in the repository's scripts folder. Simulate two-replicate pure cell populations while varying the sequencing-depth fold change from 2 to 100; for each simulation run matr.filter on the expression matrix, estimate the entropy-fit parameter S via entropy_fit/DetermineK, and compute purity with CalculateRogue. In parallel compute silhouette coefficients on the same simulated matrices and pair them with ROGUE values to demonstrate that ROGUE remains ~0.99-1 while silhouette spans 0.25-0.75. Separately, down-sample the number of cells in steps, recompute S at each cell count, and evaluate Pearson correlations of S across down-sampling levels, expecting values >0.99. Assemble ROGUE values, silhouette values, and S-correlation results into a single table (rogue_simulation_results.csv) and check that they reproduce the reported quantitative behavior, confirming the metric accounts for technical factors.

## Related tools

- **ROGUE (R package)** (Provides matr.filter, entropy_fit, DetermineK, and CalculateRogue used to filter matrices, estimate S, and compute purity on the simulated replicate mixtures) — https://github.com/PaulingLiu/ROGUE
- **tidyverse** (Dependency package installed before ROGUE for data manipulation in the simulation scripts) — https://github.com/PaulingLiu/ROGUE
- **devtools** (Used to install ROGUE from GitHub via devtools::install_github) — https://github.com/PaulingLiu/ROGUE
- **R** (Language environment in which the simulation, down-sampling, and ROGUE/silhouette computations are run)

## Examples

```
if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools"); devtools::install_github("PaulingLiu/ROGUE")
```

## Evaluation signals

- ROGUE values fall in ~0.99-1 across the entire 2- to 100-fold sequencing-depth-change range, indicating inferred purity
- Silhouette values on the same simulated data span roughly 0.25-0.75, contrasting with the stable ROGUE result
- Pearson correlations of the entropy-fit parameter S under cell down-sampling exceed 0.99
- S correlations exceed 0.99 across down-sampled cell numbers, matching the manuscript's reported invariance
- Combined ROGUE, silhouette, and S-correlation results reproduce the manuscript's quantitative table and are written successfully to rogue_simulation_results.csv

## Limitations

- Robustness is demonstrated for two-replicate simulated pure populations; behavior for more complex or heterogeneous simulated mixtures is not established by this workflow
- The recommended 0.9 purity threshold may not suit low-quality or continuous data; the threshold must be determined from global ROGUE values in those cases
- ROGUE does not yet assess purity of integrated cell populations from different protocols and platforms (noted as future work)

## Evidence

- [intro] we propose the Ratio of Global Unshifted Entropy (ROGUE) statistic to quantify the purity or homogeneity of a given single-cell population while accounting for other technical factors: "we propose the Ratio of Global Unshifted Entropy (ROGUE) statistic to quantify the purity or homogeneity of a given single-cell population while accounting for other technical factors"
- [methods] The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts): "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)"
- [readme] Before installing ROGUE, the “tidyverse” package should be installed first: "Before installing ROGUE, the “tidyverse” package should be installed first"
- [methods] CalculateRogue(.x, platform, cutoff, k, features): "CalculateRogue(.x, platform, cutoff, k, features)"
- [abstract] demonstrate the application of ROGUE-guided analyses to detect precise signals in specific subpopulations: "demonstrate the application of ROGUE-guided analyses to detect precise signals in specific subpopulations"
