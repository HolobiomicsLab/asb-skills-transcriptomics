---
name: likelihood-ratio-test-classifier-construction
description: Use when benchmarking machine learning classifiers for 'Redundant' vs
  'Unique' deletion-class prediction from two-species expression data and a classical
  maximum-likelihood baseline is needed for comparison.
when_to_use_negative:
- When the goal is predicting continuous evolutionary parameters (theta1, theta2,
  log10(sigmaSq/(2*alpha))) rather than discrete 'Redundant'/'Unique' classes — the
  LRT here serves only the classification benchmark
- When input expression data are not from two species with derived, survived, and
  lost gene columns (the 3m-column eD/eS/eL layout required by CLOUDe)
edam_operation: http://edamontology.org/operation_2990
tools:
- name: CLOUDe
  role: R package providing GenerateTrainingData and the ML classifiers that the LRT
    baseline is benchmarked against
  repo: https://github.com/anddssan/CLOUDe
- name: R
  role: Environment in which CLOUDe is loaded via source("CLOUDe.R") and the LRT classifier
    is fit on the test data
- name: mvtnorm
  role: Dependency for the multivariate normal computations underlying the OU-simulated
    data used in the LRT benchmark
- name: ranger
  role: Random forest classifier (CLOUDeClassifyRF) whose accuracy is compared against
    the LRT baseline
- name: liquidSVM
  role: Support vector machine classifier (CLOUDeClassifySVM, version 1.2.0) compared
    against the LRT baseline
- name: xgboost
  role: Extreme gradient boosting classifier (CLOUDeClassifyXGB) compared against
    the LRT baseline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbae002
    title: Predicting evolutionary targets and parameters of gene deletion from expression
      data
  source_content_hash: sha256:ab094dec7f7b699c0cb22f83f67fa35301cce5c6c13819cc27d68b23e10d6e29
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# likelihood-ratio-test-classifier-construction

## Summary

Fits a maximum-likelihood likelihood-ratio-test (LRT) classifier on the same class-balanced Ornstein–Uhlenbeck-simulated expression test data used by CLOUDe, providing the classical statistical baseline against which the NN, XGB, RF, and SVM classifiers are benchmarked.

## When to use

Use when benchmarking machine learning classifiers for 'Redundant' vs 'Unique' deletion-class prediction from two-species expression data and a classical maximum-likelihood baseline is needed for comparison.

## When NOT to use

- When the goal is predicting continuous evolutionary parameters (theta1, theta2, log10(sigmaSq/(2*alpha))) rather than discrete 'Redundant'/'Unique' classes — the LRT here serves only the classification benchmark
- When input expression data are not from two species with derived, survived, and lost gene columns (the 3m-column eD/eS/eL layout required by CLOUDe)

## Inputs

- OU-simulated class-balanced test features (p=3m columns, e.g. m=6 conditions, space-delimited file matching the CLOUDe training_prefix.features format)
- True simulated class labels ('Redundant'/'Unique') for the 2,000 test observations
- Empirical expression input file (e.g. ExampleFiles/EmpiricalData.data) used to bound simulated parameter ranges

## Outputs

- LRT predicted classes and probabilities for each test deletion event
- Accuracy of the LRT classifier (expected ~85.75% on the balanced test set)
- Combined accuracy table written to classifier_accuracies.csv comparing LRT, NN (97.90%), XGB (96.10%), RF (93.60%), and SVM (90.45%)

## How to apply

Simulate an independent class-balanced OU test set of 2,000 observations with GenerateTrainingData(m=6, Nk=20000, logThetaMin=0, logThetaMax=5, logAlphaMin=0, logAlphaMax=3, logSigmaSqMin=-2, logSigmaSqMax=3, empiricalFile), bounded by the empirical expression ranges. Fit the maximum-likelihood LRT classifier on the same test data used by CLOUDeClassifyNN, CLOUDeClassifyXGB, CLOUDeClassifyRF (ranger), and CLOUDeClassifySVM (liquidSVM), which output predicted classes and probabilities per deletion event. Compute accuracy for the LRT as the fraction of correctly predicted 'Redundant'/'Unique' labels and compare against the ML accuracies, writing the table to classifier_accuracies.csv. The rationale is that the LRT is the optimal classical test under the OU model, so ML architectures should demonstrate clear accuracy gains over it to justify their use; in the published benchmark the LRT achieved ~85.75% versus ~97.90% for the neural network.

## Related tools

- **CLOUDe** (R package providing GenerateTrainingData and the ML classifiers that the LRT baseline is benchmarked against) — https://github.com/anddssan/CLOUDe
- **R** (Environment in which CLOUDe is loaded via source("CLOUDe.R") and the LRT classifier is fit on the test data)
- **mvtnorm** (Dependency for the multivariate normal computations underlying the OU-simulated data used in the LRT benchmark)
- **ranger** (Random forest classifier (CLOUDeClassifyRF) whose accuracy is compared against the LRT baseline)
- **liquidSVM** (Support vector machine classifier (CLOUDeClassifySVM, version 1.2.0) compared against the LRT baseline)
- **xgboost** (Extreme gradient boosting classifier (CLOUDeClassifyXGB) compared against the LRT baseline)

## Examples

```
source("CLOUDe.R"); CLOUDeClassifyRF(training_prefix, testing_prefix); CLOUDeClassifySVM(training_prefix, testing_prefix)  # fit ML predictions, then compute LRT accuracy on the same 2000-observation test set
```

## Evaluation signals

- LRT accuracy on the 2,000-observation balanced test set should fall near the published ~85.75% value
- LRT accuracy should be the lowest of the five methods, below SVM (~90.45%) and well below the neural network (~97.90%)
- Accuracy must be computed as the fraction of correctly predicted 'Redundant'/'Unique' labels on identical test observations used for all five classifiers
- classifier_accuracies.csv should contain all five methods' accuracies with the NN ranked first

## Limitations

- The benchmark relies on simulated OU data whose parameter ranges (logTheta [0,5], logAlpha [0,3], logSigmaSq [-2,3]) are bounded by empirical expression ranges; performance may differ on real data
- Hyperparameter-tuned models (NN via ClassifierCVnn, XGB via ClassifierCVxgb) are stochastic (tensorflow/keras training), so exact accuracy values will vary between runs

## Evidence

- [results] Classification accuracies on balanced simulated data: [evidence span withheld — anchor status: not_found]
- [readme] Classification function signatures: "CLOUDeClassifyNN(training_prefix, testing_prefix, num_layers)   # neural network
    CLOUDeClassifyXGB(training_prefix, testing_prefix, max_depth)   # extreme gradient boosting"
- [methods] Prediction of classes and parameters: "CLOUDe can predict classes ("Redundant" or "Unique") and evolutionary parameters (theta1, theta2, and log10(sigmaSq/(2*alpha))) for each gene in a test dataset."
