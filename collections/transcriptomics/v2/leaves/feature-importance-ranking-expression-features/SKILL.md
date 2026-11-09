---
name: feature-importance-ranking-expression-features
description: Use when a trained CLOUDe classifier (e.g. Training.nn.2.classifier.hdf5)
  exists and you need to know which of the 18 expression features—derived (eD), survived
  (eS), or lost (eL) gene columns—drive the Redundant/Unique class discrimination.
when_to_use_negative:
- The goal is predicting evolutionary parameters (theta1, theta2, log10(sigmaSq/(2*alpha)))
  rather than class discrimination — use PredictorCVnn/CLOUDePredictNN instead of
  the classifier Shapley analysis.
- The model was not trained with the CLOUDe classifier functions (ClassifierCVnn/ClassifierCVxgb)
  — the Shapley wrapper and expected eL-dominance finding assume the CLOUDe Redundant/Unique
  class structure.
- Expression data lack the derived/survived/lost (eD/eS/eL) triplet format with m
  conditions per gene, since the 3m-column header structure defines the feature groups
  to aggregate.
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: CLOUDe
  role: Provides GenerateTrainingData and ClassifierCVnn to build the balanced training
    set and the fitted neural-network classifier whose features are ranked
  repo: https://github.com/anddssan/CLOUDe
- name: keras
  role: Loads the fitted .hdf5 classifier model so its class-probability prediction
    function can be wrapped for Shapley analysis
- name: tensorflow
  role: Backend for the Keras/TensorFlow-format classifier model used in the iml prediction
    wrapper
- name: iml
  role: Runs the Shapley procedure on Training.features to compute per-feature contributions
    to the predicted classes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbae002
    title: Predicting evolutionary targets and parameters of gene deletion from expression
      data
  source_content_hash: sha256:cc80b1f3367b0236c3d0ac7da1af93ebce9c350ae113b5a5cbee9327d191968b
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# feature-importance-ranking-expression-features

## Summary

Ranks the p=3m input expression features (eD, eS, eL groups) of a fitted CLOUDe neural-network classifier by their Shapley contributions to the Redundant-vs-Unique class decision, using the iml framework on the balanced OU-simulated training set.

## When to use

Use when a trained CLOUDe classifier (e.g. Training.nn.2.classifier.hdf5) exists and you need to know which of the 18 expression features—derived (eD), survived (eS), or lost (eL) gene columns—drive the Redundant/Unique class discrimination.

## When NOT to use

- The goal is predicting evolutionary parameters (theta1, theta2, log10(sigmaSq/(2*alpha))) rather than class discrimination — use PredictorCVnn/CLOUDePredictNN instead of the classifier Shapley analysis.
- The model was not trained with the CLOUDe classifier functions (ClassifierCVnn/ClassifierCVxgb) — the Shapley wrapper and expected eL-dominance finding assume the CLOUDe Redundant/Unique class structure.
- Expression data lack the derived/survived/lost (eD/eS/eL) triplet format with m conditions per gene, since the 3m-column header structure defines the feature groups to aggregate.

## Inputs

- CLOUDe.R script sourced in R (with dplyr, mvtnorm, tensorflow, keras, ranger, devtools, liquidSVM 1.2.0, xgboost installed)
- Space-delimited input file in CLOUDe format (N+1 rows, 3m columns, header eD1..eDm eS1..eSm eL1..eLm), e.g. ExampleFiles/EmpiricalData.data with N=100 deletion events at m=6 tissues
- Balanced training feature matrix Training.features (p=3m features) and one-hot class matrix Training.classes from GenerateTrainingData
- Fitted Keras/TensorFlow classifier model, e.g. Training.nn.2.classifier.hdf5 from ClassifierCVnn

## Outputs

- Per-feature Shapley contribution values for each observation in Training.features (Redundant-vs-Unique predicted classes)
- Ranked feature-importance table aggregated by mean absolute Shapley contribution, written to shapley_feature_importance.csv

## How to apply

