---
name: runtime-scaling-benchmarking
description: Use when you need to quantify or verify how penalized regression runtimes
  in HDMAC (R/Shiny) scale with number of observations and high-dimensional variables,
  or to reproduce the paper's computing-time table.
when_to_use_negative:
- When the variable count is at the magnitude of a hundred thousand, where the paper
  states penalized regression models fall short and FDR first-step screening is recommended
  instead of timing benchmarks
- When the goal is assessing predictive performance rather than runtime — use the
  provided 5-fold cross-validation instead of timing loops
- When data are low-dimensional (small numbers of observations and variables), where
  scaling behavior of penalized regression is not informative
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: HDMAC
  role: Provides the penalized regression routines (adaptive Lasso logistic regression,
    Lasso Cox regression) whose computing times are benchmarked
  repo: https://github.com/chung-R/HD-MAC
- name: R
  role: Statistical computing language in which all benchmark fits and timing measurements
    are executed
- name: Shiny
  role: R package underlying the HDMAC web app; the same routines can be exercised
    offline via the HDMAC R scripts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-020-60791-z
    title: 'HDMAC: A Web-Based Interactive Program for High-Dimensional Analysis of
      Molecular Alterations in Cancer'
  source_content_hash: sha256:9fd5c372032c34d43336953463bdf191c996872c43d286e9105a4e1f9f019798
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chung-R/HD-MAC
  tools:
  - HDMAC
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# runtime-scaling-benchmarking

## Summary

Benchmark the computing time of HDMAC's penalized regression routines — adaptive Lasso logistic regression for binary outcomes and Lasso Cox regression for survival outcomes — across a grid of observation counts {50, 200, 1000} and variable counts {50, 500, 5000} to characterize how runtime scales with data size, mirroring Table 6 of the HDMAC paper.

## When to use

Use when you need to quantify or verify how penalized regression runtimes in HDMAC (R/Shiny) scale with number of observations and high-dimensional variables, or to reproduce the paper's computing-time table.

## When NOT to use

- When the variable count is at the magnitude of a hundred thousand, where the paper states penalized regression models fall short and FDR first-step screening is recommended instead of timing benchmarks
- When the goal is assessing predictive performance rather than runtime — use the provided 5-fold cross-validation instead of timing loops
- When data are low-dimensional (small numbers of observations and variables), where scaling behavior of penalized regression is not informative

## Inputs

- Simulated covariate matrices modeled on TCGA ovarian and bladder cancer data, in 9 size combinations: observations {50, 200, 1000} x variables {50, 500, 5000}
- Binary outcome vectors (0/1) for the logistic regression benchmarks
- Survival time and censoring indicator vectors for the Cox regression benchmarks
- HDMAC R scripts (run offline in RStudio or via the Shiny app)

## Outputs

- computing_times.csv: table with rows indexed by observation count and columns by variable count, containing average runtimes in seconds for adaptive Lasso logistic regression and Lasso Cox regression
- Comparison against the paper's reference values (e.g., 12.8 s vs 4.3 s for 1000 observations x 50 variables; 128.2 s vs 16.4 s for 1000 x 5000)

## How to apply

Generate simulated covariate matrices based on the real TCGA ovarian and bladder cancer datasets used in the paper, covering all 9 combinations of observations {50, 200, 1000} x variables {50, 500, 5000}; binary outcomes accompany the logistic matrix and survival time plus censoring indicator accompany the Cox matrix. For each grid cell, fit the adaptive Lasso logistic regression via HDMAC's logistic regression routine in R and record average computing time in seconds, then fit the Lasso Cox regression via HDMAC's survival analysis routine and record its average time. Assemble both timing sets into a single table with rows indexed by observation count and columns by variable count, mirroring the paper's Table 6, and save it as computing_times.csv. The rationale is that Cox survival fitting is more expensive than logistic regression, so the benchmark documents when HDMAC's penalized models remain tractable before FDR screening becomes necessary for very large variable counts.

## Related tools

- **HDMAC** (Provides the penalized regression routines (adaptive Lasso logistic regression, Lasso Cox regression) whose computing times are benchmarked) — https://github.com/chung-R/HD-MAC
- **R** (Statistical computing language in which all benchmark fits and timing measurements are executed)
- **Shiny** (R package underlying the HDMAC web app; the same routines can be exercised offline via the HDMAC R scripts)

## Evaluation signals

- Computing time increases monotonically with the number of observations in both the logistic and Cox benchmark tables
- Computing time increases monotonically with the number of variables in both benchmark tables
- For the largest observation count, survival-analysis time exceeds logistic-regression time, matching the reported pattern (12.8 s vs 4.3 s at 1000 x 50; 128.2 s vs 16.4 s at 1000 x 5000)
- computing_times.csv has 3 rows x 3 columns covering all 9 grid combinations, mirroring the structure of the paper's Table 6
- All 9 grid cells contain finite, positive timing values averaged over repeated fits

## Limitations

- Penalized regression models fall short at around a hundred thousand variables; FDR screening is recommended at that scale instead of fitting
- Benchmark timings are based on simulated covariate matrices modeled on TCGA ovarian and bladder cancer data, so absolute times may vary with hardware and data specifics
- The default cross-validation fold number is 5 in HDMAC, which adds overhead not part of a raw fit timing if left enabled

## Evidence

- [abstract] HDMAC penalized regression: [evidence span withheld — anchor status: not_found]
- [methods] R implementation: "The platform was written and all statistical analysis was performed with the statistical computing and graphic drawing language, R"
- [methods] Lasso Cox estimator: "The Cox Lasso regression estimator"
- [discussion] Scale limitation: "Inclusion of FDR is recommended if users are dealing with variables at the magnitude of a hundred thousand where penalized regression models fall short"
- [methods] Cross validation default: "The default fold number is 5, and cross validation method will not be performed if 1 is chosen"
