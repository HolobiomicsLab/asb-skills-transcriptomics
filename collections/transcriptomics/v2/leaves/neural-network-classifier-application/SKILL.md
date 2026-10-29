---
name: neural-network-classifier-application
description: Use when simulated or empirical deletion-event expression data (space-delimited,
  N+1 rows x 3m columns with derived/survived/lost blocks) must be classified as 'Redundant'
  vs 'Unique' by the already-tuned CLOUDe neural network, e.g. when validating power
  under the alternative theta0 scenario.
when_to_use_negative:
- The goal is estimating evolutionary parameters (theta1, theta2, log10(sigmaSq/(2*alpha)))
  rather than classifying 'Redundant' vs 'Unique' — use CLOUDePredictNN instead
- No trained CLOUDe model or testing features exist yet — run GenerateTrainingData
  and ClassifierCVnn first rather than invoking CLOUDeClassifyNN directly
- The input file is not in the CLOUDe 3m-column derived/survived/lost expression format
  (e.g., raw single-species expression or an unstandardized feature table)
tools:
- name: CLOUDe
  role: R package providing CLOUDeClassifyNN for applying the trained neural network
    classifier to test data
  repo: https://github.com/anddssan/CLOUDe
- name: keras
  role: Backend for loading and running the tuned neural network classifier (training_prefix.nn.num_layers.classifier.hdf5)
- name: tensorflow
  role: Framework in which the fitted neural network classifier models are stored
    and executed
- name: xgboost
  role: Extreme gradient boosting classifier (CLOUDeClassifyXGB) run for comparison
    of classification power
- name: ranger
  role: Random forest backend for CLOUDeClassifyRF comparison classification
- name: liquidSVM
  role: Support vector machine backend for CLOUDeClassifySVM comparison classification
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae002
    title: Predicting evolutionary targets and parameters of gene deletion from expression
      data
  source_content_hash: sha256:b5006ac652208a6430f35c2325ba59ee9916382f47c542344e3ec520a3892353
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/anddssan/CLOUDe
  tools:
  - CLOUDe
  - R
  - dplyr
  - keras
  - liquidSVM
  - mvtnorm
  - ranger
  - tensorflow
  - xgboost
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# neural-network-classifier-application

## Summary

Applies the trained CLOUDe neural network classifier (CLOUDeClassifyNN) to test expression data in CLOUDe format to assign each deletion event to the 'Redundant' or 'Unique' class, including under the alternative ancestral-optimum (theta0 differs from theta1/theta2) evolutionary scenario where high power must be reproduced.

## When to use

Use when simulated or empirical deletion-event expression data (space-delimited, N+1 rows x 3m columns with derived/survived/lost blocks) must be classified as 'Redundant' vs 'Unique' by the already-tuned CLOUDe neural network, e.g. when validating power under the alternative theta0 scenario.

## When NOT to use

- The goal is estimating evolutionary parameters (theta1, theta2, log10(sigmaSq/(2*alpha))) rather than classifying 'Redundant' vs 'Unique' — use CLOUDePredictNN instead
- No trained CLOUDe model or testing features exist yet — run GenerateTrainingData and ClassifierCVnn first rather than invoking CLOUDeClassifyNN directly
- The input file is not in the CLOUDe 3m-column derived/survived/lost expression format (e.g., raw single-species expression or an unstandardized feature table)

## Inputs

- Testing expression file in CLOUDe space-delimited format: N+1 rows, 3m columns (eD1..eDm, eS1..eSm, eL1..eLm absolute log10(x+1) expression), with header row (e.g., EmpiricalData.data with N=100, m=6)
- Testing feature file (testing_prefix.features) with p=3m features extracted by GenerateTrainingData-style feature extraction
- Trained neural network model and hyperparameters: training_prefix.nn.num_layers.classifier.hdf5 and training_prefix.nn.num_layers.classifier_cv from ClassifierCVnn
- Training feature/class files output by GenerateTrainingData (training_prefix.features, training_prefix.classes) plus training_prefix.X_stdparams for feature standardization

## Outputs

