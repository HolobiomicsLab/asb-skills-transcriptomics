---
name: roc-sensitivity-specificity-evaluation
description: Use when benchmarking DE detection on simulated scRNA-Seq data with known
  DE gene labels and dropout noise, and you need ROC-based sensitivity at fixed specificity
  (0.90) to rank imputation methods.
when_to_use_negative:
- No ground-truth DE gene labels exist (real datasets without spike-in or simulation
  truth), so ROC analysis cannot be validated
- The data matrix is not count-based two-group scRNA-Seq data; per-gene two-sample
  T-tests between groups are not applicable
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_3308
tools:
- name: AutoClass
  role: Imputation method whose imputed matrix is evaluated by ROC/sensitivity analysis;
    expected to attain sensitivity ~0.61 at specificity 0.90 vs ~0.30 for raw data
  repo: https://github.com/datapplab/AutoClass
- name: SAVER
  role: Control imputation method compared in the ROC sensitivity benchmark
  repo: https://github.com/mohuangx/SAVER
- name: scImpute
  role: Control imputation method compared in the ROC sensitivity benchmark
  repo: https://github.com/Vivianstats/scImpute
- name: DCA
  role: Control imputation method (deep count autoencoder) compared in the ROC sensitivity
    benchmark
  repo: https://github.com/theislab/dca
- name: Splatter
  role: Simulates the scRNA-Seq dataset (Dataset 8) with known true counts and DE
    gene labels used as ROC ground truth
- name: scikit-learn
  role: Python library used for ROC curve construction and sensitivity/specificity
    computation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-29576-y
    title: A universal deep neural network for in-depth cleaning of single-cell RNA-Seq
      data
  source_content_hash: sha256:9c934add70cb4c27955496d424aeb5a914e8bb88d3d67d6a189438780b3edbf5
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# ROC-sensitivity-specificity-evaluation

## Summary

Evaluates differential expression detection performance on simulated scRNA-Seq data by building per-gene ROC curves from two-sample T-test t-statistics and reporting sensitivity at a fixed specificity of 0.90, comparing raw dropout-containing data against imputed matrices (AutoClass, SAVER, scImpute, DCA).

## When to use

Use when benchmarking DE detection on simulated scRNA-Seq data with known DE gene labels and dropout noise, and you need ROC-based sensitivity at fixed specificity (0.90) to rank imputation methods.

## When NOT to use

- No ground-truth DE gene labels exist (real datasets without spike-in or simulation truth), so ROC analysis cannot be validated
- The data matrix is not count-based two-group scRNA-Seq data; per-gene two-sample T-tests between groups are not applicable

## Inputs

- Raw dropout-containing scRNA-Seq count matrix (e.g., Dataset 8: 1000 genes x 500 cells, two groups)
- True (pre-noise) count matrix from Splatter simulation
- Vector of truly differentially expressed gene labels (161 DE genes)

## Outputs

- Per-gene two-sample T-test t-statistics for each method (raw, AutoClass, SAVER, scImpute, DCA)
- ROC curves built from per-gene t-statistics per method
- Sensitivity at specificity = 0.90 per method
- Median t-statistic across truly DE genes per method (table written to dataset8_de_benchmark_results.csv)

## How to apply

After simulating data (e.g., Splatter Dataset 8 with 1000 genes, 500 cells, two groups, 161 truly DE genes), run per-gene two-sample T-tests between the two groups on the raw dropout matrix and on each imputed matrix (AutoClass via AutoClassImpute with dropout_rate=0.3, plus SAVER, scImpute, DCA). Build ROC curves from the per-gene t-statistics using the known DE gene labels as ground truth, and compute sensitivity at specificity = 0.90 for each method. Also compute the median t-statistic across the 161 truly DE genes to quantify signal recovery (true data 5.79, raw data 2.11, AutoClass 5.86). Rationale: t-statistics serve as a continuous ranking score for ROC analysis, and the specificity=0.90 operating point gives a false-positive-controlled sensitivity that separates methods (AutoClass ~0.61 vs raw ~0.30).

## Related tools

- **AutoClass** (Imputation method whose imputed matrix is evaluated by ROC/sensitivity analysis; expected to attain sensitivity ~0.61 at specificity 0.90 vs ~0.30 for raw data) — https://github.com/datapplab/AutoClass
- **SAVER** (Control imputation method compared in the ROC sensitivity benchmark) — https://github.com/mohuangx/SAVER
- **scImpute** (Control imputation method compared in the ROC sensitivity benchmark) — https://github.com/Vivianstats/scImpute
- **DCA** (Control imputation method (deep count autoencoder) compared in the ROC sensitivity benchmark) — https://github.com/theislab/dca
- **Splatter** (Simulates the scRNA-Seq dataset (Dataset 8) with known true counts and DE gene labels used as ROC ground truth)
- **scikit-learn** (Python library used for ROC curve construction and sensitivity/specificity computation)

## Examples

```
from sklearn.metrics import roc_curve; fpr, tpr, _ = roc_curve(de_labels, t_stats); sens_at_spec90 = tpr[fpr <= 0.10][-1]
```

## Evaluation signals

- AutoClass attains a median DE-gene t-statistic of approximately 5.86, recovering the true-data value of 5.79 versus 2.11 for raw dropout data
- Sensitivity at specificity = 0.90 is ~0.61 for AutoClass versus ~0.30 for raw data, with AutoClass exceeding SAVER, scImpute, and DCA
- ROC curves are constructed from exactly 161 truly DE genes as positives and the remaining genes as negatives
- Results table reproduces the reported method ordering when written to dataset8_de_benchmark_results.csv

## Limitations

- The benchmark relies on simulated data (Splatter) where ground-truth DE labels are known; performance on real data without truth labels cannot be verified this way
- Existing imputation methods focus on dropouts and have strong distribution assumptions (e.g., DCA assumes zero-inflated negative binomial, SAVER assumes negative binomial), which may limit generalization of the ranking
- Sensitivity is only reported at one fixed operating point (specificity = 0.90); full-PRC/AUC behavior across thresholds is not captured by this single metric

## Evidence

- [other] On Dataset 8, the median t-statistic of truly differentially expressed genes dropped from 5.79 (true data) to 2.11 (raw data) and recovered to 5.86 after AutoClass imputation: [evidence span withheld — anchor status: not_found]
- [results] Impute data with AutoClassImpute using dropout_rate=0.3: "res = AutoClassImpute(X,dropout_rate=0.3)"
- [abstract] AutoClass outperforms state-of-the-art methods in DE analysis: "AutoClass outperforms the state-of-art methods in multiple types of scRNA-Seq data analyses, including data recovery, differential expression analysis, clustering analysis, and batch effect removal"
- [intro] DCA and other methods assume specific distributions: "deep count autoencoder (DCA)9 assumes negative binomial distribution with or without zero inflation"