Load CLOUDe.R and simulate a balanced training set with GenerateTrainingData (e.g. m=6 conditions, Nk=100 observations per class, ranges logTheta 0–5, logAlpha 0–3, logSigmaSq -2–3, empiricalFile ExampleFiles/EmpiricalData.data), yielding 18 features in Training.features. Train the neural-network classifier with ClassifierCVnn(2, 50, 50, -3, 3, 7, 0, 1, 3, ...) so the fitted Keras/TensorFlow model Training.nn.2.classifier.hdf5 is produced after five-fold cross-validation over log10(lambda) and gamma. Wrap the classifier's class-probability prediction function for the iml framework and run iml's Shapley procedure on the balanced Training.features matrix to obtain per-feature Shapley contributions. Aggregate Shapley values by feature group (eD derived, eS survived, eL lost) and rank all 18 features by mean absolute contribution; because only the lost gene's expression optimum can differ between classes, the eL1–eL6 features should dominate the top of the ranking.

## Related tools

- **CLOUDe** (Provides GenerateTrainingData and ClassifierCVnn to build the balanced training set and the fitted neural-network classifier whose features are ranked) — https://github.com/anddssan/CLOUDe
- **keras** (Loads the fitted .hdf5 classifier model so its class-probability prediction function can be wrapped for Shapley analysis)
- **tensorflow** (Backend for the Keras/TensorFlow-format classifier model used in the iml prediction wrapper)
- **iml** (Runs the Shapley procedure on Training.features to compute per-feature contributions to the predicted classes)

## Examples

```
source("CLOUDe.R"); GenerateTrainingData(6, 100, "ExampleFiles/Training", 0, 5, 0, 3, -2, 3, "ExampleFiles/EmpiricalData.data"); ClassifierCVnn(2, 50, 50, -3, 3, 7, 0, 1, 3, "ExampleFiles/Training")
```

## Evaluation signals

- Training set is balanced: Nk=100 observations per class over p=3m=18 features for m=6 conditions.
- ClassifierCVnn output files exist: optimal hyperparameters in Training.nn.2.classifier_cv and fitted model Training.nn.2.classifier.hdf5.
- Ranking places the lost-gene features eL1–eL6 at the top by mean absolute Shapley contribution, since only the L gene's expression optimum can differ between the Redundant and Unique classes.
- Aggregated table groups all 18 features into the three expected blocks eD1–eD6, eS1–eS6, eL1–eL6 and is written to shapley_feature_importance.csv.
- eD/eS features contribute near-zero mean absolute Shapley value relative to eL features, consistent with the class definition.

## Limitations

- Hyperparameter tuning of the CLOUDe neural network is time-consuming; the README notes five-fold cross-validation over lambda and gamma must precede model fitting.
- The eL-dominance result is partly by construction (classes are defined so only the lost gene's optimum can differ), so top-ranked eL features validate the pipeline rather than reveal new biology.
- Training of the CLOUDe neural network (and hence the Shapley analysis on it) must be done separately; support vector machine and random forest models are trained on-the-fly and are not covered by this workflow.

## Evidence

- [readme] eD1 to eDm denote derived gene expression values for conditions 1 to m, eS1 to eSm denote survived gene expression values for conditions 1 to m: "eD1 to eDm denote derived gene expression values for conditions 1 to m, eS1 to eSm denote survived gene expression values for conditions 1 to m"
- [readme] The ExampleFiles directory contains a file called EmpiricalData.data, which illustrates this format for N=100 deletion events at m=6 tissues.: "The ExampleFiles directory contains a file called EmpiricalData.data, which illustrates this format for N=100 deletion events at m=6 tissues."
- [methods] we perform five-fold cross-validation to identify optimal hyperparameters for the predictor with num_layer layers: "we perform five-fold cross-validation to identify optimal hyperparameters for the predictor with num_layer layers"
- [abstract] CLOUDe boasts high power and accuracy in differentiating between classes, and high accuracy and precision in estimating evolutionary parameters, with optimal performance achieved by its neural: "CLOUDe boasts high power and accuracy in differentiating between classes, and high accuracy and precision in estimating evolutionary parameters, with optimal performance achieved by its neural"
