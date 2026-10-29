---
name: machine-learning-classifier-power-evaluation
description: Use when assessing whether a trained CLOUDe classifier retains high power
  (fraction of correctly predicted Redundant/Unique labels) on new OU-simulated test
  data, including robustness checks under alternative evolutionary scenarios such
  as a distinct ancestral optimum theta0.
when_to_use_negative:
- The input is empirical expression data with unknown true classes — power cannot
  be computed without simulated ground-truth Redundant/Unique labels
- The goal is estimating evolutionary parameters (theta1, theta2, log10(sigmaSq/(2*alpha)))
  rather than classifying deletion events — use the CLOUDePredict* functions instead
- Only one architecture is of interest and no baseline comparison is needed — a full
  five-method power evaluation is unnecessary
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0203
tools:
- name: CLOUDe
  role: R package providing GenerateTrainingData, CLOUDeClassifyNN/XGB/RF/SVM, and
    output files used to simulate test data and score classification power
  repo: https://github.com/anddssan/CLOUDe
- name: tensorflow
  role: Backend for the keras neural network classifier (CLOUDeClassifyNN) whose power
    is the benchmark result
- name: xgboost
  role: Extreme gradient boosting classifier (CLOUDeClassifyXGB) evaluated in the
    power comparison
- name: ranger
  role: Random forest implementation underlying CLOUDeClassifyRF
- name: liquidSVM
  role: Support vector machine implementation underlying CLOUDeClassifySVM (pinned
    to version 1.2.0)
- name: mvtnorm
  role: Multivariate normal sampling for OU-process simulation of training/test expression
    data
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbae002
    title: Predicting evolutionary targets and parameters of gene deletion from expression
      data
  source_content_hash: sha256:a7b1e267b6ad44d4aa9d5e6056ca5e4e29e8233cf7cf6f3702135a33115e84a2
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
  - xgboost
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# machine-learning-classifier-power-evaluation

## Summary

Evaluate classification power of CLOUDe's four machine learning architectures (NN, XGB, RF, SVM) versus a maximum-likelihood LRT baseline in distinguishing 'Redundant' from 'Unique' gene deletion classes on Ornstein–Uhlenbeck-simulated expression test sets.

## When to use

Use when assessing whether a trained CLOUDe classifier retains high power (fraction of correctly predicted Redundant/Unique labels) on new OU-simulated test data, including robustness checks under alternative evolutionary scenarios such as a distinct ancestral optimum theta0.

## When NOT to use

- The input is empirical expression data with unknown true classes — power cannot be computed without simulated ground-truth Redundant/Unique labels
- The goal is estimating evolutionary parameters (theta1, theta2, log10(sigmaSq/(2*alpha))) rather than classifying deletion events — use the CLOUDePredict* functions instead
- Only one architecture is of interest and no baseline comparison is needed — a full five-method power evaluation is unnecessary

## Inputs

- OU-simulated space-delimited test data file with N+1 rows and 3m columns (derived, survived, lost expression blocks per condition; header eD1..eDm eS1..eSm eL1..eLm)
- Training features file (training_prefix.features) and one-hot class labels (training_prefix.classes) output by GenerateTrainingData
- Testing feature file (testing_prefix.features) extracted with the GenerateTrainingData-style feature extraction
- Trained CLOUDe model files (e.g., training_prefix.nn.num_layers.classifier.hdf5, training_prefix.xgb.max_depth.classifier_cv)

## Outputs

- Per-deletion-event predicted classes (test_prefix.nn.num_layers.classifications, test_prefix.xgb.max_depth.classifications, test_prefix.rf.classifications, test_prefix.svm.classifications)
- Per-deletion-event predicted class probabilities (test_prefix.*.probabilities files)
- Per-method classification power/accuracy table (e.g., classifier_accuracies.csv) comparing NN, XGB, RF, SVM, and LRT

## How to apply

