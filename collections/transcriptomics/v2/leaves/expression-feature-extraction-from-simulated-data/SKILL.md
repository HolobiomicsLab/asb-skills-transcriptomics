---
name: expression-feature-extraction-from-simulated-data
description: Use when OU-simulated deletion-event expression data (or empirical data
  in the same format) must be transformed into the p=3m feature file that CLOUDeClassifyNN/XGB/RF/SVM
  and CLOUDePredict* require.
when_to_use_negative:
- Expression data is not in the two-species derived/survived/lost deletion-event layout
  (3m columns with log10(x+1) values) — the CLOUDe feature schema does not apply
- A testing feature file already exists on the same standardized scale — re-extraction
  is unnecessary and only CLOUDeClassify*/CLOUDePredict* should be run
- The goal is hyperparameter tuning or final classification (ClassifierCVnn, CLOUDeClassifyNN),
  which consume these features rather than produce them
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0196
tools:
- name: CLOUDe
  role: Provides GenerateTrainingData(), the function that simulates OU data and outputs
    the p=3m feature file, classes, responses, and standardization parameters
  repo: https://github.com/anddssan/CLOUDe
- name: R
  role: Language in which CLOUDe.R is sourced and GenerateTrainingData is executed
  repo: https://github.com/anddssan/CLOUDe
- name: mvtnorm
  role: R library used for the multivariate normal sampling underlying the OU-process
    simulation of expression data
- name: dplyr
  role: R library required in the environment for the CLOUDe feature-generation and
    downstream processing steps
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae002
    title: Predicting evolutionary targets and parameters of gene deletion from expression
      data
  source_content_hash: sha256:2e7cdd8950a82e2f63874b637ed4aaba139f2be339583a1592345c636dca0f1d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/anddssan/CLOUDe
  tools:
  - CLOUDe
  - R
  - dplyr
  - liquidSVM
  - mvtnorm
  - ranger
  - tensorflow
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# expression-feature-extraction-from-simulated-data

## Summary

Feature-extraction step of the CLOUDe pipeline: Ornstein–Uhlenbeck-simulated expression data for derived, survived, and lost genes is converted into a p=3m standardized feature matrix (plus one-hot class labels and parameter responses) that the CLOUDe classifiers and predictors consume.

## When to use

Use when OU-simulated deletion-event expression data (or empirical data in the same format) must be transformed into the p=3m feature file that CLOUDeClassifyNN/XGB/RF/SVM and CLOUDePredict* require.

## When NOT to use

- Expression data is not in the two-species derived/survived/lost deletion-event layout (3m columns with log10(x+1) values) — the CLOUDe feature schema does not apply
- A testing feature file already exists on the same standardized scale — re-extraction is unnecessary and only CLOUDeClassify*/CLOUDePredict* should be run
- The goal is hyperparameter tuning or final classification (ClassifierCVnn, CLOUDeClassifyNN), which consume these features rather than produce them

## Inputs

- Space-delimited expression file with N+1 rows and 3m columns (header 'eD1'...'eDm' 'eS1'...'eSm' 'eL1'...'eLm'), e.g., EmpiricalData.data from the ExampleFiles directory
- OU parameter ranges: logThetaMin/logThetaMax, logAlphaMin/logAlphaMax, logSigmaSqMin/logSigmaSqMax
- Number of conditions m and number of training observations per class Nk
- Simulated test expression data (e.g., generated under the alternative scenario where theta0 differs from theta1 and theta2)

## Outputs

- training_prefix.features: file of p=3m features (p=18 for m=6)
- training_prefix.classes: one-hot encoded matrix of 'Redundant'/'Unique' classifications for all training observations
- training_prefix.responses: matrix of predicted expression optima (theta1, theta2) for all training observations
- training_prefix.data: raw simulated data
- training_prefix.X_stdparams and training_prefix.Y_stdparams: means and standard deviations for each of the p features and each of the 3m model parameters
- testing_prefix.features: test-set features extracted with the same pipeline for use by CLOUDeClassifyNN/CLOUDeClassifyXGB/CLOUDeClassifyRF/CLOUDeClassifySVM

## How to apply

