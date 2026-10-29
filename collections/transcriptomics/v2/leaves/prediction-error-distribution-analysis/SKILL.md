---
name: prediction-error-distribution-analysis
description: Use when you need to evaluate the accuracy and precision of regression-based
  parameter estimators by comparing predicted-minus-true error distributions across
  multiple model architectures on simulated data with known ground truth.
when_to_use_negative:
- Applied to empirical data where true parameter values are unknown — per-event error
  (predicted minus true) cannot be computed without simulated ground truth.
- Evaluating class prediction (redundant vs unique classification accuracy/power)
  rather than continuous parameter estimation — use the CLOUDeClassify* functions
  instead.
- The test data are unbalanced across classes, since precision comparisons by class
  require the balanced design described in the study.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0203
tools:
- name: CLOUDe
  role: R package providing the GenerateTrainingData() and CLOUDePredict* functions
    used to simulate OU test data and produce parameter predictions
  repo: https://github.com/anddssan/CLOUDe
- name: R
  role: Environment in which the CLOUDe package is sourced (source("CLOUDe.R")) and
    error distributions are computed
  repo: https://github.com/anddssan/CLOUDe
- name: tensorflow
  role: Backend for the CLOUDe neural network predictor (CLOUDePredictNN)
- name: keras
  role: Interface used to fit and apply the neural network predictor models stored
    in .hdf5 format
- name: ranger
  role: Random forest engine behind CLOUDePredictRF, trained on-the-fly for parameter
    prediction
- name: xgboost
  role: Extreme gradient boosting engine behind CLOUDePredictXGB with max_depth tuned
    by five-fold cross-validation
- name: liquidSVM
  role: Support vector machine engine behind CLOUDePredictSVM (version 1.2.0)
- name: mvtnorm
  role: Required R library for the OU-process simulation underlying the training/test
    data
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae002
    title: Predicting evolutionary targets and parameters of gene deletion from expression
      data
  source_content_hash: sha256:77ae06ae595123ac9de8dfa353787083cd3f5cc790872aa2050f2dc3a99f117f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/anddssan/CLOUDe
  tools:
  - CLOUDe
  - R
  - keras
  - liquidSVM
  - mvtnorm
  - ranger
  - tensorflow
  - xgboost
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# prediction-error-distribution-analysis

## Summary

Quantifies how precisely CLOUDe's four machine learning architectures (NN, XGB, RF, SVM) and an LRT baseline estimate the OU evolutionary parameters theta1, theta2, and log10(sigmaSq/(2*alpha)) by computing per-event prediction errors on balanced simulated test data, stratified by 'redundant' vs 'unique' class.

## When to use

Use when you need to evaluate the accuracy and precision of regression-based parameter estimators by comparing predicted-minus-true error distributions across multiple model architectures on simulated data with known ground truth.

## When NOT to use

- Applied to empirical data where true parameter values are unknown — per-event error (predicted minus true) cannot be computed without simulated ground truth.
- Evaluating class prediction (redundant vs unique classification accuracy/power) rather than continuous parameter estimation — use the CLOUDeClassify* functions instead.
- The test data are unbalanced across classes, since precision comparisons by class require the balanced design described in the study.

## Inputs

- Balanced simulated test dataset generated under the Ornstein-Uhlenbeck process via GenerateTrainingData(m, Nk, training_prefix, ...) with true theta1, theta2, and log10(sigmaSq/(2*alpha)) values
- Training feature files (training_prefix.features) outputted by GenerateTrainingData()
- Input expression file (space-delimited, N+1 rows x 3m columns, e.g. EmpiricalData.data with absolute log10(x+1) expression values for derived/survived/lost genes)

## Outputs

- *.predictions files (e.g. test_prefix.svm.predictions) with 3m predicted parameters (theta1, theta2, log10(sigmaSq/(2*alpha))) per deletion event for each architecture
- parameter_prediction_errors.csv with per-event estimation errors stratified by 'redundant' vs 'unique' class
- Summary comparison of error-distribution centering and precision across NN, XGB, RF, SVM, and LRT

## How to apply

