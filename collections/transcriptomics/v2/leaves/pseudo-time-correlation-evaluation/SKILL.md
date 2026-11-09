---
name: pseudo-time-correlation-evaluation
description: Use when a cell-cycle pseudo time-series has been recovered (e.g., via
  reCAT's get_ordIndex/get_hmm_order/get_start pipeline) and an independent benchmark
  ordering such as an Oscope-recovered series exists to quantify agreement.
when_to_use_negative:
- No independent ordering (e.g., Oscope series) is available for the same cells, so
  no cross-method correlation can be computed
- The input expression matrix has not been preprocessed by get_test_exp (values not
  log2(TPM+1), cycle base genes not retained)
- The goal is cell-cycle phase labeling of unsynchronized cells rather than validating
  a recovered time-series ordering against a benchmark
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: reCAT
  role: R package that recovers the cell cycle time-series (get_ordIndex, get_score,
    get_hmm_order, get_start) whose ordering is evaluated against the Oscope benchmark
  repo: github.com/tinglab/reCAT
- name: Oscope
  role: Independent method whose recovered oscillatory ordering serves as the reference
    series for the correlation-score comparison
- name: R
  role: Implementation environment in which reCAT and the correlation-score computation
    are run
- name: TSP
  role: Supports the consensus-TSP traveling-salesman heuristic used by get_ordIndex
    to order clusters
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-017-00039-z
    title: Reconstructing cell cycle pseudo time-series via single-cell transcriptome
      data
  source_content_hash: sha256:875fe0b7df82d787b69f03c99063638e6bbc121fb6cc9c1f19dd6649d48a997d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Oscope
  - R
  - TSP
  - cluster
  - doParallel
  - mclust
  - reCAT
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# pseudo-time-correlation-evaluation

## Summary

Validates a reCAT-recovered cell cycle pseudo time-series by correlating it against an independently recovered ordering (e.g., Oscope) on the same scRNA-seq data set, benchmarked on mESC-Quartz where the reported correlation-score is 0.54 (Fig. 3a).

## When to use

Use when a cell-cycle pseudo time-series has been recovered (e.g., via reCAT's get_ordIndex/get_hmm_order/get_start pipeline) and an independent benchmark ordering such as an Oscope-recovered series exists to quantify agreement.

## When NOT to use

- No independent ordering (e.g., Oscope series) is available for the same cells, so no cross-method correlation can be computed
- The input expression matrix has not been preprocessed by get_test_exp (values not log2(TPM+1), cycle base genes not retained)
- The goal is cell-cycle phase labeling of unsynchronized cells rather than validating a recovered time-series ordering against a benchmark

## Inputs

- reCAT-ordered cell time-series (ordIndex / hmm-ordered series from get_ordIndex, get_hmm_order, get_start) for the mESC-Quartz scRNA-seq data set
- Independently recovered Oscope ordering of the same cells
- log2(TPM+1) expression matrix filtered to cycle base genes (output of get_test_exp)

## Outputs

- Correlation-score curve for the reCAT-recovered cell cycle time-series
- Correlation-score curve for the Oscope-recovered time-series
- Scalar correlation-score value (benchmark: 0.54 on mESC-Quartz, Fig. 3a)

## How to apply

First run the full reCAT pipeline on the log2(TPM+1) preprocessed expression matrix: get_test_exp to retain cycle base genes, get_ordIndex (thread number matched to cores, e.g., 10; step_size = 2; base_cycle_range = c(6:9)) for the TSP ordering, get_score for Bayes- and mean-scores, get_hmm_order (cls_num = 3) for G1/S/G2/M segmentation, and get_start (nthread = 3) to fix the true cycle start by maximum HMM likelihood over all rotations. Then compute a correlation-score curve for the reCAT-recovered series and the independent Oscope-recovered series over the same cells. Compare the resulting correlation to the reported benchmark: on the cell-cycle-labeled mESC-Quartz data set the reCAT vs. Oscope correlation-score is 0.54, so a successful reproduction should recover a value consistent with 0.54 rather than a degraded or near-zero correlation.

## Related tools

- **reCAT** (R package that recovers the cell cycle time-series (get_ordIndex, get_score, get_hmm_order, get_start) whose ordering is evaluated against the Oscope benchmark) — github.com/tinglab/reCAT
- **Oscope** (Independent method whose recovered oscillatory ordering serves as the reference series for the correlation-score comparison)
- **R** (Implementation environment in which reCAT and the correlation-score computation are run)
- **TSP** (Supports the consensus-TSP traveling-salesman heuristic used by get_ordIndex to order clusters)

## Examples

```
source("get_ordIndex.R"); ordIndex <- get_ordIndex(test_exp, threadnum = 10, step_size = 2, base_cycle_range = c(6:9)); score_result <- get_score(t(test_exp)); start <- get_start(bayes_score = score_result$bayes_score, mean_score = score_result$mean_score, ordIndex = ordIndex, cls_num = 3, rdata = rdata, nthread = 3)
```

## Evaluation signals

- Correlation-score for reCAT vs. Oscope on mESC-Quartz reproduces the reported benchmark of 0.54 (metric_oscope_corr_054)
- Correlation-score curves for both reCAT and Oscope are computed on the identical cell set from the mESC-Quartz data set
- The recovered ordering is rotation-consistent: get_start selected the true cycle start by maximum HMM likelihood over all rotations, so phase alignment does not confound the correlation
- Input matrix passes the reCAT precondition: values are log2(TPM+1) and only cycle base genes are retained after get_test_exp

## Limitations

- The benchmark value (0.54) is specific to the cell-cycle-labeled mESC-Quartz data set and Oscope as the reference; other data sets or references will yield different correlations
- reCAT relies on a preselected gene set (378 genes) rather than data-driven gene selection; the authors prefer semi-supervised selection for better future performance
- Bayes-scores and mean-scores are noisy and need improvement to be more informative, which can affect the ordering being validated

## Evidence

- [results] mESC-Quartz correlation-score benchmark: [evidence span withheld — anchor status: not_found]
- [abstract] reCAT method overview: "we develop a traveling salesman problem and hidden Markov model-based computational method named reCAT, to recover cell cycle along time for unsynchronized"
- [methods] input preprocessing: "in reCAT, the value in test_exp should be log2(TPM+1), and then you can use ***get_test_exp*** function to get standard input data"
- [readme] get_ordIndex parameters: "the base_cycle_range is a range of four k’s for computing the reference cycle mentioned in the manuscript. This can be set to 6:9 (by default) or 7:10."
- [readme] choosing the true start: "you can use ***get_start*** function to get the real start"
- [discussion] gene-set limitation: "Instead of the preselected gene set (378 genes), we would prefer semi-supervised selection of cell cycle genes from the data"
