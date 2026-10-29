---
name: simulated-test-data-evaluation
description: Use when benchmarking deletion-target classifiers or parameter predictors
  on simulated expression test sets with known 'Redundant'/'Unique' labels and true
  OU parameters, or when measuring robustness to added noise.
when_to_use_negative:
- Data are empirical deletion events without known labels or true parameter values
  (e.g., the EmpiricalData.data file), where ground truth for power/precision cannot
  be computed
- Expression data are not in the 3m-column derived/survived/lost format output by
  GenerateTrainingData() and would require re-simulation
- Only one architecture needs prediction with no comparison across the four architectures
  or against the LRT
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: CLOUDe
  role: R package supplying GenerateTrainingData and the CLOUDeClassify*/CLOUDePredict*
    functions used to simulate OU data, classify test events, and predict evolutionary
    parameters
  repo: https://github.com/anddssan/CLOUDe
- name: tensorflow
  role: Backend for training and applying the CLOUDe neural network classifier and
    predictor (models saved in .hdf5 format)
- name: xgboost
  role: Powers the extreme gradient boosting classifier/predictor (CLOUDeClassifyXGB/CLOUDePredictXGB
    with max_depth 1–6)
- name: ranger
  role: Random forest engine behind CLOUDeClassifyRF/CLOUDePredictRF, trained on-the-fly
    on test data
- name: liquidSVM
  role: SVM engine behind CLOUDeClassifySVM/CLOUDePredictSVM, pinned to version 1.2.0,
    trained on-the-fly
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae002
    title: Predicting evolutionary targets and parameters of gene deletion from expression
      data
  source_content_hash: sha256:802943dffef4b675ef70609fced5df276a1b4ae81b1face58ec840bb557950cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/anddssan/CLOUDe
  tools:
  - CLOUDe
  - R
  - liquidSVM
  - mvtnorm
  - ranger
  - tensorflow
  - xgboost
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# simulated-test-data-evaluation

## Summary

Evaluates the CLOUDe machine learning suite (NN, XGB, RF, SVM) on balanced simulated test data generated under an Ornstein–Uhlenbeck process, computing classification power and per-event parameter-estimation error distributions for theta1, theta2, and log10(sigmaSq/(2*alpha)).

## When to use

Use when benchmarking deletion-target classifiers or parameter predictors on simulated expression test sets with known 'Redundant'/'Unique' labels and true OU parameters, or when measuring robustness to added noise.

## When NOT to use

- Data are empirical deletion events without known labels or true parameter values (e.g., the EmpiricalData.data file), where ground truth for power/precision cannot be computed
- Expression data are not in the 3m-column derived/survived/lost format output by GenerateTrainingData() and would require re-simulation
- Only one architecture needs prediction with no comparison across the four architectures or against the LRT

## Inputs

- Balanced simulated test-set features from the OU process (space-delimited, N+1 rows x 3m columns, derived/survived/lost gene log10(x+1) expression blocks)
- Fitted NN/XGB model files (training_prefix.nn.num_layers.classifier.hdf5, training_prefix.nn.num_layers.predictor.hdf5, *_cv hyperparameter files)
- Known test-set class labels and true evolutionary parameter values (theta1, theta2, log10(sigmaSq/(2*alpha)))
- Noise-perturbed test feature files at SD 0.001, 0.01, 0.1, and 1

## Outputs

- Per-event class predictions and probabilities (test_prefix.nn.num_layers.classifications/.probabilities, xgb, rf, svm variants)
- Per-event 3m predicted parameters (e.g., *.predictions files from CLOUDePredict*)
- Classification power vs. noise-level summary (noise_robustness_results.csv)
- Per-parameter error distributions stratified by class (parameter_prediction_errors.csv)

## How to apply

