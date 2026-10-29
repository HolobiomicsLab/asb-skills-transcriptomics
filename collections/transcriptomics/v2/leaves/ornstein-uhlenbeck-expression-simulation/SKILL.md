---
name: ornstein-uhlenbeck-expression-simulation
description: Use when you need simulated OU-process expression data with known Redundant/Unique
  deletion class labels (e.g., to train or benchmark CLOUDe classifiers) for Nk observations
  per class across m conditions, bounded by empirical expression ranges.
when_to_use_negative:
- The input is real expression data with unknown true deletion classes — simulation
  provides synthetic labels; use the CLOUDeClassify* functions on empirical data instead
- You only need to apply pre-trained classifiers or predictors to an existing test
  set — CLOUDeClassifyNN/XGB/RF/SVM take existing feature files and skip data generation
- The target evolutionary scenario requires a distinct ancestral optimum (theta0 differing
  from theta1 and theta2), which needs separately simulated data rather than the standard
  class-balanced generation
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_3474
tools:
- name: CLOUDe
  role: R package supplying the GenerateTrainingData() OU-simulation function and
    the output file conventions (.features, .classes, .responses)
  repo: https://github.com/anddssan/CLOUDe
- name: R
  role: Runtime environment in which CLOUDe.R is sourced and simulation is run
- name: mvtnorm
  role: R dependency of CLOUDe used for the multivariate-normal draws underlying OU
    simulation
- name: dplyr
  role: R dependency of CLOUDe required before running GenerateTrainingData
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbae002
    title: Predicting evolutionary targets and parameters of gene deletion from expression
      data
  source_content_hash: sha256:f56656efb000c8ee9f487c9bf506d9bd148558364956f3cef22e65d5c3a5021c
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# ornstein-uhlenbeck-expression-simulation

## Summary

Simulates class-balanced Ornstein–Uhlenbeck (OU) expression-evolution training and test data for gene deletion events using CLOUDe's GenerateTrainingData(), producing labeled 'Redundant'/'Unique' observations whose expression ranges are bounded by an empirical dataset.

## When to use

Use when you need simulated OU-process expression data with known Redundant/Unique deletion class labels (e.g., to train or benchmark CLOUDe classifiers) for Nk observations per class across m conditions, bounded by empirical expression ranges.

## When NOT to use

- The input is real expression data with unknown true deletion classes — simulation provides synthetic labels; use the CLOUDeClassify* functions on empirical data instead
- You only need to apply pre-trained classifiers or predictors to an existing test set — CLOUDeClassifyNN/XGB/RF/SVM take existing feature files and skip data generation
- The target evolutionary scenario requires a distinct ancestral optimum (theta0 differing from theta1 and theta2), which needs separately simulated data rather than the standard class-balanced generation

## Inputs

- empiricalFile: space-delimited input file with N+1 rows and 3m columns (header eD1..eDm eS1..eSm eL1..eLm) of absolute log10(x+1) expression values, e.g. ExampleFiles/EmpiricalData.data
- m: number of conditions (e.g., tissues), e.g. m=6
- Nk: number of simulated training observations per class (Redundant/Unique), e.g. 20000
- OU parameter range bounds: logThetaMin/Max, logAlphaMin/Max, logSigmaSqMin/Max (e.g., 0–5, 0–3, -2–3)

## Outputs

- training_prefix.features: matrix of p=3m extracted features for all simulated observations
- training_prefix.classes: one-hot encoded matrix of Redundant/Unique classifications
- training_prefix.responses: matrix of predicted expression optima for all training observations
- training_prefix.data: raw simulated data
- training_prefix.X_stdparams: means and standard deviations for each of the p features
- training_prefix.Y_stdparams: means and standard deviations for each of the 3m model parameters

## How to apply

