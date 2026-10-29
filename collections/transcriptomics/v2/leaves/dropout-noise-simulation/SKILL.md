---
name: dropout-noise-simulation
description: Use when you need ground-truth scRNA-Seq data with known DE genes to
  quantify how much dropout degrades differential expression (e.g. median t-statistic
  collapse) and how well imputation methods such as AutoClass, SAVER, scImpute, or
  DCA recover it.
when_to_use_negative:
- Real experimental scRNA-Seq data without ground-truth DE labels — the median-t-statistic
  and sensitivity-at-specificity-0.90 validation requires known true counts and known
  DE genes from simulation
- Data where the noise type of interest is not dropout (e.g. batch effects or cell-cycle
  variation), since the raw-vs-imputed DE recovery benchmark targets dropout artifacts
- When only downstream accuracy of clustering (purity score) is the target evaluation,
  not differential expression recovery
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Splatter
  role: Simulates scRNA-Seq datasets (e.g. 1000 genes, 500 cells, two or six groups)
    yielding paired true and dropout-containing counts plus DE gene labels
- name: AutoClass
  role: Imputation method under test; AutoClassImpute with dropout_rate=0.3 produces
    the imputed matrix evaluated for DE recovery
  repo: https://github.com/datapplab/AutoClass
- name: SAVER
  role: Control imputation method benchmarked against AutoClass on the dropout-containing
    matrix
- name: scImpute
  role: Control imputation method benchmarked against AutoClass on the dropout-containing
    matrix
- name: DCA
  role: Control imputation method benchmarked against AutoClass on the dropout-containing
    matrix
  repo: theislab/dca
- name: numpy
  role: Python dependency for the T-test/ROC evaluation pipeline run alongside TensorFlow
    and scikit-learn
- name: scikit-learn
  role: Python dependency used for ROC/evaluation metrics in the benchmark pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-29576-y
    title: A universal deep neural network for in-depth cleaning of single-cell RNA-Seq
      data
  source_content_hash: sha256:b1f6fbe35b087ebf9b14685a210e17a53735401f64a5470c311e85a5a4028f2a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/datapplab/AutoClass
  tools:
  - AutoClass
  - DCA
  - Python
  - SAVER
  - Splatter
  - numpy
  - scImpute
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# dropout-noise-simulation

## Summary

Simulate scRNA-Seq count matrices with controlled dropout noise (e.g. Splatter simulations with paired true pre-noise counts and DE gene labels), then benchmark imputation methods by differential expression recovery — the core validation strategy the AutoClass paper uses to show imputation restores DE signal lost to dropouts.

## When to use

Use when you need ground-truth scRNA-Seq data with known DE genes to quantify how much dropout degrades differential expression (e.g. median t-statistic collapse) and how well imputation methods such as AutoClass, SAVER, scImpute, or DCA recover it.

## When NOT to use

- Real experimental scRNA-Seq data without ground-truth DE labels — the median-t-statistic and sensitivity-at-specificity-0.90 validation requires known true counts and known DE genes from simulation
- Data where the noise type of interest is not dropout (e.g. batch effects or cell-cycle variation), since the raw-vs-imputed DE recovery benchmark targets dropout artifacts
- When only downstream accuracy of clustering (purity score) is the target evaluation, not differential expression recovery

## Inputs

- Simulated scRNA-Seq count matrix with dropout (e.g. Dataset1_raw.csv-style raw counts)
- True (pre-noise) simulated counts (e.g. Dataset1_true.csv / truecounts.csv)
- Ground-truth labels: truly differentially expressed gene list (e.g. 161 DE genes in Dataset 8) and group assignments for the two simulated cell groups
- Splatter simulation parameters (gene count, cell count, number of groups, DE probability)

## Outputs

- Per-gene two-sample T-statistics for raw and each imputed matrix
- Median t-statistic across truly DE genes per method (e.g. raw 2.11 vs AutoClass 5.86 on Dataset 8)
- ROC curves built from per-gene t-statistics and sensitivity at specificity = 0.90 per method
- Benchmark results table (e.g. dataset8_de_benchmark_results.csv) comparing raw vs imputed methods

