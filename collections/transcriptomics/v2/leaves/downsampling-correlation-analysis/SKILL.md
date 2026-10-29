---
name: downsampling-correlation-analysis
description: Use when validating that a cluster-purity metric or entropy-fit parameter
  (e.g. S in ROGUE) remains stable under varying cell numbers or sequencing-depth
  differences (2- to 100-fold) between replicates.
when_to_use_negative:
- Input is real (non-simulated) integrated data from different protocols/platforms
  — the article notes ROGUE does not yet assess purity of such integrated populations
- Low-quality or continuous data where the recommended 0.9 purity threshold must instead
  be determined from global ROGUE values rather than the default validation criteria
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ROGUE
  role: Computes the cluster-purity statistic and the entropy-fit parameter S used
    in the down-sampling correlation analysis
  repo: https://github.com/PaulingLiu/ROGUE
- name: ROGUE (R package)
  role: Provides matr.filter, entropy_fit, DetermineK, and CalculateRogue functions
    invoked on each simulated/down-sampled matrix
  repo: https://github.com/PaulingLiu/ROGUE
- name: R
  role: Runtime environment for running the reproduction scripts and simulation workflow
- name: tidyverse
  role: Dependency package installed before ROGUE
- name: devtools
  role: Used to install ROGUE from GitHub via devtools::install_github
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-16904-3
    title: An entropy-based metric for assessing the purity of single cell populations
  source_content_hash: sha256:c7b44864e937b35ee88ab6415ced90605466a77e582532daa946d2f1d4dcaadb
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# downsampling-correlation-analysis

## Summary

A robustness-testing procedure that down-samples the number of cells (or varies sequencing depth) in single-cell populations, recomputes the ROGUE entropy-fit parameter S at each level, and quantifies Pearson correlations of S across down-sampling levels to demonstrate metric stability.

## When to use

Use when validating that a cluster-purity metric or entropy-fit parameter (e.g. S in ROGUE) remains stable under varying cell numbers or sequencing-depth differences (2- to 100-fold) between replicates.

## When NOT to use

- Input is real (non-simulated) integrated data from different protocols/platforms — the article notes ROGUE does not yet assess purity of such integrated populations
- Low-quality or continuous data where the recommended 0.9 purity threshold must instead be determined from global ROGUE values rather than the default validation criteria

## Inputs

- Gene-by-cell expression count matrices for simulated two-replicate single-cell populations
- Simulated sequencing-depth fold changes ranging from 2 to 100 between replicates
- Cell numbers at successive down-sampling levels

## Outputs

- ROGUE values per simulation replicate (expected ~0.99–1)
- Silhouette coefficient values paired with ROGUE values (expected range 0.25–0.75)
- Pearson correlations of S across cell down-sampling levels (expected >0.99)
- A combined results table written to rogue_simulation_results.csv

## How to apply

Simulate two-replicate pure cell populations following the reproduction scripts in the ROGUE repository's scripts folder, varying sequencing-depth fold changes from 2 to 100. For each simulated matrix, run matr.filter(expr, min.cells, min.genes), apply entropy_fit/DetermineK to estimate the entropy-fit parameter S, and compute ROGUE via CalculateRogue. In parallel, compute silhouette coefficients on the same data and pair them with ROGUE values to contrast metric behavior. Then down-sample the number of cells in steps, recompute S at each cell number, and evaluate Pearson correlations of S across down-sampling levels. Rationale: ROGUE should confer purity (values ~0.99–1) even when silhouette spans 0.25–0.75, and S correlations under cell down-sampling should exceed 0.99, matching the manuscript's reported quantitative results.

## Related tools

- **ROGUE** (Computes the cluster-purity statistic and the entropy-fit parameter S used in the down-sampling correlation analysis) — https://github.com/PaulingLiu/ROGUE
- **ROGUE (R package)** (Provides matr.filter, entropy_fit, DetermineK, and CalculateRogue functions invoked on each simulated/down-sampled matrix) — https://github.com/PaulingLiu/ROGUE
- **R** (Runtime environment for running the reproduction scripts and simulation workflow)
- **tidyverse** (Dependency package installed before ROGUE)
- **devtools** (Used to install ROGUE from GitHub via devtools::install_github)

## Examples

```
Rscript -e 'devtools::install_github("PaulingLiu/ROGUE"); S <- DetermineK(expr, span, r, mt.method, if.adj); ROGUE <- CalculateRogue(.x, platform, cutoff, k, features)'
```

## Evaluation signals

- ROGUE values fall in ~0.99–1 across the entire 2–100 fold sequencing-depth change range
- Silhouette values fall in 0.25–0.75 on the same simulated data, contrasting with ROGUE stability
- Pearson correlations of S across cell down-sampling levels exceed 0.99
- Results assembled in rogue_simulation_results.csv match the quantitative values reported in the manuscript

## Limitations

- Silhouette values vary substantially (0.25–0.75) under the same conditions, so silhouette cannot be used as the robustness criterion here
- The recommended 0.9 purity threshold may not suit low-quality or continuous data; the threshold must then be determined from global ROGUE values
- ROGUE does not yet assess purity of integrated cell populations from different protocols and platforms (proposed as future work)

## Evidence

- [methods] ROGUE robust to depth/cell-number variation: "matr.filter(expr, min.cells, min.genes)"
- [methods] S estimation and ROGUE computation: "entropy_fit(.x, span, mt.method)"
- [methods] Reproduction scripts available: "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)"
- [readme] Installation from GitHub: "devtools::install_github("PaulingLiu/ROGUE")"
- [abstract] ROGUE purpose for cluster purity: "Here we present an entropy-based statistic, ROGUE, to accurately quantify the purity of identified cell clusters."