Generate balanced simulated test data under the Ornstein–Uhlenbeck process with GenerateTrainingData(), which writes training_prefix.features (p=3m columns), classes, responses, and standardization parameter files. For classification, run CLOUDeClassifyNN/CLOUDeClassifyXGB/CLOUDeClassifyRF/CLOUDeClassifySVM and compute classification power (accuracy) by comparing predicted classes against known labels from the *.probabilities files; for robustness testing, repeat across added-noise SDs of 0.001, 0.01, 0.1, and 1. For parameter estimation, run the four CLOUDePredict* functions to obtain per-event predictions of theta1, theta2, and log10(sigmaSq/(2*alpha)), then compute prediction-minus-true errors stratified by class. Decision points: hyperparameters must match the cross-validation outputs (num_layers in 0–5 for NN, max_depth in 1–6 for XGB), and RF/SVM train on-the-fly so no separate training step is needed. Validation criteria: error distributions centered near zero, theta1/theta2 estimated more precisely than log10(sigmaSq/(2*alpha)), higher precision for the 'redundant' class, and NN achieving the highest precision and retaining high classification power under noise.

## Related tools

- **CLOUDe** (R package supplying GenerateTrainingData and the CLOUDeClassify*/CLOUDePredict* functions used to simulate OU data, classify test events, and predict evolutionary parameters) — https://github.com/anddssan/CLOUDe
- **tensorflow** (Backend for training and applying the CLOUDe neural network classifier and predictor (models saved in .hdf5 format))
- **xgboost** (Powers the extreme gradient boosting classifier/predictor (CLOUDeClassifyXGB/CLOUDePredictXGB with max_depth 1–6))
- **ranger** (Random forest engine behind CLOUDeClassifyRF/CLOUDePredictRF, trained on-the-fly on test data)
- **liquidSVM** (SVM engine behind CLOUDeClassifySVM/CLOUDePredictSVM, pinned to version 1.2.0, trained on-the-fly)

## Examples

```
source("CLOUDe.R"); CLOUDePredictNN(training_prefix, testing_prefix, num_layers=2)
```

## Evaluation signals

- Classification power remains high across all four noise SDs (0.001, 0.01, 0.1, 1) for NN and XGB, with NN achieving the best overall power
- Error distributions (predicted minus true) for all four architectures are centered near zero for theta1, theta2, and log10(sigmaSq/(2*alpha))
- theta1 and theta2 are estimated more precisely than log10(sigmaSq/(2*alpha)) in every architecture
- The 'redundant' class shows higher precision than the 'unique' class
- The neural network exhibits the highest precision across architectures, exceeding the LRT baseline

## Limitations

- RF and SVM models are not pretrained: training happens on-the-fly at test time, so results depend on the test-set training call
- Hyperparameter search is restricted to num_layers in {0,...,5} for NN and max_depth in {1,...,6} for XGB; outside these ranges the CLOUDe functions are not defined
- Precision of log10(sigmaSq/(2*alpha)) is lower than for the expression optima (theta1, theta2), so interpretation should prioritize the latter

## Evidence

- [methods] Training data can be generated with the command: GenerateTrainingData(m, Nk, training_prefix, logThetaMin, logThetaMax, logAlphaMin, logAlphaMax, logSigmaSqMin, logSigmaSqMax, empiricalFile): "Training data can be generated with the command:

    GenerateTrainingData(m, Nk, training_prefix, logThetaMin, logThetaMax, logAlphaMin, logAlphaMax, logSigmaSqMin, logSigmaSqMax, empiricalFile)"
- [readme] The GenerateTrainingData() function also outputs the p=3m features to the file training_prefix.features: "The GenerateTrainingData() function also outputs the p=3m features to the file training_prefix.features, a one-hot encoded matrix of classifications for all training"
- [results] All four CLOUDe architectures display higher parameter-estimation precision than the LRT for both classes, with the NN outshining the others by demonstrating the highest precision.: [evidence span withheld — anchor status: not_found]
- [abstract] CLOUDe boasts high power and accuracy in differentiating between classes, and high accuracy and precision in estimating evolutionary parameters, with optimal performance achieved by its neural: "CLOUDe boasts high power and accuracy in differentiating between classes, and high accuracy and precision in estimating evolutionary parameters, with optimal performance achieved by its neural"