Generate class-balanced OU-simulated data with GenerateTrainingData (e.g., m=6, Nk=20000, logTheta in [0,5], logAlpha in [0,3], logSigmaSq in [-2,3], bounded by an empirical expression file), then produce an independent test set of ~2000 observations under the same parameter ranges — or under an alternative scenario where theta0 differs from theta1 and theta2. Apply the trained classifiers via CLOUDeClassifyNN(training_prefix, testing_prefix, num_layers), CLOUDeClassifyXGB, CLOUDeClassifyRF, and CLOUDeClassifySVM, collecting per-event predicted classes and probabilities from the test_prefix.*.classifications and test_prefix.*.probabilities files. Compute power for each method as the fraction of simulated events assigned to the correct 'Redundant' or 'Unique' class, and optionally fit a maximum-likelihood LRT classifier on the same test data as a baseline. On the paper's balanced test set, expect roughly 97.90% (NN), 96.10% (XGB), 93.60% (RF), 90.45% (SVM), and 85.75% (LRT), with the NN performing best.

## Related tools

- **CLOUDe** (R package providing GenerateTrainingData, CLOUDeClassifyNN/XGB/RF/SVM, and output files used to simulate test data and score classification power) — https://github.com/anddssan/CLOUDe
- **tensorflow** (Backend for the keras neural network classifier (CLOUDeClassifyNN) whose power is the benchmark result)
- **xgboost** (Extreme gradient boosting classifier (CLOUDeClassifyXGB) evaluated in the power comparison)
- **ranger** (Random forest implementation underlying CLOUDeClassifyRF)
- **liquidSVM** (Support vector machine implementation underlying CLOUDeClassifySVM (pinned to version 1.2.0))
- **mvtnorm** (Multivariate normal sampling for OU-process simulation of training/test expression data)

## Examples

```
CLOUDeClassifyNN(training_prefix, testing_prefix, num_layers)  # then compare test_prefix.nn.num_layers.classifications to true simulated classes
```

## Evaluation signals

- Per-method power equals the fraction of test events whose predicted label matches the true simulated class; on the balanced 2000-observation set expect NN ~97.90%, XGB ~96.10%, RF ~93.60%, SVM ~90.45%, LRT ~85.75%
- Output files test_prefix.nn.num_layers.classifications and test_prefix.nn.num_layers.probabilities exist and contain one prediction/probability per test deletion event
- Test data conform to the CLOUDe input format: space-delimited, N+1 rows, 3m columns with header eD1..eDm eS1..eSm eL1..eLm and absolute log10(x+1) expression values
- Under the alternative theta0 scenario, CLOUDe power remains high (predictions agree strongly with true simulated classes), as reported in Supplementary Fig. S3B
- Class-balanced design is preserved (Nk observations per class in training; balanced test set) so accuracy is not confounded by class imbalance

## Limitations

- Hyperparameter tuning of the neural network and XGB models via ClassifierCVnn/ClassifierCVxgb is time-consuming; power evaluation requires these models to be trained beforehand (RF and SVM train on-the-fly)
- Power estimates depend on simulation parameter ranges (logTheta, logAlpha, logSigmaSq bounds); results may not transfer to expression data outside the empirical file's range
- liquidSVM must be pinned to version 1.2.0 for reproducibility

## Evidence

- [results] On the balanced simulated test set (2000 observations), classification accuracy is 97.90% for the NN, 96.10% for XGB, 93.60% for RF, 90.45% for SVM, and 85.75% for the LRT, with the NN achieving the best performance.: [evidence span withheld — anchor status: not_found]
- [supplementary] When tested on data generated under the alternative scenario with a distinct ancestral optimum theta0, CLOUDe still achieves high power in differentiating between 'redundant' and 'unique' classes (Supplementary Fig. S3B).: [evidence span withheld — anchor status: not_found]
- [methods] GenerateTrainingData function signature from the README methods section.: "GenerateTrainingData(m, Nk, training_prefix, logThetaMin, logThetaMax, logAlphaMin, logAlphaMax, logSigmaSqMin, logSigmaSqMax, empiricalFile)"
- [readme] README description of the classification output files.: "The CLOUDeClassifyNN(), CLOUDeClassifyXGB(), CLOUDeClassifyRF(), and CLOUDeClassifySVM() functions output predicted classes and probabilities for each deletion event in the test dataset"
- [readme] README input file format specification.: "A space-delimited file with N+1 rows and 3m columns, where N is the number of genes and m is the number of conditions"
