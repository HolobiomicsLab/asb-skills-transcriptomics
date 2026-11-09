---
name: multi-classifier-performance-comparison
description: Use when several candidate classifiers must be evaluated on identical
  training/test feature files, and you need to tabulate classification power (accuracy)
  per method, optionally across graded noise levels proxying shallower sequencing
  depth.
when_to_use_negative:
- The goal is predicting continuous evolutionary parameters (theta1, theta2, log10(sigmaSq/(2*alpha)))
  rather than class discrimination — use CLOUDePredictNN/CLOUDePredictXGB/CLOUDePredictRF/CLOUDePredictSVM
  instead of the classify functions
- Only a single classifier is being applied and no head-to-head or noise-robustness
  comparison is required
- Input expression data are raw counts not yet transformed to absolute log10(x+1)
  values in the required 3m-column deletion-event format
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_2269
tools:
- name: CLOUDe
  role: R suite providing the four classifier functions (CLOUDeClassifyNN/XGB/RF/SVM)
    and GenerateTrainingData used to build the shared training/test sets being compared
  repo: https://github.com/anddssan/CLOUDe
- name: keras
  role: Backend for the multi-layer neural network classifier whose hyperparameters
    are tuned via five-fold cross-validation
- name: xgboost
  role: Extreme gradient boosting classifier run with max_depth=4 in the benchmark
- name: ranger
  role: Random forest classifier (CLOUDeClassifyRF) trained on-the-fly during test
    prediction
- name: liquidSVM
  role: Support vector machine classifier (CLOUDeClassifySVM), version 1.2.0, trained
    on-the-fly during test prediction
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae002
    title: Predicting evolutionary targets and parameters of gene deletion from expression
      data
  source_content_hash: sha256:69ad81d167d550b6e4cde400f96abc94a4c98dc0ae367271aabf7e0658bb6499
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/anddssan/CLOUDe
  tools:
  - CLOUDe
  - R
  - keras
  - liquidSVM
  - ranger
  - tensorflow
  - xgboost
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# multi-classifier-performance-comparison

## Summary

Benchmarks multiple machine learning classifiers (neural network, extreme gradient boosting, random forest, SVM) on the same classification task — here CLOUDe's 'Redundant' vs 'Unique' gene-deletion classes — including under controlled noise perturbation, to identify the best-performing architecture.

## When to use

Use when several candidate classifiers must be evaluated on identical training/test feature files, and you need to tabulate classification power (accuracy) per method, optionally across graded noise levels proxying shallower sequencing depth.

## When NOT to use

- The goal is predicting continuous evolutionary parameters (theta1, theta2, log10(sigmaSq/(2*alpha))) rather than class discrimination — use CLOUDePredictNN/CLOUDePredictXGB/CLOUDePredictRF/CLOUDePredictSVM instead of the classify functions
- Only a single classifier is being applied and no head-to-head or noise-robustness comparison is required
- Input expression data are raw counts not yet transformed to absolute log10(x+1) values in the required 3m-column deletion-event format

## Inputs

- Training feature files and fitted model files sharing a common training_prefix (outputs of GenerateTrainingData(), .features, .classes, .nn.num_layers.classifier.hdf5, .xgb.max_depth.classifier_cv, etc.)
- Noise-perturbed test feature files at SD 0.001, 0.01, 0.1, and 1 (space-delimited, p=3m columns; e.g., EmpiricalData.data format with N+1 rows and 3m expression columns for derived/survived/lost genes)
- Known ground-truth 'Redundant'/'Unique' class labels for the test observations

## Outputs

- Per-method classifications files (test_prefix.nn.num_layers.classifications, .xgb.max_depth.classifications, .rf.classifications, .svm.classifications)
- Per-method probabilities files (test_prefix.*.probabilities)
- Tabulated classification power vs. noise level per method (e.g., noise_robustness_results.csv)

## How to apply

Source the CLOUDe R package via source("CLOUDe.R") and point to the training prefix produced by GenerateTrainingData() plus the noise-perturbed test feature files (SD 0.001, 0.01, 0.1, 1). For each noise level, run CLOUDeClassifyNN(training_prefix, testing_prefix, num_layers=2), CLOUDeClassifyXGB(training_prefix, testing_prefix, max_depth=4), CLOUDeClassifyRF(training_prefix, testing_prefix), and CLOUDeClassifySVM(training_prefix, testing_prefix) to produce classifications and probabilities files per test set. Compute classification power as accuracy by comparing predicted classes against the known 'Redundant'/'Unique' labels in each probabilities file. Tabulate power vs. noise SD per method (e.g., noise_robustness_results.csv) and judge success by whether NN and XGB retain high power across all four noise SDs, with the two-hidden-layer NN remaining the best overall performer.

## Related tools

- **CLOUDe** (R suite providing the four classifier functions (CLOUDeClassifyNN/XGB/RF/SVM) and GenerateTrainingData used to build the shared training/test sets being compared) — https://github.com/anddssan/CLOUDe
- **keras** (Backend for the multi-layer neural network classifier whose hyperparameters are tuned via five-fold cross-validation)
- **xgboost** (Extreme gradient boosting classifier run with max_depth=4 in the benchmark)
- **ranger** (Random forest classifier (CLOUDeClassifyRF) trained on-the-fly during test prediction)
- **liquidSVM** (Support vector machine classifier (CLOUDeClassifySVM), version 1.2.0, trained on-the-fly during test prediction)

## Examples

```
source("CLOUDe.R"); CLOUDeClassifyNN(training_prefix, testing_prefix, num_layers=2); CLOUDeClassifyXGB(training_prefix, testing_prefix, max_depth=4)
```

## Evaluation signals

- Classification power (accuracy vs. known 'Redundant'/'Unique' labels) stays high for the NN and XGB across all four noise SDs (0.001, 0.01, 0.1, 1)
- The two-hidden-layer NN (num_layers=2) achieves the best overall classification power among the four methods
- Every classify run emits both a .classifications and a .probabilities file for each method/noise combination
- Classifications files contain exactly one predicted class per deletion event, restricted to the 'Redundant'/'Unique' label set

## Limitations

- SVM and random forest training is performed on-the-fly at test-prediction time, so only NN and XGB can be trained and cross-validated in advance — runtimes differ across methods
- Robustness findings hold for the tested noise scenarios (SD 0.001–1, proxying shallower sequencing depth); behavior outside this range is not established
- liquidSVM must be installed at pinned version 1.2.0 via install_version("liquidSVM", "1.2.0")

## Evidence

- [results] finding: noise robustness: [evidence span withheld — anchor status: not_found]
- [readme] README: classification commands: "CLOUDeClassifyNN(training_prefix, testing_prefix, num_layers)   # neural network
    CLOUDeClassifyXGB(training_prefix, testing_prefix, max_depth)   # extreme gradient boosting"
- [readme] README: output files: "The CLOUDeClassifyNN(), CLOUDeClassifyXGB(), CLOUDeClassifyRF(), and CLOUDeClassifySVM() functions output predicted classes and probabilities"
- [abstract] abstract: architectures: "uses multi-layer neural network, extreme gradient boosting, random forest, and support vector machine architectures"
- [readme] README: on-the-fly training: "Due to their speed, training of the CLOUDe support vector machine and random forest are performed on-the-fly while being applied to test data."
