---
name: single-cell-log2-tpm-preprocessing
description: Use when raw unsynchronized scRNA-seq expression (e.g. TPM values in
  ola_mES_2i.RData or youngMpp .RData) must be converted to log2(TPM+1) and restricted
  to the cycle base genes to build reCAT's standard test_exp input.
when_to_use_negative:
- The expression matrix is already log2-transformed or already restricted to cycle
  base genes — re-applying get_test_exp would double-transform or over-filter the
  data.
- The analysis goal requires data-driven (semi-supervised) selection of cell cycle
  genes from the data rather than the preselected 378-gene set, which the authors
  themselves flag as a limitation.
- The input is not unsynchronized single-cell transcriptome data (e.g. bulk RNA-seq
  or microarrays), for which the reCAT test_exp convention was not designed.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: reCAT
  role: Provides the get_test_exp function that produces the log2(TPM+1), cycle-base-gene-filtered
    standard input data
  repo: github.com/tinglab/reCAT
- name: R
  role: Language in which get_test_exp and the reCAT pipeline are implemented and
    run
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s41467-017-00039-z
    title: Reconstructing cell cycle pseudo time-series via single-cell transcriptome
      data
  source_content_hash: sha256:c2abc1b1298214adcb486f3d7aa61c72b8ac0547dd95bb7fc9dd3abff4cf06d5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - TSP
  - cluster
  - doParallel
  - mclust
  - reCAT
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# single-cell-log2-tpm-preprocessing

## Summary

reCAT's first-stage data preparation: transform unsynchronized single-cell expression values to log2(TPM+1) and filter to the cycle base genes so the matrix becomes the standard test_exp input for downstream get_ordIndex/get_score steps.

## When to use

Use when raw unsynchronized scRNA-seq expression (e.g. TPM values in ola_mES_2i.RData or youngMpp .RData) must be converted to log2(TPM+1) and restricted to the cycle base genes to build reCAT's standard test_exp input.

## When NOT to use

- The expression matrix is already log2-transformed or already restricted to cycle base genes — re-applying get_test_exp would double-transform or over-filter the data.
- The analysis goal requires data-driven (semi-supervised) selection of cell cycle genes from the data rather than the preselected 378-gene set, which the authors themselves flag as a limitation.
- The input is not unsynchronized single-cell transcriptome data (e.g. bulk RNA-seq or microarrays), for which the reCAT test_exp convention was not designed.

## Inputs

- Raw single-cell expression matrix with TPM values (e.g. ola_mES_2i.RData, youngMpp .RData, mESC-SMARTer data)
- Cycle base gene list (ccgene from github.com/tinglab/reCAT; preselected 378 genes)

## Outputs

- test_exp: log2(TPM+1) expression matrix restricted to cycle base genes only (standard reCAT input)
- Orientation-checked matrix ready for get_ordIndex and get_score(t(test_exp))

## How to apply

Load the raw single-cell expression matrix (e.g. ola_mES_2i.RData) and apply reCAT's get_test_exp function, which performs two actions: it log2-transforms TPM values as log2(TPM+1) and filters the original data down to the cycle base genes only, making the data easy to handle in following steps. Mind the orientation: the README example applies get_test_exp to t(test_exp), i.e. the transpose of the loaded matrix, because downstream calls such as get_ordIndex(test_exp, ...) and get_score(t(test_exp)) expect that layout. The result, test_exp, is the sole input to the GMM clustering plus consensus-TSP ordering (get_ordIndex with threadnum = 10, step_size = 2, base_cycle_range = c(6:9)), so any error here propagates to the whole four-step reCAT pipeline (GMM -> consensus-TSP -> scoring -> HMM/Kalman). This step is deliberately semi-supervised: it depends on a preselected gene set (378 genes from ccgene/Cyclebase) rather than data-driven gene selection.

## Related tools

- **reCAT** (Provides the get_test_exp function that produces the log2(TPM+1), cycle-base-gene-filtered standard input data) — github.com/tinglab/reCAT
- **R** (Language in which get_test_exp and the reCAT pipeline are implemented and run)

## Examples

```
source("get_test_exp.R"); load("../data/ola_mES_2i.RData"); test_exp <- get_test_exp(t(test_exp))
```

## Evaluation signals

- All values in test_exp are on the log2(TPM+1) scale (non-negative, log-transformed), matching 'the value in test_exp should be log2(TPM+1)'.
- test_exp contains only cycle base gene rows/columns — a direct filter of the original data, so gene count equals the cycle base gene set size.
- Matrix orientation is correct for the next call: get_score is invoked as get_score(t(test_exp)) in the README workflow, confirming the expected transposition.
- Downstream sanity check: plot_bayes(score_result$bayes_score, ordIndex) and plot_mean(...) must show score peaks aligned with the judged phase regions (e.g. G1: 210-230, S: 170-190, G2/M: 40-60 for ola_mES_2i), which fails if preprocessing was wrong.

## Limitations

- Relies on a preselected gene set (378 genes) rather than data-driven gene selection; the authors state they 'would prefer semi-supervised selection of cell cycle genes from the data'.
- The filter is a hard restriction to cycle base genes ('we just do a filter on the original data'), so any gene-level signal outside the cycle base set is discarded before ordering and scoring.

## Evidence

- [intro] Data processing: quality control, normalization, and clustering of single cells using a Gaussian mixture model: "The first step is data processing, including quality control, normalization, and clustering of single cells using the Gaussian mixture model (GMM) according to a user-defined phase number k"
- [methods] test_exp values must be log2(TPM+1) via get_test_exp: "in reCAT, the value in test_exp should be log2(TPM+1), and then you can use ***get_test_exp*** function to get standard input data"
- [readme] README example call on transposed matrix: "test_exp <- get_test_exp(t(test_exp))"
- [readme] Filter rationale in README: "in test_exp, there are only cycle base gene's value, we just do a filter on the original data, so that the data is easy to handle in following steps"
- [discussion] Preference for data-driven gene selection: "Instead of the preselected gene set (378 genes), we would prefer semi-supervised selection of cell cycle genes from the data"
