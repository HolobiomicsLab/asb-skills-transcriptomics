---
name: keras-neural-network-classifier-training
description: Use when an expression matrix of gene deletion events (eD/eS/eL features
  from GenerateTrainingData output or the EmpiricalData.data-style format) must be
  classified into 'Redundant' or 'Unique' classes with the neural-network architecture.
when_to_use_negative:
- The goal is to predict evolutionary parameters (theta1, theta2, log10(sigmaSq/(2*alpha)))
  rather than classes — use PredictorCVnn instead of ClassifierCVnn
- The chosen architecture is random forest or support vector machine, which CLOUDe
  trains on-the-fly during test prediction, so this pre-training step can be skipped
- The input is already a fitted model or test predictions and no retraining of the
  classifier is needed
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: CLOUDe
  role: Provides ClassifierCVnn() and the CLOUDe.R script that implement training-data
    generation, cross-validated neural-network classifier tuning, and .hdf5 model
    output
  repo: https://github.com/anddssan/CLOUDe
- name: keras
  role: R library required in the environment; used to build, train, and reload the
    fitted multi-layer neural-network classifier
- name: tensorflow
  role: Backend/format in which the fitted neural-network classifier model is saved
    (training_prefix.nn.num_layers.classifier.hdf5)
- name: devtools
  role: Used to install the pinned liquidSVM 1.2.0 dependency required by the CLOUDe
    environment
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbae002
    title: Predicting evolutionary targets and parameters of gene deletion from expression
      data
  source_content_hash: sha256:b510595cc844a49deb6edc21fb02ea4abf68271ddab622cbc0f24f99de7ac076
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/anddssan/CLOUDe
  tools:
  - CLOUDe
  - R
  - devtools
  - keras
  - tensorflow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# keras-neural-network-classifier-training

## Summary

Trains the CLOUDe multi-layer neural network classifier that discriminates 'Redundant' vs 'Unique' gene deletion classes from 3m expression features, using keras/tensorflow with five-fold cross-validated tuning of lambda (elastic-net regularization), gamma, batch size, and epochs.

## When to use

Use when an expression matrix of gene deletion events (eD/eS/eL features from GenerateTrainingData output or the EmpiricalData.data-style format) must be classified into 'Redundant' or 'Unique' classes with the neural-network architecture.

## When NOT to use

- The goal is to predict evolutionary parameters (theta1, theta2, log10(sigmaSq/(2*alpha))) rather than classes — use PredictorCVnn instead of ClassifierCVnn
- The chosen architecture is random forest or support vector machine, which CLOUDe trains on-the-fly during test prediction, so this pre-training step can be skipped
- The input is already a fitted model or test predictions and no retraining of the classifier is needed

## Inputs

- Training feature matrix (training_prefix.features, p=3m columns of absolute log10(x+1) expression values for derived/survived/lost genes)
- One-hot encoded class labels (training_prefix.classes, 'Redundant' vs 'Unique')
- Input expression file in the CLOUDe format (space-delimited, N+1 rows x 3m columns with header eD1..eDm eS1..eSm eL1..eLm), e.g. ExampleFiles/EmpiricalData.data
- Hyperparameter settings: num_layers (0-5), batchsize, num_epochs, log-lambda range and count, gamma range and count

## Outputs

- Fitted keras/TensorFlow classifier model saved to training_prefix.nn.num_layers.classifier.hdf5
- Optimal cross-validated hyperparameters written to training_prefix.nn.num_layers.classifier_cv
- Downstream: predicted classes and class probabilities per deletion event (test_prefix.nn.num_layers.classifications and .probabilities)

## How to apply