Load CLOUDe in R via source("CLOUDe.R") and call GenerateTrainingData(m, Nk, training_prefix, logThetaMin, logThetaMax, logAlphaMin, logAlphaMax, logSigmaSqMin, logSigmaSqMax, empiricalFile), where m is the number of conditions, Nk is the number of observations per class, and log-ranges bound the OU parameters theta (optimal expression), alpha (strength of selection), and sigmaSq (strength of phenotypic drift). The empiricalFile must be a space-delimited file with N+1 rows and 3m columns (header eD1...eDm eS1...eSm eL1...eLm) of absolute log10(x+1) expression values for derived, survived, and lost genes — e.g., the provided EmpiricalData.data with N=100 deletion events at m=6 tissues. A typical paper invocation uses m=6, Nk=20000, logTheta in [0,5], logAlpha in [0,3], and logSigmaSq in [-2,3], with these ranges anchored to the empirical expression values. The function writes p=3m features, a one-hot encoded class matrix, predicted expression optima (responses), raw simulated data, and per-feature/per-parameter means and standard deviations, which downstream functions (ClassifierCVnn, ClassifierCVxgb, CLOUDeClassify*) consume. To generate an independent test set, re-invoke the function with the same parameter ranges but a different prefix and smaller Nk (e.g., 2000 total observations).

## Related tools

- **CLOUDe** (R package supplying the GenerateTrainingData() OU-simulation function and the output file conventions (.features, .classes, .responses)) — https://github.com/anddssan/CLOUDe
- **R** (Runtime environment in which CLOUDe.R is sourced and simulation is run)
- **mvtnorm** (R dependency of CLOUDe used for the multivariate-normal draws underlying OU simulation)
- **dplyr** (R dependency of CLOUDe required before running GenerateTrainingData)

## Examples

```
source("CLOUDe.R"); GenerateTrainingData(m=6, Nk=20000, "Training", logThetaMin=0, logThetaMax=5, logAlphaMin=0, logAlphaMax=3, logSigmaSqMin=-2, logSigmaSqMax=3, "ExampleFiles/EmpiricalData.data")
```

## Evaluation signals

- All six output files exist with the expected prefixes: .features, .classes, .data, .responses, .X_stdparams, .Y_stdparams
- .features contains p=3m columns (e.g., 18 columns for m=6) matching the eD/eS/eL feature naming, and .classes is one-hot encoded with Nk observations per class
- Simulated feature ranges stay consistent with the empirical expression ranges used to bound the OU parameters
- An independently generated test set (same log-ranges, different prefix) yields balanced Redundant/Unique labels suitable for accuracy/power evaluation
- Downstream ClassifierCVnn/ClassifierCVxgb runs load the generated .features and .classes without format errors

## Limitations

- Simulation ranges must be defined relative to an empirical file (empiricalFile); without it the theta/alpha/sigmaSq bounds have no empirical anchor
- The standard generation produces the scenario where class definitions rely on the lost gene's expression optimum; alternative scenarios (distinct ancestral theta0) require separately generated test data
- The CLOUDe pipeline requires specific R library versions (e.g., liquidSVM 1.2.0 installed via devtools install_version) before source("CLOUDe.R") will work

## Evidence

- [abstract] CLOUDe models expression evolution as an Ornstein–Uhlenbeck process, and uses multi-layer neural network, extreme gradient boosting, random forest, and support vector machine architectures: "CLOUDe models expression evolution as an Ornstein–Uhlenbeck process, and uses multi-layer neural network, extreme gradient boosting, random forest, and support vector machine architectures"
- [methods] GenerateTrainingData signature and parameter semantics: "GenerateTrainingData(m, Nk, training_prefix, logThetaMin, logThetaMax, logAlphaMin, logAlphaMax, logSigmaSqMin, logSigmaSqMax, empiricalFile)"
- [readme] Output files produced by the simulation function: "The GenerateTrainingData() function also outputs the p=3m features to the file training_prefix.features, a one-hot encoded matrix of classifications for all training observations"
- [methods] Empirical example file anchoring the simulation ranges: "an example file called EmpiricalData.data is provided in the ExampleFiles directory"
- [readme] Input file format the simulation ranges are derived from: "A space-delimited file with N+1 rows and 3m columns, where N is the number of genes and m is the number of conditions"
