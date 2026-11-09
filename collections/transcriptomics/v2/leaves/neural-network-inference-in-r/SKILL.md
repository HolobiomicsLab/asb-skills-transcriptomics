---
name: neural-network-inference-in-r
description: Use when expression data for two-species gene deletion events (space-delimited
  3m-column EmpiricalData.data format) must be classified and have OU evolutionary
  parameters predicted via pre-trained multi-layer neural network models.
when_to_use_negative:
- When expression data are not formatted as the 3m-column derived/survived/lost per-event
  file required by CLOUDe (e.g., raw count matrices or single-gene expression tables)
- When a different algorithm family is required (random forest or SVM train on-the-fly
  and skip .hdf5 model files; xgboost uses ClassifierCVxgb/PredictorCVxgb instead)
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: CLOUDe
  role: R package providing CLOUDeClassifyNN and CLOUDePredictNN functions with pre-trained
    .hdf5 models
  repo: https://github.com/anddssan/CLOUDe
- name: R
  role: runtime environment in which CLOUDe.R is sourced and NN functions are executed
- name: tensorflow
  role: backend on which the neural network models are fitted and stored in TensorFlow
    format
- name: keras
  role: backend used to load and apply the pre-trained classifier and predictor .hdf5
    models
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbae002
    title: Predicting evolutionary targets and parameters of gene deletion from expression
      data
  source_content_hash: sha256:b60995887799ece89f08d51dd776b0509cea4739f1449e64562f9cebb61eb26e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/anddssan/CLOUDe
  tools:
  - CLOUDe
  - R
  - keras
  - tensorflow
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# neural-network-inference-in-r

## Summary

Apply the CLOUDe neural network architecture in R (tensorflow/keras backend) to classify gene deletion events as 'Redundant' or 'Unique' and to predict evolutionary parameters (theta1, theta2, log10(sigmaSq/(2*alpha))) from Ornstein–Uhlenbeck-derived expression features.

## When to use

Use when expression data for two-species gene deletion events (space-delimited 3m-column EmpiricalData.data format) must be classified and have OU evolutionary parameters predicted via pre-trained multi-layer neural network models.

## When NOT to use

- When expression data are not formatted as the 3m-column derived/survived/lost per-event file required by CLOUDe (e.g., raw count matrices or single-gene expression tables)
- When a different algorithm family is required (random forest or SVM train on-the-fly and skip .hdf5 model files; xgboost uses ClassifierCVxgb/PredictorCVxgb instead)

## Inputs

- EmpiricalData.data: space-delimited expression file with N+1 rows and 3m columns (derived/survived/lost gene log10(x+1) expression values, header eD1..eDm eS1..eSm eL1..eLm)
- Training prefix files from GenerateTrainingData (e.g., Training.features, Training.classes, Training.responses)
- Pre-trained Keras models: Training.nn.2.classifier.hdf5 and Training.nn.3.predictor.hdf5

## Outputs

- test_prefix.nn.num_layers.classifications: predicted class ('Redundant' or 'Unique') per deletion event
- test_prefix.nn.num_layers.probabilities: class probabilities per deletion event
- Predicted evolutionary parameters (theta1, theta2, log10(sigmaSq/(2*alpha)); 3m=18 values) per event

## How to apply

Load the CLOUDe package in R with source("CLOUDe.R") after installing dplyr, mvtnorm, tensorflow, keras, ranger, devtools, liquidSVM 1.2.0, and xgboost. Prepare an input file with N+1 rows and 3m columns (e.g., N=100 deletion events at m=6 tissues), where columns j, m+j, and 2m+j hold derived, survived, and lost gene expression. Run CLOUDeClassifyNN(training_prefix, testing_prefix, num_layers) with the pre-trained two-hidden-layer classifier (Training.nn.2.classifier.hdf5) to output class labels and probabilities, and CLOUDePredictNN(training_prefix, testing_prefix, num_layers) with the three-hidden-layer predictor to output 3m=18 evolutionary parameters per event. The neural network hyperparameters (num_layers, lambda, gamma, batch size, epochs) are optimized by five-fold cross-validation, selecting the architecture with the smallest validation loss; rationale is that CLOUDe's neural network achieved optimal performance among its machine learning architectures. Validate that output files contain one row per deletion event and that expected class distributions (e.g., 55/100 'Unique' for the Drosophila benchmark) are reproduced.

## Related tools

- **CLOUDe** (R package providing CLOUDeClassifyNN and CLOUDePredictNN functions with pre-trained .hdf5 models) — https://github.com/anddssan/CLOUDe
- **R** (runtime environment in which CLOUDe.R is sourced and NN functions are executed)
- **tensorflow** (backend on which the neural network models are fitted and stored in TensorFlow format)
- **keras** (backend used to load and apply the pre-trained classifier and predictor .hdf5 models)

## Examples

```
source("CLOUDe.R"); CLOUDeClassifyNN("ExampleFiles/Training", "ExampleFiles/EmpiricalData", 2); CLOUDePredictNN("ExampleFiles/Training", "ExampleFiles/EmpiricalData", 3)
```

## Evaluation signals

- Applying the NN to the 100 Drosophila deletion events should classify 55 of 100 events as 'Unique'
- The predictor must output 3m=18 evolutionary parameter values per deletion event for m=6 tissues
- CLOUDeClassifyNN writes both a .classifications and a .probabilities file, each with one row per test deletion event
- The chosen num_layers (classifier 2, predictor 3) must correspond to the smallest validation loss from five-fold cross-validation
- The source("CLOUDe.R") call must succeed with tensorflow/keras backends available before any predictions run

## Limitations

- Neural network and xgboost models require a separate, time-consuming hyperparameter tuning and training step; only SVM and random forest run on-the-fly without pre-training
- liquidSVM must be installed at the pinned version 1.2.0 for full CLOUDe functionality
- Applying the model to data with a different number of conditions m than the training data (m=6 in the example) will break the 3m-feature expectation

## Evidence

- [abstract] CLOUDe models expression evolution as an Ornstein–Uhlenbeck process, and uses multi-layer neural network, extreme gradient boosting, random forest, and support vector machine architectures: "CLOUDe models expression evolution as an Ornstein–Uhlenbeck process, and uses multi-layer neural network, extreme gradient boosting, random forest, and support vector machine architectures"
- [methods] Apply NN to 100 Drosophila events: [evidence span withheld — anchor status: not_found]
- [readme] NN load and dependencies: "Before you are able to use the CLOUDe R package, you will need to have the dplyr, mvtnorm, tensorflow, keras, ranger, devtools, liquidSVM, and xgboost libraries installed in your R environment."
- [readme] classification outputs: "The CLOUDeClassifyNN(), CLOUDeClassifyXGB(), CLOUDeClassifyRF(), and CLOUDeClassifySVM() functions output predicted classes and probabilities for each deletion event in the test dataset"
- [results] 55/100 Unique validation: [evidence span withheld — anchor status: not_found]
- [readme] input format: "A space-delimited file with N+1 rows and 3m columns, where N is the number of genes and m is the number of conditions (e.g., tissues, ages, etc.) for expression data."
