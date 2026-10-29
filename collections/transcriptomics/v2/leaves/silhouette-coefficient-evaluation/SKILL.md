---
name: silhouette-coefficient-evaluation
description: Use when benchmarking a cluster-purity or clustering-quality metric (e.g.,
  ROGUE) against silhouette coefficients on simulated or real scRNA-seq populations,
  especially to test robustness to sequencing-depth or cell-number differences.
when_to_use_negative:
- Input is already a filtered/normalized feature table or the goal is simply to pick
  the optimal number of clusters — the article notes silhouette and similar approaches
  'are not comparable among datasets and have poor interpretability of cluster purity'
- The aim is to detect rare cell types rather than assess cluster purity — dedicated
  tools like GiniClust or RaceID3 are designed for that task
- Data are low-quality or continuous where the recommended 0.9 ROGUE purity threshold
  may not apply and thresholds must be set from global ROGUE values
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ROGUE
  role: entropy-based purity statistic whose values are paired with silhouette coefficients
    to benchmark purity-metric behavior
  repo: https://github.com/PaulingLiu/ROGUE
- name: ROGUE (R package)
  role: provides matr.filter, entropy_fit/DetermineK, and CalculateRogue used alongside
    silhouette computation in R
  repo: https://github.com/PaulingLiu/ROGUE
- name: tidyverse
  role: dependency package installed before ROGUE for data manipulation in the simulation
    scripts
- name: devtools
  role: used to install ROGUE from GitHub via devtools::install_github
- name: R
  role: language in which the simulation, ROGUE computation, and silhouette evaluation
    scripts are run
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-16904-3
    title: An entropy-based metric for assessing the purity of single cell populations
  source_content_hash: sha256:f109241945aaa3b50caa9e9e312079d95bff72e297699a9f67365f08d81ec4f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/ROGUE
  tools:
  - R
  - ROGUE
  - ROGUE (R package)
  - devtools
  - tidyverse
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# silhouette-coefficient-evaluation

## Summary

Evaluates single-cell cluster separation by computing silhouette coefficients on gene-expression matrices and, as in the ROGUE paper (Nat Commun 2020), pairing silhouette values with the entropy-based ROGUE metric to benchmark how cluster-purity statistics behave on simulated replicate mixtures.

## When to use

Use when benchmarking a cluster-purity or clustering-quality metric (e.g., ROGUE) against silhouette coefficients on simulated or real scRNA-seq populations, especially to test robustness to sequencing-depth or cell-number differences.

## When NOT to use

- Input is already a filtered/normalized feature table or the goal is simply to pick the optimal number of clusters — the article notes silhouette and similar approaches 'are not comparable among datasets and have poor interpretability of cluster purity'
- The aim is to detect rare cell types rather than assess cluster purity — dedicated tools like GiniClust or RaceID3 are designed for that task
- Data are low-quality or continuous where the recommended 0.9 ROGUE purity threshold may not apply and thresholds must be set from global ROGUE values

## Inputs

- gene-by-cell UMI count matrix from simulated two-replicate scRNA-seq populations with controlled sequencing-depth fold changes (2- to 100-fold)
- simulated pure cell populations with varying cell numbers for down-sampling tests
- cluster assignments for the cells under evaluation

## Outputs

- silhouette coefficient values (expected range ~0.25-0.75 on depth-varied pure populations)
- ROGUE values from CalculateRogue (expected ~0.99-1 for pure populations)
- Pearson correlations of entropy-fit parameter S across cell down-sampling levels (>0.99)
- combined results table written to rogue_simulation_results.csv

## How to apply

First install the dependency tidyverse and the ROGUE R package via devtools::install_github("PaulingLiu/ROGUE"), then run matr.filter on the simulated gene-by-cell count matrix and estimate the entropy-fit parameter S with entropy_fit/DetermineK before calling CalculateRogue. In parallel, compute silhouette coefficients on the same simulated two-replicate populations — vary the sequencing-depth fold change from 2 to 100 between replicates — so silhouette values span roughly 0.25–0.75 while ROGUE stays near 0.99–1 for pure populations. The rationale is that silhouette measures geometric cluster separation, whereas ROGUE (Ratio of Global Unshifted Entropy) captures expression randomness while accounting for technical factors, so the pair reveals whether a purity statistic is depth-sensitive. Down-sample the number of cells in steps, recompute S at each cell number, and check Pearson correlations of S exceed 0.99 to confirm cell-number robustness. Assemble ROGUE values, silhouette values, and S-correlation results into a single table (e.g., rogue_simulation_results.csv) for validation against the manuscript's reported ranges.

## Related tools

- **ROGUE** (entropy-based purity statistic whose values are paired with silhouette coefficients to benchmark purity-metric behavior) — https://github.com/PaulingLiu/ROGUE
- **ROGUE (R package)** (provides matr.filter, entropy_fit/DetermineK, and CalculateRogue used alongside silhouette computation in R) — https://github.com/PaulingLiu/ROGUE
- **tidyverse** (dependency package installed before ROGUE for data manipulation in the simulation scripts)
- **devtools** (used to install ROGUE from GitHub via devtools::install_github)
- **R** (language in which the simulation, ROGUE computation, and silhouette evaluation scripts are run)

## Examples

```
install.packages("tidyverse"); if (!requireNamespace("devtools", quietly=TRUE)) install.packages("devtools"); devtools::install_github("PaulingLiu/ROGUE")
```

## Evaluation signals

- ROGUE values fall in ~0.99-1 across the 2-100 fold sequencing-depth change range, confirming purity is detected despite depth differences
- Silhouette values on the same simulations span ~0.25-0.75, demonstrating the geometric metric varies while ROGUE stays constant
- Pearson correlations of the entropy-fit parameter S under cell down-sampling exceed 0.99
- Results table (e.g., rogue_simulation_results.csv) contains one row per simulation replicate with paired ROGUE and silhouette values
- All simulation scripts run from the repository's ./scripts folder, which reproduces the manuscript's quantitative results

## Limitations

- The recommended 0.9 purity threshold may not suit low-quality or continuous data; the threshold should then be determined from global ROGUE values
- ROGUE does not yet assess purity of integrated cell populations generated from different protocols and platforms
- Silhouette values are not comparable among datasets and have poor interpretability of cluster purity, which motivates pairing them with an entropy-based statistic

## Evidence

- [abstract] abstract_rogue_scope: "We demonstrate that our ROGUE metric is broadly applicable, and enables accurate, sensitive and robust assessment of cluster purity on a wide range of simulated and real datasets."
- [intro] intro_silhouette_limits: [evidence span withheld — anchor status: not_found]
- [methods] methods_rogue_definition: "introduced a conceptually novel statistic, named ROGUE, to examine whether a given cluster is a pure cell population"
- [readme] readme_scripts: "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)."
