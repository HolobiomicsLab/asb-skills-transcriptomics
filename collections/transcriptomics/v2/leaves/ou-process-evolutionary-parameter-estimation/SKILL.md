---
name: ou-process-evolutionary-parameter-estimation
description: Use when absolute log10(x+1) expression data for derived, survived, and
  lost genes of deletion events (3m columns, space-delimited) must be converted into
  per-event evolutionary parameter estimates such as theta1, theta2, and log10(sigmaSq/(2*alpha)).
when_to_use_negative:
- The goal is classifying events as 'Redundant' or 'Unique' rather than estimating
  continuous evolutionary parameters — use CLOUDeClassifyNN/CLOUDeClassifyXGB instead
- Expression values are not absolute log10(x+1)-transformed values ordered as derived-then-survived-then-lost
  gene columns, since the 3m-column input format would be misinterpreted
- Only one species' expression data is available, since the OU-process model requires
  expression for derived, survived, and lost genes across two species
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_3170
tools:
- name: CLOUDe
  role: R package implementing OU-process training data generation and neural network,
    xgboost, random forest, and SVM predictors of evolutionary parameters
  repo: https://github.com/anddssan/CLOUDe
- name: keras
  role: Backend used to load and apply pre-trained HDF5 neural network predictors
    (e.g., Training.nn.3.predictor.hdf5)
- name: tensorflow
  role: Backend in which fitted CLOUDe neural network predictor models are stored
    and executed
- name: xgboost
  role: Extreme gradient boosting architecture for CLOUDePredictXGB parameter prediction
    with max_depth in 1–6
- name: ranger
  role: Random forest implementation used by CLOUDePredictRF, trained on-the-fly
- name: liquidSVM 1.2.0
  role: Support vector machine implementation used by CLOUDePredictSVM, trained on-the-fly
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbae002
    title: Predicting evolutionary targets and parameters of gene deletion from expression
      data
  source_content_hash: sha256:7c1589d84d5f4725d5af75087d768459db85ab298d8eeba2c40f14f99670acbd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/anddssan/CLOUDe
  tools:
  - CLOUDe
  - R
  - keras
  - liquidSVM 1.2.0
  - ranger
  - tensorflow
  - xgboost
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# ou-process-evolutionary-parameter-estimation

## Summary

Estimates Ornstein–Uhlenbeck evolutionary parameters (theta1, theta2, and log10(sigmaSq/(2*alpha))) for gene deletion events by applying trained CLOUDe machine learning predictors to expression data from two species.

## When to use

Use when absolute log10(x+1) expression data for derived, survived, and lost genes of deletion events (3m columns, space-delimited) must be converted into per-event evolutionary parameter estimates such as theta1, theta2, and log10(sigmaSq/(2*alpha)).

## When NOT to use

- The goal is classifying events as 'Redundant' or 'Unique' rather than estimating continuous evolutionary parameters — use CLOUDeClassifyNN/CLOUDeClassifyXGB instead
- Expression values are not absolute log10(x+1)-transformed values ordered as derived-then-survived-then-lost gene columns, since the 3m-column input format would be misinterpreted
- Only one species' expression data is available, since the OU-process model requires expression for derived, survived, and lost genes across two species

## Inputs

- Space-delimited expression file (e.g., EmpiricalData.data) with N+1 rows and 3m columns: header 'eD1'...'eDm' 'eS1'...'eSm' 'eL1'...'eLm' giving absolute log10(x+1) expression for derived, survived, and lost genes per condition
- Training feature files (training_prefix.features) and standardization parameters (training_prefix.Y_stdparams) output by GenerateTrainingData()
- Pre-trained predictor model file (e.g., Training.nn.3.predictor.hdf5 in TensorFlow HDF5 format)

## Outputs

- Per-event predicted evolutionary parameters: theta1, theta2, and log10(sigmaSq/(2*alpha)), i.e., 3m values per deletion event (e.g., 18 values at m=6)
- test_prefix.nn.num_layers.predictions file with one row of parameters per deletion event
- Cross-validation outputs training_prefix.nn.num_layers.predictor_cv with optimal hyperparameters

## How to apply

