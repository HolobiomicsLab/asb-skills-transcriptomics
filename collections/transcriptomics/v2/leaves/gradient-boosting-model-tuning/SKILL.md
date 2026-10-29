---
name: gradient-boosting-model-tuning
description: Use when OU-simulated gene-deletion training data (training_prefix.features/.classes
  from GenerateTrainingData) must be fit with an xgboost classifier and hyperparameters
  (eta, log10(lambda), gamma, max_depth) need five-fold cross-validated selection.
when_to_use_negative:
- The goal is predicting continuous evolutionary parameters (theta1, theta2, log10(sigmaSq/(2*alpha)))
  rather than classes — use PredictorCVxgb/CLOUDePredictXGB instead of the classifier
  functions
- The chosen architecture is random forest or SVM, which are trained on-the-fly when
  applied to test data and require no cross-validation tuning step
- Input is raw expression data not yet converted to the CLOUDe 3m-column deletion-event
  format and processed through GenerateTrainingData
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: CLOUDe
  role: R package providing ClassifierCVxgb (five-fold cross-validated hyperparameter
    tuning) and CLOUDeClassifyXGB (class/probability prediction) for the extreme gradient
    boosting classifier
  repo: https://github.com/anddssan/CLOUDe
- name: xgboost
  role: Backend library implementing the extreme gradient boosting models tuned and
    fit by the CLOUDe classifier functions
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbae002
    title: Predicting evolutionary targets and parameters of gene deletion from expression
      data
  source_content_hash: sha256:33817b7047e0bd11df06ed4214b6d13ae0017e5d5e2bcb85070f927f9dcf0777
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
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# gradient-boosting-model-tuning

## Summary

Hyperparameter tuning and training of the CLOUDe extreme gradient boosting (xgboost) classifier, which predicts whether a gene deletion event is 'redundant' or 'unique' from 3m expression features. Five-fold cross-validation selects eta, lambda, and gamma at a chosen tree depth before test-set classification.

## When to use

Use when OU-simulated gene-deletion training data (training_prefix.features/.classes from GenerateTrainingData) must be fit with an xgboost classifier and hyperparameters (eta, log10(lambda), gamma, max_depth) need five-fold cross-validated selection.

## When NOT to use

- The goal is predicting continuous evolutionary parameters (theta1, theta2, log10(sigmaSq/(2*alpha))) rather than classes — use PredictorCVxgb/CLOUDePredictXGB instead of the classifier functions
- The chosen architecture is random forest or SVM, which are trained on-the-fly when applied to test data and require no cross-validation tuning step
- Input is raw expression data not yet converted to the CLOUDe 3m-column deletion-event format and processed through GenerateTrainingData

## Inputs

- OU-simulated training features file (training_prefix.features, p=3m columns of absolute log10(x+1) expression values) produced by GenerateTrainingData
- One-hot encoded class labels file (training_prefix.classes) with 'Redundant'/'Unique' classes
- Empirical expression input file (e.g., EmpiricalData.data: space-delimited, N+1 rows x 3m columns with eD*/eS*/eL* header) used to bound simulated parameter ranges
- OU-simulated test features (testing_prefix.features) for final classification

## Outputs

- Optimal hyperparameter set written to training_prefix.xgb.max_depth.classifier_cv
- Predicted classes per deletion event in test_prefix.xgb.max_depth.classifications
- Predicted class probabilities in test_prefix.xgb.max_depth.probabilities
- Classification accuracy table (e.g., classifier_accuracies.csv) comparing XGB (~96.10%) against NN, RF, SVM, and LRT

## How to apply

