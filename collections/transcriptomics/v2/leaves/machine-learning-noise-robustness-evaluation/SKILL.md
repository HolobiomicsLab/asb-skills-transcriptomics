---
name: machine-learning-noise-robustness-evaluation
description: Use when assessing robustness of a trained CLOUDe classifier (or similar
  expression-based ML classifier) by re-classifying noise-perturbed test feature sets
  and tabulating classification power versus noise level per method.
when_to_use_negative:
- The goal is predicting continuous evolutionary parameters (theta1, theta2, log10(sigmaSq/(2*alpha)))
  rather than classifying 'Redundant' vs 'Unique' — use CLOUDePredict* functions instead
- No noise-perturbed test sets exist and the goal is a single clean-data classification
  run rather than a robustness sweep
- Input is raw expression data not yet converted to the space-delimited p=3m feature
  format with derived/survived/lost gene columns expected by CLOUDe
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0203
tools:
- name: CLOUDe
  role: R package supplying the classification functions (CLOUDeClassifyNN/XGB/RF/SVM)
    applied to noise-perturbed test sets
  repo: https://github.com/anddssan/CLOUDe
- name: tensorflow
  role: Backend for training and applying the CLOUDe neural network classifier
- name: keras
  role: R interface used to fit the multi-layer neural network classifier
- name: xgboost
  role: Extreme gradient boosting architecture compared in the robustness sweep via
    CLOUDeClassifyXGB
- name: ranger
  role: Random forest backend used by CLOUDeClassifyRF on each noise test set
- name: liquidSVM
  role: Support vector machine backend used by CLOUDeClassifySVM on each noise test
    set
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae002
    title: Predicting evolutionary targets and parameters of gene deletion from expression
      data
  source_content_hash: sha256:04a13f91191fc5fb8ffcb59c0bd99decb99478cb407966fc4eb98c08b0c00ab0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/anddssan/CLOUDe
  tools:
  - CLOUDe
  - R
  - keras
  - liquidSVM
  - ranger
  - tensorflow
  - xgboost
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# machine-learning-noise-robustness-evaluation

## Summary

Evaluates how added noise in test expression data (proxying shallower sequencing depth) degrades CLOUDe's classification power between 'Redundant' and 'Unique' classes, comparing NN, XGB, RF, and SVM architectures across four noise SD scenarios (0.001, 0.01, 0.1, 1).

## When to use

Use when assessing robustness of a trained CLOUDe classifier (or similar expression-based ML classifier) by re-classifying noise-perturbed test feature sets and tabulating classification power versus noise level per method.

## When NOT to use

- The goal is predicting continuous evolutionary parameters (theta1, theta2, log10(sigmaSq/(2*alpha))) rather than classifying 'Redundant' vs 'Unique' — use CLOUDePredict* functions instead
- No noise-perturbed test sets exist and the goal is a single clean-data classification run rather than a robustness sweep
- Input is raw expression data not yet converted to the space-delimited p=3m feature format with derived/survived/lost gene columns expected by CLOUDe

## Inputs

- Fitted CLOUDe model files under a training_prefix (e.g. training_prefix.nn.num_layers.classifier.hdf5 in TensorFlow format) from ClassifierCVnn/ClassifierCVxgb
- Noise-perturbed test feature files (SD 0.001, 0.01, 0.1, 1) with known 'Redundant'/'Unique' labels, formatted as space-delimited p=3m expression features output by GenerateTrainingData()

## Outputs

- Classifications files per method and noise level (test_prefix.nn.num_layers.classifications, test_prefix.xgb.max_depth.classifications, test_prefix.rf.classifications, test_prefix.svm.classifications)
- Probabilities files per method and noise level (test_prefix.nn.num_layers.probabilities, etc.)
- noise_robustness_results.csv tabulating classification power vs. noise SD per method

## How to apply

Load CLOUDe in R via source("CLOUDe.R") and point to the training prefix containing fitted models plus the noise-perturbed test feature files (SD 0.001, 0.01, 0.1, 1) previously generated with GenerateTrainingData(). For each noise level, run CLOUDeClassifyNN(training_prefix, testing_prefix, num_layers=2) with the two-hidden-layer neural network, and CLOUDeClassifyXGB(training_prefix, testing_prefix, max_depth=4), CLOUDeClassifyRF(training_prefix, testing_prefix), and CLOUDeClassifySVM(training_prefix, testing_prefix) for the other predictors. Each function writes classifications and probabilities files per test set (e.g. test_prefix.nn.num_layers.probabilities). Compute classification power (accuracy) from the probabilities files by comparing predicted 'Redundant'/'Unique' labels against known labels, then tabulate power vs. noise SD per method into noise_robustness_results.csv. The rationale is that added noise proxies shallower sequencing depth, so power retention across SD 0.001 to 1 quantifies practical robustness; the paper's finding is that NN and XGB retain high power with NN best overall.

## Related tools

- **CLOUDe** (R package supplying the classification functions (CLOUDeClassifyNN/XGB/RF/SVM) applied to noise-perturbed test sets) — https://github.com/anddssan/CLOUDe
- **tensorflow** (Backend for training and applying the CLOUDe neural network classifier)
- **keras** (R interface used to fit the multi-layer neural network classifier)
- **xgboost** (Extreme gradient boosting architecture compared in the robustness sweep via CLOUDeClassifyXGB)
- **ranger** (Random forest backend used by CLOUDeClassifyRF on each noise test set)
- **liquidSVM** (Support vector machine backend used by CLOUDeClassifySVM on each noise test set)

## Examples

```
source("CLOUDe.R"); CLOUDeClassifyNN(training_prefix, testing_prefix, num_layers=2)
```

## Evaluation signals

- Classification power (accuracy vs. known 'Redundant'/'Unique' labels) remains high across all four noise SDs (0.001, 0.01, 0.1, 1) for NN and XGB
- NN achieves the best overall classification power among the four architectures, matching the paper's reported result
- Each method produces paired classifications and probabilities files (e.g. test_prefix.nn.num_layers.classifications and .probabilities) for every noise level
- noise_robustness_results.csv contains a power value per method per noise SD with no missing cells
- Hyperparameters used match the cross-validation optima: two hidden layers (num_layers=2) for NN and max_depth=4 for XGB

## Limitations

- Robustness is only assessed for added-noise perturbations proxying shallower sequencing depth; other noise modalities are not covered
- RF and SVM training occur on-the-fly at classification time, so no pre-fitted model files exist for these methods
- Results depend on training data generated by GenerateTrainingData from an Ornstein–Uhlenbeck process with parameter ranges supplied by the user

## Evidence

- [methods] Noise robustness workflow steps: [evidence span withheld — anchor status: not_found]
- [results] Robustness finding: [evidence span withheld — anchor status: not_found]
- [abstract] Research question framing: [evidence span withheld — anchor status: not_found]
- [readme] Classification command signatures: "CLOUDeClassifyXGB(training_prefix, testing_prefix, max_depth)   # extreme gradient boosting"
- [readme] Output file specification: "output predicted classes and probabilities for each deletion event in the test dataset to the respective files"