## How to apply

Simulate a scRNA-Seq dataset with Splatter (the AutoClass paper uses 1000 genes and 500 cells; Dataset 8 uses two groups with 161 truly DE genes), retaining both the true (pre-noise) counts and the DE gene labels. Add dropout noise to the true counts to produce the observed count matrix that serves as the raw input, then impute it with AutoClass (AutoClassImpute with dropout_rate=0.3) and with control methods SAVER, scImpute, and DCA. Run per-gene two-sample T-tests between the two groups on raw and each imputed matrix, compute the median t-statistic across the truly DE genes, build ROC curves from per-gene t-statistics, and record sensitivity at specificity = 0.90. Rationale: dropout compresses true DE signal (paper reports median t-statistic falling from 5.79 on true data to 2.11 on raw data), so DE recovery is a sensitive functional readout of imputation quality.

## Related tools

- **Splatter** (Simulates scRNA-Seq datasets (e.g. 1000 genes, 500 cells, two or six groups) yielding paired true and dropout-containing counts plus DE gene labels)
- **AutoClass** (Imputation method under test; AutoClassImpute with dropout_rate=0.3 produces the imputed matrix evaluated for DE recovery) — https://github.com/datapplab/AutoClass
- **SAVER** (Control imputation method benchmarked against AutoClass on the dropout-containing matrix)
- **scImpute** (Control imputation method benchmarked against AutoClass on the dropout-containing matrix)
- **DCA** (Control imputation method benchmarked against AutoClass on the dropout-containing matrix) — theislab/dca
- **numpy** (Python dependency for the T-test/ROC evaluation pipeline run alongside TensorFlow and scikit-learn)
- **scikit-learn** (Python dependency used for ROC/evaluation metrics in the benchmark pipeline)

## Examples

```
res = AutoClassImpute(X, dropout_rate=0.3)
```

## Evaluation signals

- AutoClass should recover a median DE-gene t-statistic of approximately 5.86 on Dataset 8, matching the true data's 5.79 and exceeding raw data's 2.11 and all controls
- Sensitivity at specificity = 0.90 should be ~0.61 for AutoClass versus ~0.30 for raw data, reproducing the reported method ordering (AutoClass > SAVER/scImpute/DCA > raw)
- The raw (dropout-containing) matrix must show substantial DE signal loss relative to the true counts (5.79 → 2.11) before imputation, confirming the dropout simulation was effective
- Number of truly DE genes analyzed (161 for Dataset 8) matches the simulation parameters, and both true counts and DE labels are retained alongside the noisy counts

## Limitations

- The DE-recovery readout is tied to simulated data with known DE labels; conclusions may not transfer to real data where ground truth is unavailable
- Imputation hyperparameters such as dropout_rate=0.3 must match the simulated dropout level; the paper notes robustness is limited to key hyperparameters (bottleneck size, pre-clustering number, classifier weight)
- The paper's discussion is entirely affirmative and no explicit limitations section is included in the provided text

## Evidence

- [intro] we simulated a scRNA-Seq Dataset 1 (see “Methods” and Supplementary Table 2) using Splatter13 with 1000 genes and 500 cells in six groups: "we simulated a scRNA-Seq Dataset 1 (see “Methods” and Supplementary Table 2) using Splatter13 with 1000 genes and 500 cells in six groups"
- [results] res = AutoClassImpute(X,dropout_rate=0.3): "res = AutoClassImpute(X,dropout_rate=0.3)"
- [results] X = expr.values #Splash generated scRNA-seq data with dropout: "X = expr.values #Splash generated scRNA-seq data with dropout"
- [readme] AutoClass is free of distribution assumptions, and effectively cleans a wide range of noises and artifacts.: "AutoClass is free of distribution assumptions, and effectively cleans a wide range of noises and artifacts."
- [abstract] Existing denoising and imputation methods largely focus on a single type of noise (i.e., dropouts) and have strong distribution assumptions: "Existing denoising and imputation methods largely focus on a single type of noise (i.e., dropouts) and have strong distribution assumptions"