Model expression evolution as an Ornstein–Uhlenbeck process and generate labeled training data with GenerateTrainingData(m, Nk, training_prefix, ...), which writes features, one-hot classes, responses, and standardization parameters (training_prefix.X_stdparams, training_prefix.Y_stdparams). For neural-network-based prediction, tune hyperparameters (lambda, gamma, number of hidden layers) via five-fold cross-validation with PredictorCVnn(), then apply the fitted predictor with CLOUDePredictNN(training_prefix, testing_prefix, num_layers). Alternative architectures include extreme gradient boosting (PredictorCVxgb / CLOUDePredictXGB with max_depth in 1–6), random forest, and SVM, which train on-the-fly. Each deletion event (one row, 3m features with m conditions for derived, survived, and lost genes) receives 3m predicted parameters; for the Drosophila application, the three-hidden-layer predictor processed 100 events from EmpiricalData.data at m=6 tissues.

## Related tools

- **CLOUDe** (R package implementing OU-process training data generation and neural network, xgboost, random forest, and SVM predictors of evolutionary parameters) — https://github.com/anddssan/CLOUDe
- **keras** (Backend used to load and apply pre-trained HDF5 neural network predictors (e.g., Training.nn.3.predictor.hdf5))
- **tensorflow** (Backend in which fitted CLOUDe neural network predictor models are stored and executed)
- **xgboost** (Extreme gradient boosting architecture for CLOUDePredictXGB parameter prediction with max_depth in 1–6)
- **ranger** (Random forest implementation used by CLOUDePredictRF, trained on-the-fly)
- **liquidSVM 1.2.0** (Support vector machine implementation used by CLOUDePredictSVM, trained on-the-fly)

## Examples

```
source("CLOUDe.R"); CLOUDePredictNN("ExampleFiles/Training", "ExampleFiles/EmpiricalData", 3)
```

## Evaluation signals

- Prediction output file contains exactly one row of 3m predicted parameters (e.g., 18 rows-values for m=6) per deletion event, matching N=100 events in EmpiricalData.data
- Predicted log10(sigmaSq/(2*alpha)) values fall within the parameter ranges specified in GenerateTrainingData (logSigmaSqMin–logSigmaSqMax, logAlphaMin–logAlphaMax)
- Five-fold cross-validation selected the number of hidden layers with the smallest validation loss (three hidden layers for the Drosophila application)
- Predicted values are on the standardized scale defined by training_prefix.Y_stdparams and invert consistently to raw OU parameter units
- CLOUDe predictor shows high accuracy and precision in estimating evolutionary parameters as reported for the neural network architecture

## Limitations

- Training is required for the neural network and xgboost predictors because hyperparameter estimation is time-consuming; random forest and SVM train on-the-fly instead
- The liquidSVM dependency must be pinned to version 1.2.0 (install_version("liquidSVM", "1.2.0"))
- The predictor depends on the number of conditions m matching between training and empirical data (e.g., m=6 tissues in the example), and on tensorflow/keras backends being available

## Evidence

- [abstract] CLOUDe models expression evolution as an Ornstein–Uhlenbeck process, and uses multi-layer neural network, extreme gradient boosting, random forest, and support vector machine architectures: "CLOUDe models expression evolution as an Ornstein–Uhlenbeck process, and uses multi-layer neural network, extreme gradient boosting, random forest, and support vector machine architectures"
- [methods] CLOUDe can predict classes and evolutionary parameters for each gene in a test dataset.: "CLOUDe can predict classes ("Redundant" or "Unique") and evolutionary parameters (theta1, theta2, and log10(sigmaSq/(2*alpha))) for each gene in a test dataset."
- [readme] GenerateTrainingData function outputs features, one-hot classes, responses, raw simulated data, and standardization parameters.: "The GenerateTrainingData() function also outputs the p=3m features to the file training_prefix.features, a one-hot encoded matrix of classifications for all training observations to the file"
- [readme] README specifies the space-delimited input format with N+1 rows and 3m columns for derived, survived, and lost gene expression.: "A space-delimited file with N+1 rows and 3m columns, where N is the number of genes and m is the number of conditions (e.g., tissues, ages, etc.) for expression data."
- [results] Application of CLOUDe to empirical Drosophila data supports that deletion targets genes with unique functions.: [evidence span withheld — anchor status: not_found]
