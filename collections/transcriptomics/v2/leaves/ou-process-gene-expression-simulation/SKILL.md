---
name: ou-process-gene-expression-simulation
description: Use when building CLOUDe training sets from simulated expression data
  under an OU process — e.g., when you need 'redundant-skewed' or 'unique-skewed'
  unbalanced training sets, or balanced training data, to train NN/XGB classifiers
  on expression evolution of deletion events.
when_to_use_negative:
- Your data are already a labeled training feature table produced by GenerateTrainingData
  — regenerate only if class balance or OU parameter ranges must change.
- The downstream model is random forest or SVM — per the README these are trained
  on-the-fly during testing and do not require pre-generated OU simulation output
  in the same way.
- The expression input lacks the 3m-column derived/survived/lost gene structure required
  by the CLOUDe input format.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3299
tools:
- name: CLOUDe
  role: R package providing GenerateTrainingData to simulate OU-based expression training
    data
  repo: https://github.com/anddssan/CLOUDe
- name: mvtnorm
  role: R dependency used by CLOUDe for multivariate normal draws in OU simulation
- name: R
  role: Runtime environment; CLOUDe.R is sourced in R
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae002
    title: Predicting evolutionary targets and parameters of gene deletion from expression
      data
  source_content_hash: sha256:81654aa7fd2435e5b26d77ef69fa46adf9365165367aafb6762d3fd0ff54f485
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/anddssan/CLOUDe
  tools:
  - CLOUDe
  - R
  - dplyr
  - mvtnorm
  - tensorflow
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# ou-process-gene-expression-simulation

## Summary

Simulates training data for CLOUDe's machine learning classifiers by modeling expression evolution of deleted gene sets as an Ornstein–Uhlenbeck process, using the empirical expression file as reference. It produces balanced Redundant/Unique training classes with features, one-hot class labels, and evolutionary parameter responses.

## When to use

Use when building CLOUDe training sets from simulated expression data under an OU process — e.g., when you need 'redundant-skewed' or 'unique-skewed' unbalanced training sets, or balanced training data, to train NN/XGB classifiers on expression evolution of deletion events.

## When NOT to use

- Your data are already a labeled training feature table produced by GenerateTrainingData — regenerate only if class balance or OU parameter ranges must change.
- The downstream model is random forest or SVM — per the README these are trained on-the-fly during testing and do not require pre-generated OU simulation output in the same way.
- The expression input lacks the 3m-column derived/survived/lost gene structure required by the CLOUDe input format.

## Inputs

- empiricalFile: space-delimited expression file with N+1 rows and 3m columns, header eD1..eDm eS1..eSm eL1..eLm (e.g., EmpiricalData.data with N=100 deletion events at m=6 tissues)
- m: number of conditions (e.g., tissues) per gene
- Nk: number of training observations for each of the two classes (Redundant, Unique)
- training_prefix: prefix for all output files
- logThetaMin, logThetaMax, logAlphaMin, logAlphaMax, logSigmaSqMin, logSigmaSqMax: log-range bounds for OU parameters theta, alpha, sigmaSq

## Outputs

- training_prefix.features: p=3m feature matrix for all training observations
- training_prefix.classes: one-hot encoded classification matrix (Redundant vs Unique)
- training_prefix.responses: matrix of predicted expression optima (theta1, theta2, log10(sigmaSq/(2*alpha)))
- training_prefix.data: raw simulated data
- training_prefix.X_stdparams and training_prefix.Y_stdparams: means and standard deviations for feature and parameter standardization

## How to apply

Load CLOUDe in R with source("CLOUDe.R") after installing dplyr, mvtnorm, tensorflow, keras, ranger, devtools, liquidSVM 1.2.0, and xgboost. Call GenerateTrainingData(m, Nk, training_prefix, logThetaMin, logThetaMax, logAlphaMin, logAlphaMax, logSigmaSqMin, logSigmaSqMax, empiricalFile), where m is the number of conditions (e.g., 6 tissues), Nk is the number of training observations per class, and the log-transformed ranges define how theta (optimal expression), alpha (strength of selection), and sigmaSq (strength of phenotypic drift) are drawn. The empirical file is a space-delimited file with N+1 rows and 3m columns of absolute log10(x+1) expression values (columns j, m+j, 2m+j for derived, survived, and lost genes). To create unbalanced training data for redundancy-skew experiments, generate class-specific training sets with very unequal Nk so the resulting 'redundant-skewed' or 'unique-skewed' training data mimic highly unbalanced class frequencies. Outputs (features, classes, responses, data, and standardization parameters) feed directly into ClassifierCVnn or ClassifierCVxgb for hyperparameter tuning and final model fitting.

## Related tools

- **CLOUDe** (R package providing GenerateTrainingData to simulate OU-based expression training data) — https://github.com/anddssan/CLOUDe
- **mvtnorm** (R dependency used by CLOUDe for multivariate normal draws in OU simulation)
- **R** (Runtime environment; CLOUDe.R is sourced in R)

## Examples

```
GenerateTrainingData(m, Nk, training_prefix, logThetaMin, logThetaMax, logAlphaMin, logAlphaMax, logSigmaSqMin, logSigmaSqMax, empiricalFile)
```

## Evaluation signals

- Output files training_prefix.features, training_prefix.classes, training_prefix.responses, training_prefix.data, training_prefix.X_stdparams, and training_prefix.Y_stdparams all exist and are non-empty.
- training_prefix.features has p=3m columns (e.g., 18 columns for m=6 tissues) with one row per simulated training observation.
- training_prefix.classes is one-hot encoded with exactly two columns (Redundant, Unique) and Nk rows per class under balanced generation.
- Downstream NN classifier trained on the simulated data achieves near-reported performance (e.g., 96.65% power and 96.35% accuracy on a balanced test set for unbalanced training).
- Feature standardization parameters in X_stdparams are consistent with the empirical expression scale (absolute log10(x+1) values).

## Limitations

- The empirical input must match the strict CLOUDe format (space-delimited, N+1 rows, 3m columns, header eD/eS/eL); mismatches will break simulation or downstream training.
- Balanced vs unbalanced training depends on Nk choice: to reproduce 'redundant-skewed' or 'unique-skewed' experiments you must deliberately generate very unequal class sizes.
- Hyperparameter tuning for the neural network (lambda, gamma, num_layers) is time-consuming; simulation outputs feed a separate, expensive cross-validation step.

## Evidence

- [abstract] CLOUDe models expression evolution as an Ornstein–Uhlenbeck process, and uses multi-layer neural network, extreme gradient boosting, random forest, and support vector machine architectures: "CLOUDe models expression evolution as an Ornstein–Uhlenbeck process, and uses multi-layer neural network, extreme gradient boosting, random forest, and support vector machine architectures"
- [methods] GenerateTrainingData signature with OU parameter ranges: "GenerateTrainingData(m, Nk, training_prefix, logThetaMin, logThetaMax, logAlphaMin, logAlphaMax, logSigmaSqMax, empiricalFile)"
- [readme] Training data output files description: "The GenerateTrainingData() function also outputs the p=3m features to the file training_prefix.features, a one-hot encoded matrix of classifications for all training observations to the file"
- [readme] Input file format specification: "A space-delimited file with N+1 rows and 3m columns, where N is the number of genes and m is the number of conditions (e.g., tissues, ages, etc.) for expression data."
