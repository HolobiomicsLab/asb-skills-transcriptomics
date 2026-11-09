---
name: gene-expression-feature-classification
description: Use when the goal is to predict evolutionary targets of gene deletion
  events ('Redundant' vs 'Unique' classes) from two-species expression data, or to
  benchmark classifier power across four added-noise SD levels (0.001, 0.01, 0.1,
  1).
when_to_use_negative:
- The task is predicting continuous evolutionary parameters (theta1, theta2, log10(sigmaSq/(2*alpha)))
  rather than discrete 'Redundant'/'Unique' classes — use CLOUDePredictNN/CLOUDePredictXGB/CLOUDePredictRF/CLOUDePredictSVM
  instead.
- Expression data are not for deletion events in two species (no derived/survived/lost
  gene triplet structure), so the 3m-column input format and OU-based class labels
  do not apply.
- SVM/RF training workflow is needed as a separate offline step — the README states
  RF and SVM are trained on-the-fly, so only NN and XGB require prior CV-based training.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CLOUDe
  role: R suite providing CLOUDeClassifyNN/XGB/RF/SVM for classifying deletion events
    as 'Redundant' or 'Unique' from expression features
  repo: https://github.com/anddssan/CLOUDe
- name: tensorflow
  role: Backend for the CLOUDe neural network classifier models saved in TensorFlow
    format (.hdf5)
- name: keras
  role: Neural network implementation used by CLOUDeClassifyNN with num_layers hidden
    layers
- name: xgboost
  role: Extreme gradient boosting classifier invoked as CLOUDeClassifyXGB(training_prefix,
    testing_prefix, max_depth)
- name: ranger
  role: Random forest classifier invoked as CLOUDeClassifyRF(training_prefix, testing_prefix)
- name: liquidSVM 1.2.0
  role: Support vector machine classifier invoked as CLOUDeClassifySVM(training_prefix,
    testing_prefix)
- name: R
  role: Environment in which the CLOUDe.R script is sourced and all classification
    commands are run
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae002
    title: Predicting evolutionary targets and parameters of gene deletion from expression
      data
  source_content_hash: sha256:0920c41564b4d2efb7cc8f11b2c7112914d80638efcca8cf9308c0bd9a7f7279
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/anddssan/CLOUDe
  tools:
  - CLOUDe
  - R
  - keras
  - liquidSVM
  - liquidSVM 1.2.0
  - ranger
  - tensorflow
  - xgboost
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# gene-expression-feature-classification

## Summary

Classifies deleted genes as 'Redundant' or 'Unique' from expression features (3m absolute log10(x+1) expression values for derived, survived, and lost genes) using the CLOUDe machine learning suite, including stress-testing classifier power under added Gaussian noise that proxies shallower sequencing depth.

## When to use

Use when the goal is to predict evolutionary targets of gene deletion events ('Redundant' vs 'Unique' classes) from two-species expression data, or to benchmark classifier power across four added-noise SD levels (0.001, 0.01, 0.1, 1).

## When NOT to use

- The task is predicting continuous evolutionary parameters (theta1, theta2, log10(sigmaSq/(2*alpha))) rather than discrete 'Redundant'/'Unique' classes — use CLOUDePredictNN/CLOUDePredictXGB/CLOUDePredictRF/CLOUDePredictSVM instead.
- Expression data are not for deletion events in two species (no derived/survived/lost gene triplet structure), so the 3m-column input format and OU-based class labels do not apply.
- SVM/RF training workflow is needed as a separate offline step — the README states RF and SVM are trained on-the-fly, so only NN and XGB require prior CV-based training.

## Inputs

- Space-delimited expression input file (N+1 rows, 3m columns) formatted per the README with header "eD1" ... "eDm" "eS1" ... "eSm" "eL1" ... "eLm" (e.g., EmpiricalData.data for N=100 deletion events at m=6 tissues)
- Training files output by GenerateTrainingData(): training_prefix.features, training_prefix.classes, training_prefix.data
- Noise-perturbed test feature files at SD 0.001, 0.01, 0.1, and 1
- Fitted NN model files training_prefix.nn.num_layers.classifier.hdf5 (TensorFlow format) for the neural network classifier

## Outputs

- Per-noise-level classifications files: test_prefix.nn.num_layers.classifications, test_prefix.xgb.max_depth.classifications, test_prefix.rf.classifications, test_prefix.svm.classifications
- Per-noise-level probabilities files: test_prefix.nn.num_layers.probabilities, test_prefix.xgb.max_depth.probabilities, test_prefix.rf.probabilities, test_prefix.svm.probabilities
- Tabulated power-vs-noise summary (e.g., noise_robustness_results.csv) comparing NN, XGB, RF, and SVM classification power across SD 0.001/0.01/0.1/1

