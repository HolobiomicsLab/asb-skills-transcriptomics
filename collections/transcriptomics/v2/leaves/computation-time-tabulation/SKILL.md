---
name: computation-time-tabulation
description: Use when benchmarking or validating computational cost of penalized regression
  routines (Lasso/adaptive Lasso Cox or logistic) on high-dimensional genomic data
  across grids of sample and variable counts.
when_to_use_negative:
- 'When the goal is gene-level inference rather than timing — the paper recommends
  FDR screening instead: ''Inclusion of FDR is recommended if users are dealing with
  variables at the magnitude of a hundred thousand where penalized regression models
  fall short'''
- When using real TCGA patient covariates — this skill's benchmarking workflow relies
  on simulated covariate matrices patterned on the ovarian and bladder cancer datasets
- When the regression models are not penalized (Ridge/Lasso/adaptive Lasso) or the
  outcomes are not binary or survival type, as HDMAC's routines under test are defined
  for those outcome types
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: HDMAC
  role: Provides the penalized regression routines (adaptive Lasso logistic regression
    for binary outcomes and Lasso Cox regression for survival outcomes) whose computing
    times are measured and tabulated
  repo: https://github.com/chung-R/HD-MAC
- name: R
  role: Statistical computing language in which HDMAC is written and in which the
    timing benchmark fits are executed
  repo: https://github.com/chung-R/HD-MAC
- name: Shiny
  role: R package underlying HDMAC's web interface; the benchmarked routines are the
    same ones exposed through this interactive app
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-020-60791-z
    title: 'HDMAC: A Web-Based Interactive Program for High-Dimensional Analysis of
      Molecular Alterations in Cancer'
  source_content_hash: sha256:539b5b4620ffd65f53dc7583aabf2eef97f92767520a9d94f51c67f6d7f7c1e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chung-R/HD-MAC
  tools:
  - HDMAC
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# computation-time-tabulation

## Summary

Benchmarks HDMAC's penalized regression routines by recording average computing times of adaptive Lasso logistic regression (binary outcomes) and Lasso Cox regression (survival outcomes) across a 3x3 grid of observation counts {50, 200, 1000} and variable counts {50, 500, 5000}, reproducing the paper's Table 6 computing-time structure.

## When to use

Use when benchmarking or validating computational cost of penalized regression routines (Lasso/adaptive Lasso Cox or logistic) on high-dimensional genomic data across grids of sample and variable counts.

## When NOT to use

- When the goal is gene-level inference rather than timing — the paper recommends FDR screening instead: 'Inclusion of FDR is recommended if users are dealing with variables at the magnitude of a hundred thousand where penalized regression models fall short'
- When using real TCGA patient covariates — this skill's benchmarking workflow relies on simulated covariate matrices patterned on the ovarian and bladder cancer datasets
- When the regression models are not penalized (Ridge/Lasso/adaptive Lasso) or the outcomes are not binary or survival type, as HDMAC's routines under test are defined for those outcome types

## Inputs

- Simulated covariate matrices sized n x p for n in {50, 200, 1000} and p in {50, 500, 5000}
- Simulated binary outcome vectors for logistic regression runs
- Simulated survival times and censoring indicators for Cox regression runs
- HDMAC R routines for adaptive Lasso logistic regression and Lasso Cox regression

## Outputs

- computing_times.csv: a table with rows indexed by observation count and columns by variable count, containing average computing times in seconds for both logistic-regression and Cox-regression fits

## How to apply

Generate simulated covariate matrices based on the real TCGA ovarian and bladder cancer datasets used in the paper, covering the 9 combinations of observations {50, 200, 1000} x variables {50, 500, 5000}: simulate a binary outcome for logistic regression and survival time plus censoring indicator for Cox regression. For each combination, fit the adaptive Lasso logistic regression using HDMAC's logistic regression routine in R and record the average computing time in seconds; likewise fit the Lasso Cox regression using HDMAC's survival analysis routine. Assemble both timings into a single table with rows indexed by observation count and columns by variable count, mirroring the paper's Table 6, and save it as computing_times.csv. The rationale is that Cox penalized regression is computationally heavier than logistic regression, and the paper reports this gap widening with sample size.

## Related tools

- **HDMAC** (Provides the penalized regression routines (adaptive Lasso logistic regression for binary outcomes and Lasso Cox regression for survival outcomes) whose computing times are measured and tabulated) — https://github.com/chung-R/HD-MAC
- **R** (Statistical computing language in which HDMAC is written and in which the timing benchmark fits are executed) — https://github.com/chung-R/HD-MAC
- **Shiny** (R package underlying HDMAC's web interface; the benchmarked routines are the same ones exposed through this interactive app)

## Evaluation signals

- Computing time increases monotonically with the number of observations (50, 200, 1000) and with the number of variables (50, 500, 5000) in every cell of the table
- For the largest observation count (n=1000), survival-analysis (Lasso Cox) time exceeds logistic-regression time: 12.8 s vs 4.3 s at 1000 x 50 and 128.2 s vs 16.4 s at 1000 x 5000 as reported in the paper
- The output table contains all 9 combinations of observation count x variable count, mirroring the structure of the paper's Table 6
- computing_times.csv is produced with rows indexed by observation count and columns by variable count, and all entries are numeric seconds
- Each timing reflects the average over replicated fits, not a single run, since per-cell averages smooth out variance in R execution time

## Limitations

- Penalized regression models fall short at around one hundred thousand variables; the paper notes 'Inclusion of FDR is recommended if users are dealing with variables at the magnitude of a hundred thousand where penalized regression models fall short', so timings beyond the p=5000 grid are not representative
- Timings are obtained on simulated covariate matrices patterned on TCGA ovarian and bladder cancer data, not on the raw TCGA datasets themselves
- The repository README (chung-R/HD-MAC) contains no benchmark documentation, so function-level invocation details must come from the HDMAC R scripts themselves

## Evidence

- [abstract] Platform scope grounding the penalized routines being timed: "we developed a web-based interactive and user-friendly platform for high-dimensional analysis of molecular alterations in cancer (HDMAC)"
- [methods] HDMAC is implemented in R, the language used to run the timing benchmark: "The platform was written and all statistical analysis was performed with the statistical computing and graphic drawing language, R"
- [methods] Shiny is the R package behind the HDMAC interactive app whose routines are benchmarked: "with the help of Shiny, an R package that facilitates the building of interactive web Apps straight from R"
- [discussion] Scale limitation of penalized regression motivating the p=5000 timing grid ceiling: "Inclusion of FDR is recommended if users are dealing with variables at the magnitude of a hundred thousand where penalized regression models fall short"