First simulate expression evolution under the Ornstein–Uhlenbeck process, drawing theta, alpha, and sigmaSq from the ranges defined by logThetaMin–logThetaMax, logAlphaMin–logAlphaMax, and logSigmaSqMin–logSigmaSqMax, and arrange values as absolute log10(x+1) expression in a space-delimited file with N+1 rows and 3m columns (first m columns derived gene eD1–eDm, next m survived eS1–eSm, last m lost eL1–eLm). Then run GenerateTrainingData(m, Nk, training_prefix, ..., empiricalFile) where the empirical file (e.g., EmpiricalData.data with N=100 deletion events at m=6 tissues, giving p=18 features) anchors the parameter ranges. The function writes the p=3m features to training_prefix.features, a one-hot encoded matrix of 'Redundant'/'Unique' classifications to training_prefix.classes, predicted expression optima to training_prefix.responses, raw simulated data to training_prefix.data, and standardization means/SDs to training_prefix.X_stdparams and training_prefix.Y_stdparams. For a test set (including data simulated under the alternative ancestral-optimum scenario with distinct theta0), apply the identical GenerateTrainingData-style extraction so the testing feature file is on the same standardized scale as the training features before calling CLOUDeClassifyNN(training_prefix, testing_prefix, num_layers) or the XGB/RF/SVM counterparts. Rationale: downstream classifiers score classification power as the fraction of simulated events assigned to the correct 'Redundant' or 'Unique' class, so feature consistency between training and testing files is essential.

## Related tools

- **CLOUDe** (Provides GenerateTrainingData(), the function that simulates OU data and outputs the p=3m feature file, classes, responses, and standardization parameters) — https://github.com/anddssan/CLOUDe
- **R** (Language in which CLOUDe.R is sourced and GenerateTrainingData is executed) — https://github.com/anddssan/CLOUDe
- **mvtnorm** (R library used for the multivariate normal sampling underlying the OU-process simulation of expression data)
- **dplyr** (R library required in the environment for the CLOUDe feature-generation and downstream processing steps)

## Examples

```
source("CLOUDe.R"); GenerateTrainingData(m, Nk, training_prefix, logThetaMin, logThetaMax, logAlphaMin, logAlphaMax, logSigmaSqMin, logSigmaSqMax, empiricalFile)
```

## Evaluation signals

- training_prefix.features contains exactly p=3m columns (18 features for the m=6 EmpiricalData example) and one row per training observation (2*Nk rows)
- training_prefix.classes is a valid one-hot encoded matrix with exactly two classes ('Redundant' and 'Unique')
- training_prefix.X_stdparams contains a mean and standard deviation for each of the p features, and Y_stdparams for each of the 3m model parameters
- Input values are absolute log10(x+1) expression values arranged as eD, eS, eL blocks of m conditions matching the header 'eD1'...'eLm'
- Test-set features extracted the same way feed classifiers that achieve high power (predictions agree strongly with true simulated classes, as in Supplementary Fig. S3B under the alternative theta0 scenario)

## Limitations

- Feature generation requires the empiricalFile to define realistic parameter ranges; if its format or scale (log10(x+1), 3m columns) is violated, the extracted features will be mis-scaled
- The environment must have dplyr, mvtnorm, tensorflow, keras, ranger, devtools, liquidSVM (pinned to version 1.2.0), and xgboost installed before sourcing CLOUDe.R
- Simulated data under the alternative ancestral-optimum scenario (theta0 distinct from theta1 and theta2) must still be extracted identically; inconsistent standardization between training and testing prefixes degrades classification power

## Evidence

- [readme] The GenerateTrainingData() function also outputs the p=3m features to the file training_prefix.features, a one-hot encoded matrix of classifications: "The GenerateTrainingData() function also outputs the p=3m features to the file training_prefix.features, a one-hot encoded matrix of classifications"
- [readme] A space-delimited file with N+1 rows and 3m columns, where N is the number of genes and m is the number of conditions: "A space-delimited file with N+1 rows and 3m columns, where N is the number of genes and m is the number of conditions"
- [readme] The ExampleFiles directory contains a file called EmpiricalData.data, which illustrates this format for N=100 deletion events at m=6 tissues.: "The ExampleFiles directory contains a file called EmpiricalData.data, which illustrates this format for N=100 deletion events at m=6 tissues."
- [readme] where training_prefix and testing_prefix are the prefixes to training and testing feature files that were outputted by the GenerateTrainingData() function: "where training_prefix and testing_prefix are the prefixes to training and testing feature files that were outputted by the GenerateTrainingData() function"
- [methods] GenerateTrainingData(m, Nk, training_prefix, logThetaMin, logThetaMax, logAlphaMin, logAlphaMax, logSigmaSqMin, logSigmaSqMax, empiricalFile): "GenerateTrainingData(m, Nk, training_prefix, logThetaMin, logThetaMax, logAlphaMin, logAlphaMax, logSigmaSqMin, logSigmaSqMax, empiricalFile)"
- [other] Extract the p=3m features from the simulated test data with the GenerateTrainingData-style feature extraction so a testing feature file is available alongside the training features.: [evidence span withheld — anchor status: not_found]