## How to apply

Source the CLOUDe package in R via source("CLOUDe.R") and generate noise-perturbed test feature files from the OU-simulated training data produced by GenerateTrainingData(). For each noise SD (0.001, 0.01, 0.1, 1), run CLOUDeClassifyNN(training_prefix, testing_prefix, num_layers=2) with the two-hidden-layer neural network, plus CLOUDeClassifyXGB(max_depth=4), CLOUDeClassifyRF, and CLOUDeClassifySVM, each writing classifications and probabilities files per test set. Compute classification power (accuracy) from the probabilities files by comparing predicted classes against known 'Redundant'/'Unique' labels. Tabulate power vs. noise SD per method and export the robustness pattern (e.g., noise_robustness_results.csv). The rationale is that noise added to test expression data proxies shallower sequencing depth, so comparing power across SD levels reveals how much depth degradation each architecture can tolerate.

## Related tools

- **CLOUDe** (R suite providing CLOUDeClassifyNN/XGB/RF/SVM for classifying deletion events as 'Redundant' or 'Unique' from expression features) — https://github.com/anddssan/CLOUDe
- **tensorflow** (Backend for the CLOUDe neural network classifier models saved in TensorFlow format (.hdf5))
- **keras** (Neural network implementation used by CLOUDeClassifyNN with num_layers hidden layers)
- **xgboost** (Extreme gradient boosting classifier invoked as CLOUDeClassifyXGB(training_prefix, testing_prefix, max_depth))
- **ranger** (Random forest classifier invoked as CLOUDeClassifyRF(training_prefix, testing_prefix))
- **liquidSVM 1.2.0** (Support vector machine classifier invoked as CLOUDeClassifySVM(training_prefix, testing_prefix))
- **R** (Environment in which the CLOUDe.R script is sourced and all classification commands are run)

## Examples

```
source("CLOUDe.R"); CLOUDeClassifyNN(training_prefix, testing_prefix, num_layers=2); CLOUDeClassifyXGB(training_prefix, testing_prefix, max_depth=4); CLOUDeClassifyRF(training_prefix, testing_prefix); CLOUDeClassifySVM(training_prefix, testing_prefix)
```

## Evaluation signals

- Classification power (accuracy vs. known 'Redundant'/'Unique' labels) stays high across all four noise SDs (0.001, 0.01, 0.1, 1) for both NN and XGB.
- NN achieves the best overall classification power among the four architectures (NN, XGB, RF, SVM), matching the paper's reported result.
- Each classifier invocation produces both a .classifications and a .probabilities file per noise level, named per the README convention (e.g., test_prefix.nn.num_layers.classifications).
- Training files must come from GenerateTrainingData(); verify training_prefix.features/classes exist and the NN model .hdf5 file is present before running classifiers.
- Simulated noise perturbation should span four orders of magnitude of SD (0.001 to 1) so shallow-sequencing-depth degradation is meaningfully probed.

## Limitations

- Only the NN and XGB classifiers require prior hyperparameter tuning/training; RF and SVM are trained on-the-fly, so pre-training steps for them are unnecessary and skipped.
- num_layers is restricted to {0,1,2,3,4,5} and max_depth to {1,2,3,4,5,6}; the benchmark workflow uses num_layers=2 and max_depth=4.
- Neural network hyperparameter tuning is time-consuming (five-fold cross-validation over lambda and gamma), so tuning the predictor is done separately from the classifier workflow.
- Robustness was assessed for expression noise added to test data only; this does not by itself guarantee robustness to other batch effects or platform differences.

## Evidence

- [results] CLOUDe retains high classification power under noise: [evidence span withheld — anchor status: not_found]
- [methods] Classifier commands: "CLOUDeClassifyNN(training_prefix, testing_prefix, num_layers)   # neural network"
- [readme] Training data generation: "The GenerateTrainingData() function also outputs the p=3m features to the file training_prefix.features, a one-hot encoded matrix of classifications for all training observations to the file"
- [readme] Installation dependencies: "you will need to have the dplyr, mvtnorm, tensorflow, keras, ranger, devtools, liquidSVM, and xgboost libraries installed in your R environment"
- [abstract] High power finding: "CLOUDe boasts high power and accuracy in differentiating between classes, and high accuracy and precision in estimating evolutionary parameters, with optimal performance achieved by its neural"
