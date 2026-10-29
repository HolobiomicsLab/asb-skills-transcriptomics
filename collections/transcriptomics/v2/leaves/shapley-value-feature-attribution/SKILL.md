---
name: shapley-value-feature-attribution
description: Use when a fitted Keras/TensorFlow CLOUDe NN classifier exists and you
  need model-agnostic per-feature importance (Shapley values) on expression features
  to identify which genes' features drive class discrimination.
when_to_use_negative:
- When no fitted classifier model exists — Shapley attribution requires the trained
  CLOUDe NN classifier, not raw expression data alone
- When the goal is to estimate evolutionary parameters (theta1, theta2, log10(sigmaSq/(2*alpha)))
  rather than explain classifier decisions — use CLOUDePredictNN instead
- When the training set is highly imbalanced or features are not interpretable as
  derived/survived/lost gene groups, since the validation signal (eL dominance) depends
  on balanced classes and the OU-process feature structure
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: CLOUDe
  role: Provides the fitted neural network classifier (via ClassifierCVnn) whose predictions
    the Shapley procedure explains
  repo: https://github.com/anddssan/CLOUDe
- name: keras
  role: Loads and serves the fitted Keras/TensorFlow classifier's class-probability
    prediction function for wrapping by iml
  repo: https://github.com/anddssan/CLOUDe
- name: iml
  role: Runs the model-agnostic Shapley procedure on Training.features to compute
    per-feature contributions to predicted classes
- name: R
  role: Implementation environment; CLOUDe is sourced via source("CLOUDe.R")
  repo: https://github.com/anddssan/CLOUDe
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbae002
    title: Predicting evolutionary targets and parameters of gene deletion from expression
      data
  source_content_hash: sha256:bfb0d243196f3c610f5b5bcb03ef30cd36de5a7279a611e6cbaec91606f75216
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/anddssan/CLOUDe
  tools:
  - CLOUDe
  - R
  - iml
  - keras
  - tensorflow
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# shapley-value-feature-attribution

## Summary

Compute per-feature Shapley contributions of a trained CLOUDe neural network classifier using R's iml framework, ranking all p=3m expression features (eD, eS, eL groups) by mean absolute contribution to explain which features drive Redundant-vs-Unique class discrimination.

## When to use

Use when a fitted Keras/TensorFlow CLOUDe NN classifier exists and you need model-agnostic per-feature importance (Shapley values) on expression features to identify which genes' features drive class discrimination.

## When NOT to use

- When no fitted classifier model exists — Shapley attribution requires the trained CLOUDe NN classifier, not raw expression data alone
- When the goal is to estimate evolutionary parameters (theta1, theta2, log10(sigmaSq/(2*alpha))) rather than explain classifier decisions — use CLOUDePredictNN instead
- When the training set is highly imbalanced or features are not interpretable as derived/survived/lost gene groups, since the validation signal (eL dominance) depends on balanced classes and the OU-process feature structure

## Inputs

- Fitted CLOUDe neural network classifier in TensorFlow HDF5 format (training_prefix.nn.num_layers.classifier.hdf5)
- Balanced training feature matrix Training.features (p=3m columns, eD/eS/eL groups) from GenerateTrainingData()
- Training classes file Training.classes (one-hot encoded 'Redundant'/'Unique' labels)

## Outputs

- Ranked table of 18 features by mean absolute Shapley contribution (shapley_feature_importance.csv)
- Per-feature Shapley values aggregated by group (eD derived, eS survived, eL lost)

## How to apply

Wrap the fitted Keras classifier's class-probability prediction function (e.g. Training.nn.2.classifier.hdf5) so it can be called by the iml framework, then run iml's Shapley procedure on the balanced Training.features matrix to obtain per-feature Shapley contributions to the 'Redundant' vs 'Unique' predicted classes. Aggregate Shapley values by feature group (eD derived, eS survived, eL lost) and rank all 18 features (m=6 conditions, p=3m) by mean absolute contribution. The biological rationale is that only the L gene's expression optimum can differ between classes, so the eL features should dominate the ranking; this consistency between model attribution and class definition serves as the validation criterion. Write the ranked table to shapley_feature_importance.csv for reporting.

## Related tools

- **CLOUDe** (Provides the fitted neural network classifier (via ClassifierCVnn) whose predictions the Shapley procedure explains) — https://github.com/anddssan/CLOUDe
- **keras** (Loads and serves the fitted Keras/TensorFlow classifier's class-probability prediction function for wrapping by iml) — https://github.com/anddssan/CLOUDe
- **iml** (Runs the model-agnostic Shapley procedure on Training.features to compute per-feature contributions to predicted classes)
- **R** (Implementation environment; CLOUDe is sourced via source("CLOUDe.R")) — https://github.com/anddssan/CLOUDe

## Examples

```
source("CLOUDe.R"); GenerateTrainingData(6, 100, "ExampleFiles/Training", 0, 5, 0, 3, -2, 3, "ExampleFiles/EmpiricalData.data"); ClassifierCVnn(2, 50, 50, -3, 3, 7, 0, 1, 3, "ExampleFiles/Training")
```

## Evaluation signals

- The ranked feature table places all lost-gene features (eL1–eL6) at the top, since only the L gene's expression optimum can differ between 'redundant' and 'unique' classes
- Shapley aggregation covers all p=3m=18 features with non-negative mean absolute contributions summing to a stable total across samples
- The wrapped Keras prediction function reproduces the classifier's class probabilities on Training.features before Shapley computation
- Output CSV shapley_feature_importance.csv contains the ranked table with feature names matching the eD/eS/eL header format
- Training data is balanced (Nk=100 observations per class) before attribution to avoid class-prior bias in Shapley values

## Limitations

- Shapley computation over the 18-feature space can be computationally expensive; the CLOUDe README notes hyperparameter tuning and model training are time-consuming steps that precede attribution
- The validation signal (eL feature dominance) is partially expected by construction, so high eL importance alone does not establish novel biological insight
- Attribution applies to the neural network architecture; conclusions may not transfer to the CLOUDe extreme gradient boosting, random forest, or support vector machine classifiers

## Evidence

- [results] Shapley analysis of the CLOUDe NN classifier showed that features associated with the L gene are the most important for discriminating between the 'redundant' and 'unique' classes: [evidence span withheld — anchor status: not_found]
- [readme] ClassifierCVnn command signature: "ClassifierCVnn(num_layers, batchsize, num_epochs, log_lambda_min, log_lambda_max, num_lambda, gamma_min, gamma_max, num_gamma, training_prefix)"
- [readme] Feature header format: ""eD1" "eD2" ... "eDm" "eS1" "eS2" ... "eSm" "eL1" "eL2" ... "eLm""
- [methods] Generating training data from OU process: "GenerateTrainingData(m, Nk, training_prefix, logThetaMin, logThetaMax, logAlphaMin, logAlphaMax, logSigmaSqMin, logSigmaSqMax, empiricalFile)"
- [abstract] Optimal NN performance: "CLOUDe boasts high power and accuracy in differentiating between classes, and high accuracy and precision in estimating evolutionary parameters, with optimal performance achieved by its neural"
