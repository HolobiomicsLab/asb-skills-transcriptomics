---
name: parallel-scaling-plateau-analysis
description: Use when nested CV fitting (e.g. 10 x 10-fold nestcv.train or nestcv.glmnet)
  is too slow serially and you need to quantify how much wall-clock time cv.cores
  parallelisation saves on multi-core hardware.
when_to_use_negative:
- 'Single-core machines or CI runners: mclapply-based outer-loop parallelisation cannot
  yield speedup and timings would be misleading'
- When the analysis goal is model accuracy or hyperparameter tuning rather than wall-clock
  performance benchmarking — cv.cores does not change fitted results
- Windows-only environments where you cannot use mclapply semantics; nestedcv falls
  back to parLapply and timing behaviour differs
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3577
tools:
- name: nestedcv
  role: Provides nestcv.train/nestcv.glmnet with the cv.cores argument that parallelises
    the outer CV loop; the object being benchmarked
  repo: https://CRAN.R-project.org/package=nestedcv
- name: parallel
  role: Supplies mclapply (unix/mac) / parLapply (windows) used internally to parallelise
    the outer CV loop, and detectCores(logical = FALSE) for physical-core detection
- name: randomForest
  role: Random forest engine behind nestcv.train in the speedup benchmark (largest
    gain, up to 5.0x)
- name: glmnet
  role: Regularised regression engine behind nestcv.glmnet in the benchmark (smaller
    gain, up to 2.5x)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad048
    title: 'nestedcv: an R package for fast implementation of nested cross-validation
      with embedded feature selection designed for transcriptomics and high-dimensional
      data'
  source_content_hash: sha256:70f101eafaf5ccde5e702072f907c02a794654a7c645bbce2a5b0f50b56b4dba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - glmnet
  - nestedcv
  - parallel
  - randomForest
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# parallel-scaling-plateau-analysis

## Summary

Benchmark the wall-clock speedup gained by parallelising the outer CV loop of nestedcv model fitting (nestcv.train, nestcv.glmnet) with the cv.cores argument, verifying that speedup plateaus near the number of physical cores.

## When to use

Use when nested CV fitting (e.g. 10 x 10-fold nestcv.train or nestcv.glmnet) is too slow serially and you need to quantify how much wall-clock time cv.cores parallelisation saves on multi-core hardware.

## When NOT to use

- Single-core machines or CI runners: mclapply-based outer-loop parallelisation cannot yield speedup and timings would be misleading
- When the analysis goal is model accuracy or hyperparameter tuning rather than wall-clock performance benchmarking — cv.cores does not change fitted results
- Windows-only environments where you cannot use mclapply semantics; nestedcv falls back to parLapply and timing behaviour differs

## Inputs

- R dataset for nested CV (e.g. R4RA rheumatoid arthritis RNA-Seq E-MTAB-11611, Sonar, iris, BostonHousing2)
- Outcome vector y and predictor matrix x
- Model specification (nestcv.train with method e.g. 'ranger'/'rf', or nestcv.glmnet with family)
- Multi-core machine with known physical core count (parallel::detectCores(logical = FALSE))

## Outputs

- Wall-clock timing table for serial vs parallel runs per model (e.g. benchmark_speedup.csv)
- Per-model speedup ratios (serial time / parallel time) with expected maxima ~5.0x (nestcv.train random forest) and ~2.5x (nestcv.glmnet)
- Fitted nestcv objects from the parallel runs (summaries via summary())

## How to apply

Fit the target model twice on the same data: once with cv.cores = 1 and once with a multi-core setting (e.g. cv.cores = 8), recording wall-clock time for each run. nestedcv parallelises the outer CV loop via mclapply from the parallel package on unix/mac and parLapply on windows; detect physical cores with parallel::detectCores(logical = FALSE) rather than logical cores. Compute the speedup ratio (serial time / parallel time) per model and check it is positive (> 1x) and plateaus near the physical core count. In the R4RA rheumatoid arthritis RNA-Seq dataset (E-MTAB-11611) benchmark with default 10 x 10 outer/inner folds, random forest via nestcv.train sped up up to 5.0x while nestcv.glmnet improved less (up to 2.5x), so expect model-dependent ceilings — glmnet's C-level BLAS threading limits the outer-loop gain.

## Related tools

- **nestedcv** (Provides nestcv.train/nestcv.glmnet with the cv.cores argument that parallelises the outer CV loop; the object being benchmarked) — https://CRAN.R-project.org/package=nestedcv
- **parallel** (Supplies mclapply (unix/mac) / parLapply (windows) used internally to parallelise the outer CV loop, and detectCores(logical = FALSE) for physical-core detection)
- **randomForest** (Random forest engine behind nestcv.train in the speedup benchmark (largest gain, up to 5.0x))
- **glmnet** (Regularised regression engine behind nestcv.glmnet in the benchmark (smaller gain, up to 2.5x))

## Examples

```
cores <- parallel::detectCores(logical = FALSE); res <- nestcv.glmnet(y, x, family = "binomial", cv.cores = cores)
```

## Evaluation signals

- Each model's speedup ratio (serial time / parallel time) must be positive and > 1x
- Parallel timing should plateau near the number of physical cores (detectCores(logical = FALSE)), not logical cores
- nestcv.train random forest speedup should approach the reported maximum of ~5.0x; nestcv.glmnet ~2.5x — large deviations suggest misconfigured cores or I/O overhead
- Serial and parallel runs must use identical fold settings (default 10 x 10 outer/inner folds) so timing differences reflect only parallelisation
- Timing and speedup results written to benchmark_speedup.csv should be internally consistent (serial_time / parallel_time == speedup)

## Limitations

- Speedup is model-dependent: nestcv.glmnet gains less (up to 2.5x) than random forest via nestcv.train (up to 5.0x), so glmnet benchmarks should not be extrapolated to other engines
- Parallelisation applies only to the outer CV loop (nestcv.glmnet, nestcv.train, nestcv.SuperLearner, outercv); inner-loop work remains serial
- Platform-dependent implementation: mclapply on unix/mac vs parLapply on windows, so absolute timings are not portable across operating systems

## Evidence

- [other] Parallelisation via cv.cores improved performance by up to 5.0 times for random forest fitted using nestcv.train(), while improvement for nestcv.glmnet() was less (up to 2.5 times).: [evidence span withheld — anchor status: not_found]
- [intro] Currently `nestcv.glmnet`, `nestcv.train`, `nestcv.SuperLearner` and `outercv` all allow parallelisation of the outer CV loop using `mclapply` from the `parallel` package on unix/mac and `parLapply`: "Currently `nestcv.glmnet`, `nestcv.train`, `nestcv.SuperLearner` and `outercv` all allow parallelisation of the outer CV loop using `mclapply` from the `parallel` package on unix/mac and `parLapply`"
- [readme] cores <- parallel::detectCores(logical = FALSE)  # detect physical cores

res <- nestcv.glmnet(y, x, family = "multinomial", cv.cores = cores): "cores <- parallel::detectCores(logical = FALSE)  # detect physical cores

res <- nestcv.glmnet(y, x, family = "multinomial", cv.cores = cores)"
- [intro] `nestedcv` partitions the dataset into outer and inner folds (default 10 x 10
folds).: "`nestedcv` partitions the dataset into outer and inner folds (default 10 x 10
folds)."