- test_prefix.nn.num_layers.classifications: predicted class ('Redundant' or 'Unique') per deletion event
- test_prefix.nn.num_layers.probabilities: per-event class probabilities
- Per-method classification power (fraction correctly classified) across CLOUDeClassifyNN/XGB/RF/SVM for the alternative-scenario test set

## How to apply

First ensure testing features (p=3m) exist via the GenerateTrainingData-style feature extraction, producing a testing_prefix.features file alongside the training features. Then call CLOUDeClassifyNN(training_prefix, testing_prefix, num_layers) with the num_layers value (0-5) chosen by the earlier five-fold cross-validation (ClassifierCVnn), which uses the TensorFlow/keras model stored in training_prefix.nn.num_layers.classifier.hdf5. Collect per-deletion-event labels from test_prefix.nn.num_layers.classifications and class probabilities from test_prefix.nn.num_layers.probabilities. For the alternative theta0 scenario, use test data simulated under the OU process where the pre-duplication optimum theta0 differs from theta1 and theta2, and optionally run CLOUDeClassifyXGB, CLOUDeClassifyRF, and CLOUDeClassifySVM for comparison. Evaluate power as the fraction of events assigned the correct class; the paper reports CLOUDe retains high power here (Supplementary Fig. S3B). Note the neural network requires prior training, whereas random forest and support vector machine training happens on-the-fly at classification time.

## Related tools

- **CLOUDe** (R package providing CLOUDeClassifyNN for applying the trained neural network classifier to test data) — https://github.com/anddssan/CLOUDe
- **keras** (Backend for loading and running the tuned neural network classifier (training_prefix.nn.num_layers.classifier.hdf5))
- **tensorflow** (Framework in which the fitted neural network classifier models are stored and executed)
- **xgboost** (Extreme gradient boosting classifier (CLOUDeClassifyXGB) run for comparison of classification power)
- **ranger** (Random forest backend for CLOUDeClassifyRF comparison classification)
- **liquidSVM** (Support vector machine backend for CLOUDeClassifySVM comparison classification)

## Examples

```
CLOUDeClassifyNN(training_prefix, testing_prefix, num_layers)
```

## Evaluation signals

- test_prefix.nn.num_layers.classifications contains one 'Redundant'/'Unique' label per test deletion event, matching the N rows of the input file
- test_prefix.nn.num_layers.probabilities provides per-event probabilities consistent with a two-class prediction
- Classification power (fraction of events assigned the correct simulated class) remains high for the alternative theta0 test set, reproducing Supplementary Fig. S3B
- Neural network results agree strongly with XGB/RF/SVM comparison runs, with the neural network achieving optimal performance as reported in the paper
- Testing feature file has exactly p=3m standardized features consistent with training_prefix.X_stdparams

## Limitations

- The neural network and extreme gradient boosting must be trained beforehand (ClassifierCVnn/ClassifierCVxgb); only random forest and support vector machine train on-the-fly during classification
- num_layers is restricted to 0, 1, 2, 3, 4 or 5, and classification depends on the optimal hyperparameters (lambda, gamma) found via five-fold cross-validation
- Classification was validated on OU-simulated data under the alternative ancestral-optimum scenario; empirical classification power depends on the expression data fitting the modeled process

## Evidence

- [readme] CLOUDeClassifyNN command: "    CLOUDeClassifyNN(training_prefix, testing_prefix, num_layers)   # neural network"
- [readme] class outputs: "The CLOUDeClassifyNN(), CLOUDeClassifyXGB(), CLOUDeClassifyRF(), and CLOUDeClassifySVM() functions output predicted classes and probabilities for each deletion event in the test dataset"
- [methods] classes and parameters: "CLOUDe can predict classes ("Redundant" or "Unique") and evolutionary parameters (theta1, theta2, and log10(sigmaSq/(2*alpha))) for each gene in a test dataset."
- [supplementary] alternative scenario power: [evidence span withheld — anchor status: not_found]
- [readme] on-the-fly training: "Due to their speed, training of the CLOUDe support vector machine and random forest are performed on-the-fly while being applied to test data."
