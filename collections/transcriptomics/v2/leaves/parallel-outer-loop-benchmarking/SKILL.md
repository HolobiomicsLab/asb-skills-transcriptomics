---
name: parallel-outer-loop-benchmarking
description: Use when timing nested CV fits of nestedcv (e.g. nestcv.train with random
  forest, nestcv.glmnet) on high-dimensional transcriptomic data and comparing serial
  (cv.cores=1) vs parallel (cv.cores=8) wall-clock times.
when_to_use_negative:
- When the goal is faster fitting rather than benchmarking — simply set cv.cores to
  the physical core count and run once
- When the model call does not support outer-loop parallelisation (cv.cores is available
  for nestcv.glmnet, nestcv.train, nestcv.SuperLearner and outercv, not all nestedcv
  functions)
- When the critical cost is inside inner CV folds or the model fit itself rather than
  the outer loop — parallelising outer folds alone will yield little gain
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: nestedcv
  role: Provides nestcv.train, nestcv.glmnet and the cv.cores argument that parallelises
    the outer CV loop
  repo: https://CRAN.R-project.org/package=nestedcv
- name: parallel
  role: 'Backend used by cv.cores: mclapply on unix/mac and parLapply on windows to
    distribute outer folds'
- name: glmnet
  role: Fits the elastic-net linear models whose nestcv.glmnet parallel speedup (up
    to 2.5x) is benchmarked
- name: randomForest
  role: Random forest engine for nestcv.train whose parallel speedup (up to 5.0x)
    is benchmarked
- name: caret
  role: Framework through which nestcv.train accesses model-fitting methods such as
    random forest
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad048
    title: 'nestedcv: an R package for fast implementation of nested cross-validation
      with embedded feature selection designed for transcriptomics and high-dimensional
      data'
  source_content_hash: sha256:e155335c85e190f8b3cf13ef3577427ff9baa7231e99d7285524668fd726ade5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - caret
  - glmnet
  - nestedcv
  - parallel
  - randomForest
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# parallel-outer-loop-benchmarking

## Summary

Benchmarks wall-clock speedup from parallelising the outer CV loop of nestedcv model fits (nestcv.train, nestcv.glmnet) via the cv.cores argument, which dispatches outer folds with mclapply from the parallel package. The paper reports up to 5.0x speedup for random forest nestcv.train and up to 2.5x for nestcv.glmnet on the R4RA dataset (10 x 10 default folds).

## When to use

Use when timing nested CV fits of nestedcv (e.g. nestcv.train with random forest, nestcv.glmnet) on high-dimensional transcriptomic data and comparing serial (cv.cores=1) vs parallel (cv.cores=8) wall-clock times.

## When NOT to use

- When the goal is faster fitting rather than benchmarking — simply set cv.cores to the physical core count and run once
- When the model call does not support outer-loop parallelisation (cv.cores is available for nestcv.glmnet, nestcv.train, nestcv.SuperLearner and outercv, not all nestedcv functions)
- When the critical cost is inside inner CV folds or the model fit itself rather than the outer loop — parallelising outer folds alone will yield little gain

## Inputs

- High-dimensional predictor matrix x and outcome vector y (e.g. R4RA rheumatoid arthritis RNA-Seq data from E-MTAB-11611)
- nestedcv model call (nestcv.train with method such as 'ranger', or nestcv.glmnet) with default 10 x 10 outer/inner folds
- Core count from parallel::detectCores(logical = FALSE) for the cv.cores argument

## Outputs

- Serial and parallel wall-clock times per model
- Speedup ratio (serial time / parallel time) per model
- benchmark_speedup.csv containing timing and speedup results

## How to apply

Fit the same model (default 10 x 10 outer/inner folds, stratified via caret's createFolds) twice: once with cv.cores = 1 and once with cv.cores set to the physical core count detected by parallel::detectCores(logical = FALSE), since cv.cores parallelises only the outer CV loop (via mclapply on unix/mac, parLapply on windows, or the future framework). Record wall-clock time for each run and compute speedup = serial time / parallel time. On the R4RA rheumatoid arthritis RNA-Seq dataset (E-MTAB-11611), validate the ratio against the reported maxima of 5.0x for nestcv.train random forest and 2.5x for nestcv.glmnet; the smaller glmnet gain reflects that only outer folds are distributed while inner CV and coordinate descent remain serial. Write per-model timing and speedup values to a file such as benchmark_speedup.csv.

## Related tools

- **nestedcv** (Provides nestcv.train, nestcv.glmnet and the cv.cores argument that parallelises the outer CV loop) — https://CRAN.R-project.org/package=nestedcv
- **parallel** (Backend used by cv.cores: mclapply on unix/mac and parLapply on windows to distribute outer folds)
- **glmnet** (Fits the elastic-net linear models whose nestcv.glmnet parallel speedup (up to 2.5x) is benchmarked)
- **randomForest** (Random forest engine for nestcv.train whose parallel speedup (up to 5.0x) is benchmarked)
- **caret** (Framework through which nestcv.train accesses model-fitting methods such as random forest)

## Examples

```
cores <- parallel::detectCores(logical = FALSE); res <- nestcv.train(y, x, method = "ranger", cv.cores = cores)
```

## Evaluation signals

- Each model's speedup ratio (serial time / parallel time) is positive and > 1x
- Parallel timing plateaus near the number of physical cores (detectCores(logical = FALSE))
- Random forest nestcv.train speedup approaches the reported maximum of ~5.0x
- nestcv.glmnet speedup stays near the reported ~2.5x maximum, reflecting serial inner CV
- Outer fold predictions and tuned hyperparameters are unchanged between the serial and parallel runs

## Limitations

- Only the outer CV loop is parallelised; inner CV and model fitting run serially, so glmnet gains are limited (~2.5x)
- cv.cores uses mclapply on unix/mac but parLapply on windows (with additional setup), and a future-based alternative exists — behaviour differs by platform
- Speedup depends on core count and plateaus near the physical core count; logical cores give no further benefit

## Evidence

- [intro] Outer-loop parallelisation mechanism: "allow parallelisation of the outer CV loop using `mclapply` from the `parallel` package on unix/mac and `parLapply`"
- [readme] Physical core detection for cv.cores: "cores <- parallel::detectCores(logical = FALSE)  # detect physical cores"
- [readme] Example parallel nested CV fit: "res <- nestcv.glmnet(y, x, family = "multinomial", cv.cores = cores)"
