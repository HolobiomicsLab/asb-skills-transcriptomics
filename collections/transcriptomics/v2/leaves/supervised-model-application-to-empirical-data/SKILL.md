---
name: supervised-model-application-to-empirical-data
description: Use when a pre-trained supervised model (e.g., CLOUDe NN classifier/predictor
  in .hdf5 format) must be applied to an empirical test dataset of deletion events
  formatted as a space-delimited N+1 x 3m expression matrix to recover per-event classes
  and parameters.
when_to_use_negative:
- Input expression data is not in the required 3m-column derived/survived/lost gene
  layout (columns j, m+j, 2m+j per condition) — reformat or the model features will
  be misaligned.
- The goal is to train new models or tune hyperparameters — use ClassifierCVnn/PredictorCVnn
  (five-fold cross-validation) rather than the apply step.
- Only two species' expression data with gene deletion events are not available; CLOUDe
  is designed for expression data from two species and OU-process-simulated training
  data.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3524
tools:
- name: CLOUDe
  role: R package providing CLOUDeClassifyNN and CLOUDePredictNN to apply pre-trained
    neural network classifier and predictor to empirical test data
  repo: https://github.com/anddssan/CLOUDe
- name: R
  role: Runtime environment in which CLOUDe.R is sourced and prediction functions
    are invoked
- name: tensorflow
  role: Backend storing/loading the .hdf5 neural network models in TensorFlow format
- name: keras
  role: Backend used by the CLOUDe neural network classifier and predictor
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbae002
    title: Predicting evolutionary targets and parameters of gene deletion from expression
      data
  source_content_hash: sha256:15356c26d7a0be534bca823c6a513baa3fd31c9c1bc833f370b6227e3ac91aa1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/anddssan/CLOUDe
  tools:
  - CLOUDe
  - R
  - keras
  - tensorflow
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# supervised-model-application-to-empirical-data

## Summary

Apply pre-trained supervised machine learning models (here, CLOUDe neural network classifier and predictor with 2 and 3 hidden layers) to empirical test data to output per-event class labels, class probabilities, and evolutionary parameter estimates, validating outputs against expected findings.

## When to use

Use when a pre-trained supervised model (e.g., CLOUDe NN classifier/predictor in .hdf5 format) must be applied to an empirical test dataset of deletion events formatted as a space-delimited N+1 x 3m expression matrix to recover per-event classes and parameters.

## When NOT to use

- Input expression data is not in the required 3m-column derived/survived/lost gene layout (columns j, m+j, 2m+j per condition) — reformat or the model features will be misaligned.
- The goal is to train new models or tune hyperparameters — use ClassifierCVnn/PredictorCVnn (five-fold cross-validation) rather than the apply step.
- Only two species' expression data with gene deletion events are not available; CLOUDe is designed for expression data from two species and OU-process-simulated training data.

## Inputs

- EmpiricalData.data: space-delimited file with N+1 rows and 3m columns (N=100 Drosophila deletion events, m=6 tissues), header eD1..eLm, absolute log10(x+1) expression values
- Pre-trained classifier model file Training.nn.2.classifier.hdf5 (TensorFlow format)
- Pre-trained predictor model file Training.nn.3.predictor.hdf5 (TensorFlow format)
- Training feature files produced by GenerateTrainingData() (e.g., Training.features) with matching standardized parameter files (X_stdparams, Y_stdparams)

## Outputs

- test_prefix.nn.2.classifications: one predicted class ('Redundant' or 'Unique') per deletion event
- test_prefix.nn.2.probabilities: class probabilities per deletion event
- test_prefix.nn.3.predictions: 3m = 18 predicted evolutionary parameters (theta1, theta2, log10(sigmaSq/(2*alpha))) per event, one row per deletion event

## How to apply

