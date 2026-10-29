---
name: k-fold-cross-validation-metrics-computation
description: Use when a labeled sequence-derived feature table (e.g., All_Features_Dataset.csv)
  must be benchmarked with repeated 5-fold cross-validation and a multi-metric performance
  table (Accuracy/Precision/Sensitivity/Specificity/F1/MCC) is required.
when_to_use_negative:
- The data has a dedicated independent test set and unbiased final evaluation is the
  goal — use independent-set testing (as in the paper's Table 6) rather than only
  cross-validation.
- Comparing classifiers or feature sets without fixed cross-validation folds and repeats
  — uncontrolled fold assignment makes metric tables non-reproducible.
- The features have not yet been generated (Dataset.csv has not been processed by
  Feature Generation.ipynb) — run feature extraction first.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0099
tools:
- name: scikit-learn
  role: Provides the Logistic Regression, SVM, and GNB classifiers and cross-validation
    machinery used to produce the metrics
  repo: https://github.com/zahid6454/iRNA5hmC-PS
- name: pandas
  role: Loads All_Features_Dataset.csv / Dataset.csv and assembles feature matrices
    and label vectors per combination
  repo: https://github.com/zahid6454/iRNA5hmC-PS
- name: numpy
  role: Numerical computation of metric means and standard deviations across cross-validation
    repeats
  repo: https://github.com/zahid6454/iRNA5hmC-PS
- name: Jupyter
  role: Execution environment for Model.ipynb, which runs cross-validation and prints
    the results tables and AUROC/AUPR curves
  repo: https://github.com/zahid6454/iRNA5hmC-PS
- name: Anaconda
  role: Distribution in which Jupyter and the required pip packages are installed
  repo: https://github.com/zahid6454/iRNA5hmC-PS
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2020.10.032
    title: Accurate prediction of RNA 5-hydroxymethylcytosine modification by utilizing
      novel position-specific gapped k-mer descriptors
  source_content_hash: sha256:2ee17651c0dec04a22a665e14aae005881d68c84fc1beb58f84b8d463c4e2f80
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zahid6454/iRNA5hmC-PS
  tools:
  - Anaconda
  - Jupyter
  - Python
  - iRNA5hmC-PS
  - numpy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# k-fold-cross-validation-metrics-computation

## Summary

Evaluates a scikit-learn classifier (e.g., Logistic Regression on RNA 5hmC position-specific features) under repeated k-fold cross-validation and reports Accuracy, Precision, Sensitivity, Specificity, F1-Score, and MCC with standard deviations, as done in iRNA5hmC-PS.

## When to use

Use when a labeled sequence-derived feature table (e.g., All_Features_Dataset.csv) must be benchmarked with repeated 5-fold cross-validation and a multi-metric performance table (Accuracy/Precision/Sensitivity/Specificity/F1/MCC) is required.

## When NOT to use

- The data has a dedicated independent test set and unbiased final evaluation is the goal — use independent-set testing (as in the paper's Table 6) rather than only cross-validation.
- Comparing classifiers or feature sets without fixed cross-validation folds and repeats — uncontrolled fold assignment makes metric tables non-reproducible.
- The features have not yet been generated (Dataset.csv has not been processed by Feature Generation.ipynb) — run feature extraction first.

## Inputs

- All_Features_Dataset.csv (feature matrix generated from Dataset.csv by Feature Generation.ipynb)
- Label vector with 5hmC site positives/negatives from the benchmark dataset
- Feature-mode combination specification (A = Ps-k-mer, B = PsM(G)D, C = PsD(G)M, D = PsM(G)M(G)M, plus GC-content)

## Outputs

- Per-combination cross-validation metrics table (e.g., table3_ablation_results.csv) with mean and standard deviation of Accuracy, Precision, Sensitivity, Specificity, F1-Score, and MCC
- AUROC and AUPR curves (as produced by Model.ipynb)

## How to apply

Load the engineered feature matrix (All_Features_Dataset.csv, produced by Feature Generation.ipynb from Dataset.csv) with pandas and split it into feature matrix X and label vector y. Train the chosen scikit-learn classifier (Logistic Regression with default hyperparameters was best in this study) under repeated k-fold cross-validation — the article uses 5-fold cross-validation, repeated 50 times in the results tables, and the ablation task specifies 10-times 5-fold. From the pooled out-of-fold predictions compute Accuracy, Precision, Sensitivity, Specificity, F1-Score, and MCC, reporting the mean and standard deviation across repeats. Apply the group-wise feature importance calculation strategy to select the discriminative feature subset before evaluation when comparing feature-mode combinations. Validate against the paper's reference values: LR on 5-fold CV achieves 78.3% accuracy, 78.0% precision, 80.0% sensitivity, 79.5% specificity, 0.78 F1-score, and 0.56 MCC.

## Related tools

- **scikit-learn** (Provides the Logistic Regression, SVM, and GNB classifiers and cross-validation machinery used to produce the metrics) — https://github.com/zahid6454/iRNA5hmC-PS
- **pandas** (Loads All_Features_Dataset.csv / Dataset.csv and assembles feature matrices and label vectors per combination) — https://github.com/zahid6454/iRNA5hmC-PS
- **numpy** (Numerical computation of metric means and standard deviations across cross-validation repeats) — https://github.com/zahid6454/iRNA5hmC-PS
- **Jupyter** (Execution environment for Model.ipynb, which runs cross-validation and prints the results tables and AUROC/AUPR curves) — https://github.com/zahid6454/iRNA5hmC-PS
- **Anaconda** (Distribution in which Jupyter and the required pip packages are installed) — https://github.com/zahid6454/iRNA5hmC-PS

## Examples

```
import pandas as pd; df = pd.read_csv('All_Features_Dataset.csv'); # run Model.ipynb (Run All) in Jupyter to obtain CV metric tables and AUROC/AUPR curves
```

## Evaluation signals

- The A+B+C+D feature-mode combination with group-wise feature selection reproduces the paper's reference values: Accuracy 78.3, Precision 78.0, Sensitivity 80.0, Specificity 79.5, F1 0.78, MCC 0.56 within expected cross-validation variation
- All six metrics are reported with both mean and standard deviation across cross-validation repeats (e.g., SVM 75.5 (0.41), 75.1 (0.50), ...)
- Model.ipynb, run to completion, prints cross-validation and independent-test results in tables plus AUROC and AUPR curves
- The 15-row ablation table contains exactly the 15 non-empty combinations of the four feature modes A, B, C, D together with GC-content

## Limitations

- Cross-validation estimates vary run to run; reproduced metrics will not match the paper exactly and must be compared within expected CV variation
- The paper reports that position-specific features do not work well in conjunction with existing frequency-based counterparts, deferring combined use to a future feature-subspace ensemble approach
- The reported 78.3% performance (12.8% better than previous studies) is specific to this 41-nt RNA 5hmC benchmark dataset and LR with these features; it may not transfer to other datasets or modifications

## Evidence

- [results] LR achieves 78.3%, 78.0%, 80.0%, 79.5%, 0.78, 0.56 in terms of Accuracy, Precision, Sensitivity, Specificity, F1-Score, and MCC, respectively: "LR achieves 78.3%, 78.0%, 80.0%, 79.5%, 0.78, 0.56 in terms of Accuracy, Precision, Sensitivity, Specificity, F1-Score, and MCC, respectively"
- [abstract] We also use a group-wise feature importance calculation strategy to select a small subset of features containing maximum discriminative information: "We also use a group-wise feature importance calculation strategy to select a small subset of features containing maximum discriminative information"
- [readme] When "All_Features_Dataset.csv" file is available execute the "Model.ipynb" file just like "Feature Generation.ipynb" file was executed earlier.: "When "All_Features_Dataset.csv" file is available execute the "Model.ipynb" file just like "Feature Generation.ipynb" file was executed earlier."
- [readme] Cross Validation Results of the three classifiers along with Logistic Regression (Proposed as Best Method) will be shown in the output cell: "Cross Validation Results of the three classifiers along with Logistic Regression (Proposed as Best Method) will be shown in the output cell"
- [discussion] our proposed position-specific features do not work well in conjunction with existing frequency-based counterparts: "our proposed position-specific features do not work well in conjunction with existing frequency-based counterparts"
