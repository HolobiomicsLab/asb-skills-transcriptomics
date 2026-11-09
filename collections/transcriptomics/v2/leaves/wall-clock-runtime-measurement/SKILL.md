---
name: wall-clock-runtime-measurement
description: Use when quantifying the computational cost or parallel speedup of nested
  cross-validation (default 10 x 10 outer/inner folds) on high-dimensional datasets,
  e.g. RNA-Seq matrices with P >> n where full nested CV is expensive.
when_to_use_negative:
- When the goal is unbiased accuracy estimation rather than runtime benchmarking —
  cv.cores only parallelises the outer CV loop and does not change model performance
  estimates
- When the dataset is small enough that nested CV completes quickly (e.g. the iris
  example) — parallelisation overhead may swamp any gain
- When the model fits a final model on the whole dataset only, with no nested outer/inner
  loop to parallelise
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: nestedcv
  role: Provides nestcv.train and nestcv.glmnet whose cv.cores argument parallelises
    the outer CV loop being benchmarked
  repo: https://CRAN.R-project.org/package=nestedcv
- name: R
  role: Runtime environment in which timing loops and the parallel package are executed
- name: glmnet
  role: Fits the elastic net models inside nestcv.glmnet for the slower-scaling (2.5x)
    benchmark arm
- name: randomForest
  role: Random forest engine behind the nestcv.train arm showing the largest speedup
    (up to 5.0x)
- name: parallel
  role: Supplies mclapply (unix/mac) / parLapply (windows) used by cv.cores, and detectCores(logical
    = FALSE) to pick the parallel worker count
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad048
    title: 'nestedcv: an R package for fast implementation of nested cross-validation
      with embedded feature selection designed for transcriptomics and high-dimensional
      data'
  source_content_hash: sha256:d89a75e05667ec28a26d4a97e9d56d4096351c655d1e47d88206b884b71b5899
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - glmnet
  - nestedcv
  - parallel
  - randomForest
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# wall-clock-runtime-measurement

## Summary

Benchmarks the wall-clock speedup gained by parallelising the outer CV loop of nestedcv functions (nestcv.train, nestcv.glmnet) via the cv.cores argument, comparing serial vs parallel run times to quantify performance gains on high-dimensional transcriptomic data.

## When to use

Use when quantifying the computational cost or parallel speedup of nested cross-validation (default 10 x 10 outer/inner folds) on high-dimensional datasets, e.g. RNA-Seq matrices with P >> n where full nested CV is expensive.

## When NOT to use

- When the goal is unbiased accuracy estimation rather than runtime benchmarking — cv.cores only parallelises the outer CV loop and does not change model performance estimates
- When the dataset is small enough that nested CV completes quickly (e.g. the iris example) — parallelisation overhead may swamp any gain
- When the model fits a final model on the whole dataset only, with no nested outer/inner loop to parallelise

## Inputs

- Expression matrix x (e.g. RNA-Seq count/normalised matrix from E-MTAB-11611 R4RA rheumatoid arthritis dataset) with outcome vector y
- Model specification: nestcv.train with a caret method (e.g. 'ranger'/'rf' random forest) or nestcv.glmnet with a glmnet family
- Core count from parallel::detectCores(logical = FALSE) (physical cores)

## Outputs

- Wall-clock timings for serial (cv.cores = 1) and parallel (cv.cores = N) runs per model
- Speedup ratio (serial time / parallel time) per model, e.g. up to 5.0x for nestcv.train random forest and up to 2.5x for nestcv.glmnet
- benchmark_speedup.csv containing timing and speedup results

## How to apply

Fit the model twice: once with cv.cores = 1 (serial) and once with cv.cores set to the number of physical cores, detected via parallel::detectCores(logical = FALSE), recording wall-clock time for each run. On unix/mac the outer CV loop is parallelised with mclapply from the parallel package; on Windows parLapply is used instead. Use the same fold structure (nestedcv default is 10 x 10 outer/inner folds, stratified on the outcome) for both runs so the speedup ratio (serial time / parallel time) is attributable solely to cv.cores. Run separately for each model type: random forest via nestcv.train and elastic net via nestcv.glmnet, since the paper reports different maxima (5.0x vs 2.5x). Write timings and speedup ratios to a CSV (e.g. benchmark_speedup.csv) and check that speedup plateaus near the physical core count.

## Related tools

- **nestedcv** (Provides nestcv.train and nestcv.glmnet whose cv.cores argument parallelises the outer CV loop being benchmarked) — https://CRAN.R-project.org/package=nestedcv
- **R** (Runtime environment in which timing loops and the parallel package are executed)
- **glmnet** (Fits the elastic net models inside nestcv.glmnet for the slower-scaling (2.5x) benchmark arm)
- **randomForest** (Random forest engine behind the nestcv.train arm showing the largest speedup (up to 5.0x))
- **parallel** (Supplies mclapply (unix/mac) / parLapply (windows) used by cv.cores, and detectCores(logical = FALSE) to pick the parallel worker count)

## Examples

```
cores <- parallel::detectCores(logical = FALSE); res <- nestcv.glmnet(y, x, family = "multinomial", cv.cores = cores); summary(res)
```

## Evaluation signals

- Each model's speedup ratio (serial time / parallel time) must be positive and > 1x
- Parallel timing should plateau near the number of physical cores (parallel::detectCores(logical = FALSE))
- nestcv.train random forest speedup should reach up to ~5.0x; nestcv.glmnet up to ~2.5x
- Serial and parallel runs must use identical fold structure (nestedcv default 10 x 10 folds) so results are comparable
- benchmark_speedup.csv exists and contains timing and speedup columns for both model arms

## Limitations

- Parallelisation targets only the outer CV loop (nestcv.glmnet, nestcv.train, nestcv.SuperLearner and outercv); the inner loop is not parallelised, capping achievable speedup
- Platform-dependent backend: mclapply on unix/mac vs parLapply on Windows, so speedups are not directly comparable across operating systems
- Speedup varies by model type — random forest (nestcv.train) gains far more than glmnet — and plateaus near the physical core count; using logical cores does not guarantee further gains

## Evidence

- [other] Parallelisation via cv.cores improved performance by up to 5.0 times for random forest fitted using nestcv.train(), while improvement for nestcv.glmnet() was less (up to 2.5 times).: [evidence span withheld — anchor status: not_found]
- [intro] parallelisation of the outer CV loop using mclapply from the parallel package on unix/mac and parLapply: "parallelisation of the outer CV loop using `mclapply` from the
`parallel` package on unix/mac and `parLapply`"
- [intro] partitions the dataset into outer and inner folds (default 10 x 10 folds): "`nestedcv` partitions the dataset into outer and inner folds (default 10 x 10
folds)."
- [readme] cores <- parallel::detectCores(logical = FALSE)  # detect physical cores: "cores <- parallel::detectCores(logical = FALSE)  # detect physical cores"