Load the CLOUDe R package with source("CLOUDe.R") and obtain the balanced simulated test set generated under the Ornstein-Uhlenbeck process via GenerateTrainingData(), which yields p=3m expression features plus true theta1, theta2, and log10(sigmaSq/(2*alpha)) values. Run each predictor on the test data — CLOUDePredictNN(training_prefix, testing_prefix, num_layers), CLOUDePredictXGB(training_prefix, testing_prefix, max_depth), CLOUDePredictRF(training_prefix, testing_prefix) using ranger, and CLOUDePredictSVM(training_prefix, testing_prefix) using liquidSVM — each emitting a *.predictions file with 3m parameters per deletion event; compute an LRT-based estimate for comparison. For each architecture and parameter, compute per-event errors (predicted minus true), tabulate the error distributions, and stratify by 'redundant' vs 'unique' class. The rationale is that centered, narrow error distributions indicate unbiased, high-precision estimation, enabling a like-for-like comparison of machine learning architectures against the likelihood-based LRT on identical simulated data.

## Related tools

- **CLOUDe** (R package providing the GenerateTrainingData() and CLOUDePredict* functions used to simulate OU test data and produce parameter predictions) — https://github.com/anddssan/CLOUDe
- **R** (Environment in which the CLOUDe package is sourced (source("CLOUDe.R")) and error distributions are computed) — https://github.com/anddssan/CLOUDe
- **tensorflow** (Backend for the CLOUDe neural network predictor (CLOUDePredictNN))
- **keras** (Interface used to fit and apply the neural network predictor models stored in .hdf5 format)
- **ranger** (Random forest engine behind CLOUDePredictRF, trained on-the-fly for parameter prediction)
- **xgboost** (Extreme gradient boosting engine behind CLOUDePredictXGB with max_depth tuned by five-fold cross-validation)
- **liquidSVM** (Support vector machine engine behind CLOUDePredictSVM (version 1.2.0))
- **mvtnorm** (Required R library for the OU-process simulation underlying the training/test data)

## Examples

```
source("CLOUDe.R"); CLOUDePredictNN(training_prefix, testing_prefix, num_layers); CLOUDePredictRF(training_prefix, testing_prefix)
```

## Evaluation signals

- Error distributions (predicted minus true) for all four CLOUDe architectures are centered near zero for theta1, theta2, and log10(sigmaSq/(2*alpha)).
- Precision (spread of errors) is higher for theta1 and theta2 than for log10(sigmaSq/(2*alpha)) across architectures.
- Precision is higher for the 'redundant' class than the 'unique' class.
- The neural network (CLOUDePredictNN) exhibits the highest precision for the stationary variance parameter relative to XGB, RF, SVM, and the LRT.
- All four CLOUDe architectures outperform the LRT baseline in parameter-estimation precision, matching the paper's reported performance patterns.

## Limitations

- Error quantification is only possible on simulated data with known true parameters; empirical applications (e.g. Drosophila EmpiricalData.data) cannot be validated this way.
- The NN and XGB predictors require prior five-fold cross-validation (PredictorCVnn/PredictorCVxgb) to select num_layers, lambda, gamma, eta, and max_depth; RF and SVM train on-the-fly.
- liquidSVM must be installed at the pinned version 1.2.0, and the tensorflow/keras models are output in .hdf5 format, so environment mismatches can change predictions.

## Evidence

- [results] All four CLOUDe architectures display higher parameter-estimation precision than the LRT for both classes, with the NN outshining the others by demonstrating the highest precision.: [evidence span withheld — anchor status: not_found]
- [methods] CLOUDe can predict classes and evolutionary parameters for each gene in a test dataset: "CLOUDe can predict classes ("Redundant" or "Unique") and evolutionary parameters (theta1, theta2, and log10(sigmaSq/(2*alpha))) for each gene in a test dataset."
- [readme] To predict evolutionary parameters, the following commands can be used depending on the chosen algorithm: "To predict evolutionary parameters, the following commands can be used depending on the chosen algorithm:"
- [abstract] CLOUDe boasts high power and accuracy in differentiating between classes, and high accuracy and precision in estimating evolutionary parameters: "CLOUDe boasts high power and accuracy in differentiating between classes, and high accuracy and precision in estimating evolutionary parameters, with optimal performance achieved by its neural"
