---
name: machine-learning-regression-across-architectures
description: Use when expression data for deletion events (3m-column absolute log10(x+1)
  format) is available and per-event continuous evolutionary parameters must be estimated
  and compared across multiple machine learning regressors rather than classified.
when_to_use_negative:
- The goal is classifying deleted genes as 'redundant' vs 'unique' rather than estimating
  continuous parameters — use the CLOUDeClassify* functions instead.
- The input is not expression data for deletion events in the 3m-column derived/survived/lost
  gene format (e.g. raw count matrices or unrelated omics features).
- Only a single estimator (e.g. the LRT) is required with no cross-architecture comparison
  of estimation precision.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: CLOUDe
  role: R package providing CLOUDePredictNN, CLOUDePredictXGB, CLOUDePredictRF, and
    CLOUDePredictSVM regression functions and GenerateTrainingData
  repo: https://github.com/anddssan/CLOUDe
- name: tensorflow
  role: Backend for the keras neural network predictor models saved as .hdf5
- name: keras
  role: Fits the multi-layer neural network regressor for theta1, theta2, and log10(sigmaSq/(2*alpha))
- name: xgboost
  role: Extreme gradient boosting predictor with max_depth in {1,...,6}
- name: ranger
  role: Random forest predictor, trained on-the-fly
- name: liquidSVM
  role: Support vector machine predictor (pinned to version 1.2.0), trained on-the-fly
- name: mvtnorm
  role: Supports multivariate normal draws for OU-process training data generation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae002
    title: Predicting evolutionary targets and parameters of gene deletion from expression
      data
  source_content_hash: sha256:684d2b7f46cd96821b6ed80664a823e1d43c2817e1f40c32760addf0aa739c94
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# machine-learning-regression-across-architectures

## Summary

Trains four CLOUDe regression architectures (multi-layer neural network, extreme gradient boosting, random forest, support vector machine) to predict the 3m evolutionary parameters theta1, theta2, and log10(sigmaSq/(2*alpha)) per deletion event, then compares their estimation accuracy and precision against each other and an LRT-based estimate.

## When to use

Use when expression data for deletion events (3m-column absolute log10(x+1) format) is available and per-event continuous evolutionary parameters must be estimated and compared across multiple machine learning regressors rather than classified.

## When NOT to use

- The goal is classifying deleted genes as 'redundant' vs 'unique' rather than estimating continuous parameters — use the CLOUDeClassify* functions instead.
- The input is not expression data for deletion events in the 3m-column derived/survived/lost gene format (e.g. raw count matrices or unrelated omics features).
- Only a single estimator (e.g. the LRT) is required with no cross-architecture comparison of estimation precision.

## Inputs

- Expression data file (space-delimited, N+1 rows x 3m columns, header eD1...eDm eS1...eSm eL1...eLm; columns j, m+j, 2m+j = derived, survived, lost gene at condition j)
- Balanced simulated test data generated under the Ornstein-Uhlenbeck process with true parameter values
- Training feature/response files (training_prefix.features, training_prefix.responses, training_prefix.X_stdparams, training_prefix.Y_stdparams) from GenerateTrainingData

## Outputs

- test_prefix.nn.num_layers.predictions, test_prefix.xgb.max_depth.predictions, test_prefix.rf.predictions, test_prefix.svm.predictions (3m predicted parameters per deletion event)
- Per-event estimation errors (predicted minus true) per architecture and parameter
- parameter_prediction_errors.csv summarizing error distributions stratified by redundant vs unique class

## How to apply

Generate OU-process training data with GenerateTrainingData(m, Nk, training_prefix, logThetaMin, logThetaMax, logAlphaMin, logAlphaMax, logSigmaSqMin, logSigmaSqMax, empiricalFile), which also emits the p=3m feature file and response matrix of expression optima. For the neural network and extreme gradient boosting, first run PredictorCVnn or PredictorCVxgb five-fold cross-validation to select lambda, gamma (and eta for XGB), and the optimal num_layers in {0,...,5} or max_depth in {1,...,6} by smallest validation loss; random forest (ranger) and SVM (liquidSVM 1.2.0) train on-the-fly. Then run CLOUDePredictNN, CLOUDePredictXGB, CLOUDePredictRF, and CLOUDePredictSVM on the test prefix to emit each *.predictions file containing theta1, theta2, and log10(sigmaSq/(2*alpha)) for every deletion event. Compute per-event estimation errors (predicted minus true parameter) stratified by 'redundant' vs 'unique' class and compare distributions across architectures and against an LRT-based estimate.

## Related tools

- **CLOUDe** (R package providing CLOUDePredictNN, CLOUDePredictXGB, CLOUDePredictRF, and CLOUDePredictSVM regression functions and GenerateTrainingData) — https://github.com/anddssan/CLOUDe
- **tensorflow** (Backend for the keras neural network predictor models saved as .hdf5)
- **keras** (Fits the multi-layer neural network regressor for theta1, theta2, and log10(sigmaSq/(2*alpha)))
- **xgboost** (Extreme gradient boosting predictor with max_depth in {1,...,6})
- **ranger** (Random forest predictor, trained on-the-fly)
- **liquidSVM** (Support vector machine predictor (pinned to version 1.2.0), trained on-the-fly)
- **mvtnorm** (Supports multivariate normal draws for OU-process training data generation)

## Examples

```
source("CLOUDe.R"); CLOUDePredictNN(training_prefix, testing_prefix, num_layers)
```

## Evaluation signals

- Estimation error distributions (predicted minus true parameter) are centered near zero for all four architectures.
- Precision is higher for theta1 and theta2 than for log10(sigmaSq/(2*alpha)).
- The 'redundant' class shows higher parameter-estimation precision than the 'unique' class.
- All four CLOUDe architectures outperform the LRT in precision, with the neural network achieving the highest precision for the stationary variance.
- Each predictor emits its expected *.predictions file (test_prefix.nn.num_layers.predictions, etc.).

## Limitations

- Hyperparameter tuning of the neural network and extreme gradient boosting predictors via five-fold cross-validation is time-consuming; SVM and random forest are instead trained on-the-fly.
- liquidSVM must be pinned to version 1.2.0 for compatibility.
- The RNN/XGB predictors require hyperparameter ranges to be specified (e.g. num_layers in {0,1,2,3,4,5}, max_depth in {1,2,3,4,5,6}, log(lambda) and gamma grids), and the optimal setting is chosen by smallest validation loss.

## Evidence

- [methods] CLOUDe can predict classes ("Redundant" or "Unique") and evolutionary parameters (theta1, theta2, and log10(sigmaSq/(2*alpha))) for each gene in a test dataset.: "CLOUDe can predict classes ("Redundant" or "Unique") and evolutionary parameters (theta1, theta2, and log10(sigmaSq/(2*alpha))) for each gene in a test dataset."
- [abstract] uses multi-layer neural network, extreme gradient boosting, random forest, and support vector machine architectures: "CLOUDe models expression evolution as an Ornstein–Uhlenbeck process, and uses multi-layer neural network, extreme gradient boosting, random forest, and support vector machine architectures"
- [readme] we perform five-fold cross-validation to identify optimal hyperparameters for the predictor with maximum depth of trees of max_depth: "we perform five-fold cross-validation to identify optimal hyperparameters for the predictor with maximum depth of trees of max_depth"
- [results] All four CLOUDe architectures display higher parameter-estimation precision than the LRT for both classes, with the NN outshining the others by demonstrating the highest precision.: [evidence span withheld — anchor status: not_found]
