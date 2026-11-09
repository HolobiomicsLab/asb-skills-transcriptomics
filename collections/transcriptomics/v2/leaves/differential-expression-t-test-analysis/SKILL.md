---
name: differential-expression-t-test-analysis
description: Use when you have a simulated two-group scRNA-Seq dataset with known
  truly differentially expressed gene labels and need to quantify how imputation (e.g.,
  AutoClass) restores DE signal lost to dropout.
when_to_use_negative:
- Data is real (non-simulated) scRNA-Seq with no ground-truth DE gene labels — the
  median-t-statistic-over-truly-DE-genes metric cannot be computed
- The analysis goal is clustering or batch-effect assessment rather than differential
  expression recovery
- The count matrix has not been imputed and the question concerns imputation quality
  rather than DE signal restoration
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Splatter
  role: 'Simulates the two-group scRNA-Seq dataset (Dataset 8: 1000 genes, 500 cells,
    161 truly DE genes) with true pre-noise counts and DE labels'
- name: AutoClass
  role: Imputes the dropout-containing matrix (AutoClassImpute with dropout_rate=0.3)
    whose DE t-statistics are benchmarked
  repo: https://github.com/datapplab/AutoClass
- name: SAVER
  role: Control imputation method whose DE recovery is compared against AutoClass
- name: scImpute
  role: Control imputation method whose DE recovery is compared against AutoClass
- name: DCA
  role: Control imputation method (deep count autoencoder) whose DE recovery is compared
    against AutoClass
- name: Python 3
  role: Runtime for T-tests, ROC computation, and CSV output
- name: scikit-learn
  role: ROC curve construction and sensitivity-at-specificity computation (listed
    as 'sklearn' in the AutoClass dependencies)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-29576-y
    title: A universal deep neural network for in-depth cleaning of single-cell RNA-Seq
      data
  source_content_hash: sha256:3ab7d74587f763042b34a3ec1fe8a5c26247efb16e1dc1730faee8c7c8b9f4ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/datapplab/AutoClass
  tools:
  - AutoClass
  - DCA
  - Python
  - Python 3
  - SAVER
  - Splatter
  - numpy
  - scImpute
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# differential-expression-t-test-analysis

## Summary

Benchmark differential expression recovery on scRNA-Seq data by running per-gene two-sample T-tests between cell groups on raw (dropout-containing) and imputed matrices, then summarizing performance as the median t-statistic over truly DE genes and sensitivity at specificity = 0.90 from per-gene t-statistic ROC curves.

## When to use

Use when you have a simulated two-group scRNA-Seq dataset with known truly differentially expressed gene labels and need to quantify how imputation (e.g., AutoClass) restores DE signal lost to dropout.

## When NOT to use

- Data is real (non-simulated) scRNA-Seq with no ground-truth DE gene labels — the median-t-statistic-over-truly-DE-genes metric cannot be computed
- The analysis goal is clustering or batch-effect assessment rather than differential expression recovery
- The count matrix has not been imputed and the question concerns imputation quality rather than DE signal restoration

## Inputs

- True (pre-noise) simulated scRNA-Seq count matrix with DE gene labels (e.g., truecounts.csv)
- Dropout-containing observed count matrix (e.g., Dataset1_raw.csv-style raw counts)
- Imputed count matrices from AutoClass, SAVER, scImpute, and DCA
- Two-group cell condition labels

## Outputs

- Per-gene two-sample T-test t-statistics for each method (raw, true, and each imputed matrix)
- Median t-statistic across the 161 truly DE genes per method
- ROC curves from per-gene t-statistics and sensitivity at specificity = 0.90 per method
- Results table written to dataset8_de_benchmark_results.csv (median DE t-statistics and sensitivities, raw vs imputed)

## How to apply

Simulate (or obtain) a two-group scRNA-Seq count matrix with ground-truth DE gene labels — in the paper, Dataset 8 was generated with Splatter using 1000 genes, 500 cells, and 161 truly differentially expressed genes, keeping both the true (pre-noise) counts and the DE labels. Impute the dropout-containing raw matrix with AutoClass (AutoClassImpute with dropout_rate=0.3) and control methods SAVER, scImpute, and DCA. Run per-gene two-sample T-tests between the two groups on the raw matrix and each imputed matrix (the true counts serve as the reference ceiling). Compute the median t-statistic across the truly DE genes for each method — the paper reports the median dropping from 5.79 (true data) to 2.11 (raw data) and recovering to 5.86 with AutoClass. Additionally build ROC curves from the per-gene t-statistics and compute sensitivity at specificity = 0.90 (~0.61 for AutoClass vs ~0.30 for raw) to verify the reported method ordering.

## Related tools

- **Splatter** (Simulates the two-group scRNA-Seq dataset (Dataset 8: 1000 genes, 500 cells, 161 truly DE genes) with true pre-noise counts and DE labels)
- **AutoClass** (Imputes the dropout-containing matrix (AutoClassImpute with dropout_rate=0.3) whose DE t-statistics are benchmarked) — https://github.com/datapplab/AutoClass
- **SAVER** (Control imputation method whose DE recovery is compared against AutoClass)
- **scImpute** (Control imputation method whose DE recovery is compared against AutoClass)
- **DCA** (Control imputation method (deep count autoencoder) whose DE recovery is compared against AutoClass)
- **Python 3** (Runtime for T-tests, ROC computation, and CSV output)
- **scikit-learn** (ROC curve construction and sensitivity-at-specificity computation (listed as 'sklearn' in the AutoClass dependencies))

## Examples

```
res = AutoClassImpute(X,dropout_rate=0.3)  # then per-gene scipy.stats.ttest_ind between groups; median t over 161 true DE genes
```

## Evaluation signals

- AutoClass attains a median DE-gene t-statistic of approximately 5.86, matching the true data (5.79) and far exceeding raw data (2.11)
- Sensitivity at specificity = 0.90 is ~0.61 for AutoClass versus ~0.30 for raw data, reproducing the reported method ordering (AutoClass > SAVER, scImpute, DCA > raw)
- Output file dataset8_de_benchmark_results.csv exists with columns for median DE t-statistics and sensitivities for each method
- The true (pre-noise) counts produce the highest median t-statistic, serving as the expected ceiling for all imputed methods

## Limitations

- Validation depends on simulated ground truth: DE gene labels and true counts must come from the Splatter simulation, so the approach does not directly transfer to real data
- The article contains no explicit limitations section; Supplementary Tables 1–3 (hyperparameter and simulation parameters) are referenced but not included in the provided text
- No software version numbers are stated for the analysis stack beyond TensorFlow >=2.0 for AutoClass itself

## Evidence

- [other] On Dataset 8, the median t-statistic of truly differentially expressed genes dropped from 5.79 (true data) to 2.11 (raw data) and recovered to 5.86 after AutoClass imputation: [evidence span withheld — anchor status: not_found]
- [results] AutoClassImpute invocation with dropout_rate=0.3: "res = AutoClassImpute(X,dropout_rate=0.3)"
- [abstract] Imputed matrices evaluated via differential expression analysis: "AutoClass outperforms the state-of-art methods in multiple types of scRNA-Seq data analyses, including data recovery, differential expression analysis, clustering analysis, and batch effect removal"
- [intro] Imputation controls used in the benchmark: "deep count autoencoder (DCA)9 assumes negative binomial distribution with or without zero inflation"
- [methods] Runtime and dependency requirements for running AutoClass imputation: "AutoClass runs with Python 3, and you need to have TensorFlow (>=2.0) and a few other pacakges installed"