Load the CLOUDe package with source("CLOUDe.R") after installing xgboost, and start from the training files emitted by GenerateTrainingData (in the source workflow: m=6 conditions, Nk=20000 observations per class, OU parameter ranges logTheta in [0,5], logAlpha in [0,3], logSigmaSq in [-2,3], bounded by the empirical expression file). Call ClassifierCVxgb with max_depth (the source workflow uses max_depth=4; allowable values are 1-6), grids where eta is evenly spaced in [eta_min, eta_max] for num_eta points, log10(lambda) evenly spaced in [log_lambda_min, log_lambda_max] for num_lambda points, and gamma evenly spaced in [gamma_min, gamma_max] for num_gamma points — the same log10(lambda) in [-3,3] and gamma in [0,1] ranges used for the neural network are a reasonable grid choice. Five-fold cross-validation identifies the optimal lambda, gamma, and eta combination for the classifier, conditional on the chosen tree depth, and writes them to training_prefix.xgb.max_depth.classifier_cv. With tuned hyperparameters in place, apply CLOUDeClassifyXGB(training_prefix, testing_prefix, max_depth) to the independent test set (2000 OU-simulated observations in the source workflow) to produce per-gene predicted classes ('Redundant' or 'Unique') and probabilities. Judge success by comparing predicted labels to the true simulated classes: on the balanced test set the XGB classifier is expected to reach ~96.10% accuracy, second only to the neural network (~97.90%) and above RF (93.60%), SVM (90.45%), and the maximum-likelihood LRT baseline (85.75%).

## Related tools

- **CLOUDe** (R package providing ClassifierCVxgb (five-fold cross-validated hyperparameter tuning) and CLOUDeClassifyXGB (class/probability prediction) for the extreme gradient boosting classifier) — https://github.com/anddssan/CLOUDe
- **xgboost** (Backend library implementing the extreme gradient boosting models tuned and fit by the CLOUDe classifier functions)

## Examples

```
source("CLOUDe.R"); ClassifierCVxgb(4, eta_min, eta_max, num_eta, log_lambda_min, log_lambda_max, num_lambda, gamma_min, gamma_max, num_gamma, training_prefix); CLOUDeClassifyXGB(training_prefix, testing_prefix, 4)
```

## Evaluation signals

- training_prefix.xgb.max_depth.classifier_cv exists and contains the optimal eta, log10(lambda), and gamma values chosen by smallest five-fold validation loss
- The max_depth value passed to CLOUDeClassifyXGB matches the depth used during ClassifierCVxgb tuning (both from {1,2,3,4,5,6}; source workflow used 4)
- test_prefix.xgb.max_depth.classifications contains exactly one 'Redundant'/'Unique' label per test observation, with a matching row count in the .probabilities file
- XGB classification accuracy on the balanced 2000-observation simulated test set falls near 96.10%, ranked between the NN (~97.90%) and RF (93.60%) in the accuracy table
- Grid settings honor the documented spacing rules: eta, gamma, and log10(lambda) each drawn as evenly spaced points across their specified [min, max] intervals

## Limitations

- Hyperparameter estimation is described as a time-consuming process; unlike the random forest and SVM, the XGB model requires this explicit tuning and training step before test predictions
- max_depth is restricted to integer values 1 through 6, so tuning cannot explore deeper trees without modifying the package
- Cross-validation only selects hyperparameters conditional on a given max_depth; the optimal depth is chosen separately by smallest validation loss across separate runs

## Evidence

- [methods] ClassifierCVxgb signature: [evidence span withheld — anchor status: not_found]
- [methods] Five-fold cross-validation purpose: "we perform five-fold cross-validation to identify optimal hyperparameters for the predictor with maximum depth of trees of max_depth"
- [readme] Cross-validation output file: "These functions output the set of optimal hyperparameters chosen through cross-validation to the files training_prefix.xgb.max_depth.classifier_cv"
- [readme] Allowable tree depths: "max_depth is the maximum depth (1, 2, 3, 4, 5 or 6) of trees in the models"
- [abstract] Expected relative performance: "CLOUDe boasts high power and accuracy in differentiating between classes, and high accuracy and precision in estimating evolutionary parameters"
