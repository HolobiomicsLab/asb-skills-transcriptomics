---
name: neural-network-hyperparameter-cross-validation
description: Use when training the CLOUDe neural network on Ornstein–Uhlenbeck-simulated
  training data (p=3m expression features) and you must select optimal lambda, gamma,
  and number of hidden layers by five-fold cross-validation before classification.
when_to_use_negative:
- You are training the CLOUDe random forest or support vector machine — the README
  states these are trained on-the-fly when applied to test data, so cross-validation-based
  pretraining is unnecessary
- You want to predict evolutionary parameters (theta1, theta2, log10(sigmaSq/(2*alpha)))
  rather than classes — use PredictorCVnn instead of ClassifierCVnn
- Your input is already a fitted model or empirical predictions rather than training
  data from the OU-simulation GenerateTrainingData pipeline
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0092
tools:
- name: CLOUDe
  role: Provides ClassifierCVnn, which performs the five-fold cross-validation over
    num_layers, lambda, and gamma and fits the neural network classifier
  repo: https://github.com/anddssan/CLOUDe
- name: keras
  role: Backend library used to build and train the multi-layer neural network during
    cross-validation and final fitting
- name: tensorflow
  role: Backend under keras; fitted classifier models are saved in TensorFlow HDF5
    (.hdf5) format
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae002
    title: Predicting evolutionary targets and parameters of gene deletion from expression
      data
  source_content_hash: sha256:e5148c04b9497468d7cf5277a0ad552f98a05ef51e577eda1f36c4f096859685
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# neural-network-hyperparameter-cross-validation

## Summary

Tunes the CLOUDe neural network classifier via five-fold cross-validation over the regularization parameter lambda, elastic net parameter gamma, and number of hidden layers (num_layers 0–5) before fitting the final Keras/TensorFlow model on OU-simulated gene-deletion expression data.

## When to use

Use when training the CLOUDe neural network on Ornstein–Uhlenbeck-simulated training data (p=3m expression features) and you must select optimal lambda, gamma, and number of hidden layers by five-fold cross-validation before classification.

## When NOT to use

- You are training the CLOUDe random forest or support vector machine — the README states these are trained on-the-fly when applied to test data, so cross-validation-based pretraining is unnecessary
- You want to predict evolutionary parameters (theta1, theta2, log10(sigmaSq/(2*alpha))) rather than classes — use PredictorCVnn instead of ClassifierCVnn
- Your input is already a fitted model or empirical predictions rather than training data from the OU-simulation GenerateTrainingData pipeline

## Inputs

- training_prefix.features file (p=3m expression features output by GenerateTrainingData, e.g., 18 columns for m=6 conditions)
- training_prefix.classes file (one-hot encoded Redundant/Unique classifications for all training observations)
- training_prefix.X_stdparams (feature means and standard deviations used for standardization)
- EmpiricalData.data-style input file (space-delimited, N+1 rows x 3m columns of log10(x+1) expression values for derived, survived, and lost genes)

## Outputs

- training_prefix.nn.num_layers.classifier_cv file containing the optimal hyperparameters (lambda, gamma) chosen through cross-validation for each num_layers
- training_prefix.nn.num_layers.classifier.hdf5 file containing the fitted neural network classifier model in TensorFlow format
- Downstream test_prefix.nn.num_layers.classifications and test_prefix.nn.num_layers.probabilities files after applying CLOUDeClassifyNN

## How to apply

After generating OU training data with GenerateTrainingData (e.g., m=6, Nk=20000 per class, bounded by empirical expression ranges), call ClassifierCVnn to run five-fold cross-validation over num_layers in {0,...,5}, log10(lambda) evenly spaced in [-3, 3] (e.g., 7 points), and gamma evenly spaced in [0, 1] (e.g., 3 points), with batch size 50 and 50 epochs. The optimal lambda and gamma are identified for each candidate network depth, the model is refit conditional on those optima, and the number of hidden layers is chosen as the one with the smallest validation loss. Fit the final classifier and save it in TensorFlow HDF5 format; for the unbalanced-training setting, subsequent CLOUDeClassifyNN application on a balanced test set should reach ~96.65% power (redundant recall) and ~96.35% accuracy, and ~97.90% accuracy on a balanced 2000-observation simulated test set.

## Related tools

- **CLOUDe** (Provides ClassifierCVnn, which performs the five-fold cross-validation over num_layers, lambda, and gamma and fits the neural network classifier) — https://github.com/anddssan/CLOUDe
- **keras** (Backend library used to build and train the multi-layer neural network during cross-validation and final fitting)
- **tensorflow** (Backend under keras; fitted classifier models are saved in TensorFlow HDF5 (.hdf5) format)

## Examples

```
ClassifierCVnn(0:5, 50, 50, -3, 3, 7, 0, 1, 3, training_prefix)
```

## Evaluation signals

- The classifier_cv output file exists and reports optimal hyperparameters per num_layers, with the chosen network depth being the one with the smallest validation loss
- The fitted model file training_prefix.nn.num_layers.classifier.hdf5 exists and loads in keras/tensorflow
- Applying CLOUDeClassifyNN to a balanced 2000-observation OU-simulated test set yields classification accuracy near the reported 97.90% for the neural network
- For unbalanced redundant-skewed/unique-skewed training, power (redundant recall) and accuracy on the balanced test set fall within a few percentage points of the reported 96.65% power and 96.35% accuracy
- Output per-gene files contain both predicted class labels ('Redundant'/'Unique') and class probabilities for every deletion event in the test set

## Limitations

- Hyperparameter tuning of the neural network is described as a time-consuming process, which is why the CLOUDe authors separate cross-validation from final model training
- num_layers is restricted to the set {0, 1, 2, 3, 4, 5} hidden layers; architectures outside this range are not supported by ClassifierCVnn
- Requires dplyr, mvtnorm, tensorflow, keras (plus ranger, devtools, liquidSVM 1.2.0, and xgboost for the full package) to be installed in the R environment before sourcing CLOUDe.R

## Evidence

- [methods] For the CLOUDe neural network, we perform five-fold cross-validation to identify optimal hyperparameters for the predictor with num_layer layers: "we perform five-fold cross-validation to identify optimal hyperparameters for the predictor with num_layer layers"
- [readme] The optimal number of hidden layers is chosen as the one with the smallest validation loss.: "The optimal number of hidden layers is chosen as the one with the smallest validation loss."
- [readme] Conditional on the optimal lambda and gamma hyperparameters, CLOUDe then fits a neural network predictor with num_layer hidden layers.: "Conditional on the optimal lambda and gamma hyperparameters, CLOUDe then fits a neural network predictor with num_layer hidden layers."
- [readme] These functions output the set of optimal hyperparameters chosen through cross-validation to the files training_prefix.nn.num_layers.classifier_cv and training_prefix.nn.num_layers.predictor_cv: "These functions output the set of optimal hyperparameters chosen through cross-validation to the files training_prefix.nn.num_layers.classifier_cv"