Load the CLOUDe package in R with source("CLOUDe.R"), ensuring the tensorflow and keras backends plus dplyr, mvtnorm, ranger, devtools, liquidSVM (version 1.2.0), and xgboost libraries are installed. Format the empirical input as a space-delimited file with N+1 rows and 3m columns (header "eD1"..."eSm"..."eL1"..."eLm"), where columns j, m+j, and 2m+j hold absolute log10(x+1) expression for derived, survived, and lost genes at condition j. Run CLOUDeClassifyNN(training_prefix, testing_prefix, num_layers=2) using the pre-trained Training.nn.2.classifier.hdf5 model to write per-event class labels ('Redundant' or 'Unique') and probabilities, then run CLOUDePredictNN(training_prefix, testing_prefix, num_layers=3) using Training.nn.3.predictor.hdf5 to write the 3m = 18 predicted parameters (theta1, theta2, log10(sigmaSq/(2*alpha))) per event. The rationale is that the neural network architecture achieved optimal performance in the benchmark, and the 2- and 3-hidden-layer models were selected by five-fold cross-validation as the smallest validation loss configurations.

## Related tools

- **CLOUDe** (R package providing CLOUDeClassifyNN and CLOUDePredictNN to apply pre-trained neural network classifier and predictor to empirical test data) — https://github.com/anddssan/CLOUDe
- **R** (Runtime environment in which CLOUDe.R is sourced and prediction functions are invoked)
- **tensorflow** (Backend storing/loading the .hdf5 neural network models in TensorFlow format)
- **keras** (Backend used by the CLOUDe neural network classifier and predictor)

## Examples

```
source("CLOUDe.R"); CLOUDeClassifyNN("ExampleFiles/Training", "ExampleFiles/EmpiricalData", 2); CLOUDePredictNN("ExampleFiles/Training", "ExampleFiles/EmpiricalData", 3)
```

## Evaluation signals

- Apply to the 100 Drosophila melanogaster/D. pseudoobscura deletion events (Dryad expression data, Assis 2019) and confirm 55 of 100 events are classified as 'Unique'.
- Check that both the .classifications and .probabilities files exist and contain exactly one row per deletion event (100 rows).
- Check that the parameter prediction output contains 3m = 18 columns per event, covering theta1, theta2, and log10(sigmaSq/(2*alpha)) at m=6 tissues.
- Verify the number of hidden layers used matches the pre-trained models: 2 for the classifier, 3 for the predictor.
- Confirm empirical data columns follow the eD1..eDm, eS1..eSm, eL1..eLm ordering before feature extraction.

## Limitations

- Hyperparameter tuning and model training are separate from application; the apply step requires pre-trained .hdf5 model files and matching training feature/standardization files, otherwise feature scaling will be inconsistent.
- Performance claims are grounded in the paper's benchmarks where the neural network achieved optimal performance; other architectures (xgboost, random forest, SVM) may yield different predictions on the same events.
- liquidSVM must be pinned to version 1.2.0 for the full package environment to work.

## Evidence

- [methods] CLOUDe can predict classes ("Redundant" or "Unique") and evolutionary parameters (theta1, theta2, and log10(sigmaSq/(2*alpha))) for each gene in a test dataset.: "CLOUDe can predict classes ("Redundant" or "Unique") and evolutionary parameters (theta1, theta2, and log10(sigmaSq/(2*alpha))) for each gene in a test dataset."
- [readme] space-delimited file with N+1 rows and 3m columns: "A space-delimited file with N+1 rows and 3m columns, where N is the number of genes and m is the number of conditions (e.g., tissues, ages, etc.) for expression data."
- [readme] Optimal number of hidden layers chosen by smallest validation loss: "The optimal number of hidden layers is chosen as the one with the smallest validation loss."
- [abstract] Application of CLOUDe to empirical Drosophila data classifies most deleted genes as unique: "Application of CLOUDe to empirical data from Drosophila suggests that deletion primarily targets genes with unique functions, with further analysis showing these functions to be enriched for protein"
- [results] 55 of the 100 events classified as Unique: [evidence span withheld — anchor status: not_found]
