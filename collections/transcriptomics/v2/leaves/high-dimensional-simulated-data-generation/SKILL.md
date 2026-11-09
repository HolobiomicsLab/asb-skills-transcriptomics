---
name: high-dimensional-simulated-data-generation
description: Use when benchmarking or stress-testing penalized regression (Ridge/Lasso/adaptive
  Lasso logistic or Cox) and you need synthetic n×p covariate grids with matching
  outcome columns instead of real patient data.
when_to_use_negative:
- When the analysis requires real patient-level genomic/clinical data rather than
  simulation (use the HDMAC TCGA download R codes instead)
- When the variable count approaches a hundred thousand — the article states penalized
  regression models fall short there and FDR screening is recommended
- When the outcome is neither binary nor a survival endpoint (HDMAC's routines target
  survival and binary outcomes only)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: HDMAC
  role: Platform whose penalized regression routines (Ridge/Lasso/adaptive Lasso;
    Cox for survival, logistic for binary outcomes) are benchmarked on the simulated
    data
  repo: https://github.com/chung-R/HD-MAC
- name: R
  role: Statistical computing language in which the simulation, model fitting, and
    timing are performed
  repo: https://github.com/chung-R/HD-MAC
- name: Shiny
  role: R package backing the HDMAC web app; simulations allow offline-equivalent
    benchmarking of its regression routines
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-020-60791-z
    title: 'HDMAC: A Web-Based Interactive Program for High-Dimensional Analysis of
      Molecular Alterations in Cancer'
  source_content_hash: sha256:e077e3f712ef18bcb7d96d54af4731bcb7096a5e9ea7117401e6ed32ff4f3383
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chung-R/HD-MAC
  tools:
  - HDMAC
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# high-dimensional-simulated-data-generation

## Summary

Generates simulated high-dimensional covariate matrices — with binary outcomes for logistic regression and survival time/censoring indicators for Cox regression — mirroring the TCGA ovarian and bladder cancer datasets, across the 9 combinations of observations {50, 200, 1000} × variables {50, 500, 5000} used to benchmark HDMAC's penalized regression computing times.

## When to use

Use when benchmarking or stress-testing penalized regression (Ridge/Lasso/adaptive Lasso logistic or Cox) and you need synthetic n×p covariate grids with matching outcome columns instead of real patient data.

## When NOT to use

- When the analysis requires real patient-level genomic/clinical data rather than simulation (use the HDMAC TCGA download R codes instead)
- When the variable count approaches a hundred thousand — the article states penalized regression models fall short there and FDR screening is recommended
- When the outcome is neither binary nor a survival endpoint (HDMAC's routines target survival and binary outcomes only)

## Inputs

- Reference real datasets: TCGA ovarian and bladder cancer data (source for simulation parameters)
- Grid specification: observations {50, 200, 1000} × variables {50, 500, 5000} (9 combinations)
- Outcome specification: binary outcome for logistic regression; survival time and censoring indicator for Cox regression
- Model choice among Ridge, Lasso, or adaptive Lasso penalized regression

## Outputs

- Simulated covariate matrices (n×p) for each of the 9 grid combinations, with binary outcome and/or survival time + censoring indicator columns
- Average computing times in seconds per model and grid combination
- computing_times.csv — a table with rows by observation count and columns by variable count (mirroring Table 6 of the paper)

## How to apply

Generate simulated covariate matrices modeled on the real TCGA ovarian and bladder cancer datasets from the paper, enumerating all 9 combinations of observations {50, 200, 1000} by variables {50, 500, 5000}. For each simulated matrix, attach a binary outcome column when the target model is HDMAC's logistic regression routine, or a survival time plus censoring indicator when the target is HDMAC's Lasso Cox regression routine, implemented in R. Fit the corresponding model (e.g., adaptive Lasso logistic regression, Lasso Cox regression) per combination and record average computing time in seconds. Assemble timings into a table with rows indexed by observation count and columns by variable count (mirroring the paper's Table 6) and save it as computing_times.csv. Note that HDMAC provides cross-validation to estimate prediction power of each regression model, and that the authors recommend FDR first-step screening instead of penalized regression when variables approach the hundred-thousand magnitude.

## Related tools

- **HDMAC** (Platform whose penalized regression routines (Ridge/Lasso/adaptive Lasso; Cox for survival, logistic for binary outcomes) are benchmarked on the simulated data) — https://github.com/chung-R/HD-MAC
- **R** (Statistical computing language in which the simulation, model fitting, and timing are performed) — https://github.com/chung-R/HD-MAC
- **Shiny** (R package backing the HDMAC web app; simulations allow offline-equivalent benchmarking of its regression routines)

## Evaluation signals

- Computing time increases monotonically with both the number of observations and the number of variables across the 9-combination grid
- For the largest observation count, Cox regression time exceeds logistic regression time: 12.8 s vs 4.3 s at 1000 observations × 50 variables, and 128.2 s vs 16.4 s at 1000 × 5000
- The grid contains exactly 9 combinations: 3 observation counts × 3 variable counts
- Simulated matrices carry the correct outcome type per model: binary outcome for logistic, survival time + censoring indicator for Cox
- computing_times.csv exists and its row/column structure matches Table 6 of the paper (rows = observations, columns = variables)

## Limitations

- Timings derive from simulated data modeled on TCGA ovarian/bladder datasets, not from the actual TCGA matrices, so absolute times may differ from real-data runs
- Penalized regression models fall short at around a hundred thousand variables; the article recommends including FDR screening for such magnitudes, so this simulation grid (up to 5000 variables) does not cover that regime
- Cross-validation (default 5-fold in HDMAC) adds computing overhead that should be accounted for when comparing timings

## Evidence

- [methods] R used for simulation and benchmarking: "The platform was written and all statistical analysis was performed with the statistical computing and graphic drawing language, R"
- [abstract] Cross validation estimates prediction power: "Cross validation is provided as the method to estimate the prediction power of each regression model."
- [discussion] FDR recommended at very large variable counts: "Inclusion of FDR is recommended if users are dealing with variables at the magnitude of a hundred thousand where penalized regression models fall short"
- [intro] Objective: penalized regression for survival and binary outcomes with high-dimensional covariates: "we aimed to develop a web-based interactive and user-friendly platform to fulfill the following goals. First, it would fit the regression models with survival and binary outcomes and high-dimensional"