First source("CLOUDe.R") and generate a balanced training set with GenerateTrainingData(m, Nk, training_prefix, ...), which writes the p=3m feature matrix to training_prefix.features and one-hot class labels to training_prefix.classes. Then call ClassifierCVnn(num_layers, batchsize, num_epochs, log_lambda_min, log_lambda_max, num_lambda, gamma_min, gamma_max, num_gamma, training_prefix), which performs five-fold cross-validation over log10(lambda) evenly spaced in [log_lambda_min, log_lambda_max] and gamma evenly spaced in [gamma_min, gamma_max]; num_layers ranges over {0,1,2,3,4,5} and the optimal number of hidden layers is the one with the smallest validation loss. Conditional on the optimal lambda and gamma, CLOUDe fits the keras model, writing optimal hyperparameters to training_prefix.nn.num_layers.classifier_cv and the fitted model in TensorFlow format to training_prefix.nn.num_layers.classifier.hdf5. The .hdf5 model can then be loaded and wrapped (e.g., as a class-probability predictor for iml Shapley analysis). Note that only the neural network and extreme gradient boosting require this pre-training step; random forest and support vector machine are trained on-the-fly during test prediction.

## Related tools

- **CLOUDe** (Provides ClassifierCVnn() and the CLOUDe.R script that implement training-data generation, cross-validated neural-network classifier tuning, and .hdf5 model output) — https://github.com/anddssan/CLOUDe
- **keras** (R library required in the environment; used to build, train, and reload the fitted multi-layer neural-network classifier)
- **tensorflow** (Backend/format in which the fitted neural-network classifier model is saved (training_prefix.nn.num_layers.classifier.hdf5))
- **devtools** (Used to install the pinned liquidSVM 1.2.0 dependency required by the CLOUDe environment)

## Examples

```
ClassifierCVnn(2, 50, 50, -3, 3, 7, 0, 1, 3, "ExampleFiles/Training")
```

## Evaluation signals

- GenerateTrainingData output files exist: training_prefix.features, training_prefix.classes, training_prefix.responses, training_prefix.data, training_prefix.X_stdparams, training_prefix.Y_stdparams
- training_prefix.nn.num_layers.classifier_cv records the optimal lambda and gamma chosen by five-fold cross-validation
- training_prefix.nn.num_layers.classifier.hdf5 loads successfully with keras/tensorflow and reproduces class-probability predictions on held-out training features
- Training set is balanced (Nk observations per class) and the feature matrix has p=3m columns ordered eD, eS, eL
- Model selection minimizes validation loss across num_layers in {0,1,2,3,4,5}; predictions fall in {'Redundant','Unique'} with associated probabilities

## Limitations

- Hyperparameter tuning of the neural network is time-consuming, which is why CLOUDe separates cross-validated tuning from final model fitting
- liquidSVM must be installed at the pinned version 1.2.0 (install_version("liquidSVM", "1.2.0")) for the CLOUDe environment to work
- Only num_layers in {0,1,2,3,4,5} and the specified log-lambda/gamma grids are searchable; the optimal architecture is chosen solely by smallest validation loss
- The .hdf5 classifier is specific to the trained num_layers setting, and class predictions assume the two classes 'Redundant' and 'Unique'

## Evidence

- [methods] ClassifierCVnn(num_layers, batchsize, num_epochs, log_lambda_min, log_lambda_max, num_lambda, gamma_min, gamma_max, num_gamma, training_prefix): "ClassifierCVnn(num_layers, batchsize, num_epochs, log_lambda_min, log_lambda_max, num_lambda, gamma_min, gamma_max, num_gamma, training_prefix)"
- [methods] GenerateTrainingData signature: "GenerateTrainingData(m, Nk, training_prefix, logThetaMin, logThetaMax, logAlphaMin, logAlphaMax, logSigmaSqMin, logSigmaSqMax, empiricalFile)"
- [readme] Five-fold CV over hyperparameters: "we perform five-fold cross-validation to identify optimal hyperparameters for the predictor with num_layer layers"
- [readme] Model output in TensorFlow format: "and the fitted neural network models in TensorFlow format to the files training_prefix.nn.num_layers.classifier.hdf5"
- [intro] Required R libraries: "you will need to have the dplyr, mvtnorm, tensorflow, keras, ranger, devtools, liquidSVM, and xgboost libraries installed"
- [abstract] CLOUDe architectures: "uses multi-layer neural network, extreme gradient boosting, random forest, and support vector machine architectures"
