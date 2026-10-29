---
name: k-fold-cross-validation-evaluation
description: Use when benchmarking a sequence classifier on balanced positive/negative
  RNA 41-nt segments and you need both 5-fold cross-validation metrics and an independent-test
  comparison of Sn, Sp, Acc, MCC, and AUC.
when_to_use_negative:
- Cross-species or cross-tissue generalization is the goal — the article performs
  that as a separate cross-species/cross-tissue validation workflow, not within the
  per-tissue 5-fold CV
- Sequences have not been de-duplicated by CD-HIT to less than 80% similarity — inflated
  CV metrics will result
- The dataset is unbalanced or the task is regression rather than balanced binary
  m6A-site classification
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0099
tools:
- name: im6A-TS-CNN
  role: CNN classifier whose 11 tissue-specific models are evaluated via 5-fold cross-validation
    and independent testing
  repo: https://github.com/liukeweiaway/DeepM6A_cnn
- name: Keras in TensorFlow 2.0
  role: Framework used to build and train the tissue-specific CNNs under Python 3.6
- name: iRNA-m6A
  role: Existing SVM-based method used as the comparison baseline in the same 5-fold
    cross-validation and independent test
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1016/j.omtn.2020.07.034
    title: 'im6A-TS-CNN: Identifying the N6-Methyladenine Site in Multiple Tissues
      by Using the Convolutional Neural Network'
  source_content_hash: sha256:043e57aa3ba4594fae34467472777113a8d000009392f8544e85da91d3d79e7b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/liukeweiaway/DeepM6A_cnn
  repo_url: https://github.com/liukeweiaway/DeepM6A_cnn
  tools:
  - Keras in TensorFlow 2.0
  - Python 3.6
  - iRNA-m6A
  - im6A-TS-CNN
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# k-fold-cross-validation-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates a tissue-specific CNN classifier (im6A-TS-CNN) via 5-fold cross-validation on 41-nt m6A benchmark datasets, reporting Sn, Sp, Acc, MCC, and AUC per species-tissue combination (h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, r_l) alongside an independent-test comparison.

## When to use

Use when benchmarking a sequence classifier on balanced positive/negative RNA 41-nt segments and you need both 5-fold cross-validation metrics and an independent-test comparison of Sn, Sp, Acc, MCC, and AUC.

## When NOT to use

- Cross-species or cross-tissue generalization is the goal — the article performs that as a separate cross-species/cross-tissue validation workflow, not within the per-tissue 5-fold CV
- Sequences have not been de-duplicated by CD-HIT to less than 80% similarity — inflated CV metrics will result
- The dataset is unbalanced or the task is regression rather than balanced binary m6A-site classification

## Inputs

- 11 balanced species-tissue m6A benchmark datasets of 41-nt RNA sequences (h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, r_l) with training and independent testing splits
- One-hot encoded 41-nt segments (A=(1,0,0,0), U=(0,1,0,0), C=(0,0,1,0), G=(0,0,0,1)) as 164-dimensional vectors

## Outputs

- Per-dataset Sn, Sp, Acc, MCC, and AUC for 5-fold cross-validation and independent test (e.g., written to performance_metrics.csv)
- ROC curves for the 5-fold cross-validation test and independent test

## How to apply

Split each tissue-specific training set (e.g., h_b with 4,605 positives and 4,605 negatives) into 5 folds and run 5-fold cross-validation on the Keras/TensorFlow 2.0 CNN (200 filters, kernel size 6, max-pooling 4, dropout 0.16, SGD lr 0.001, categorical cross-entropy, up to 2,000 epochs with early stopping patience 50 and min_delta 0.001). Record Sn, Sp, Acc, MCC, and AUC per fold and aggregate per species-tissue combination. Separately evaluate the trained model on the held-out independent testing set and compile both metric sets into a single table. Judge stability by comparing 5-fold vs independent-test values (the paper found the method 'stable' when the two agree) and expect most AUCs above 0.8 (e.g., h_b 5-fold AUC 0.8029; r_k 5-fold AUC 0.9017, independent-test AUC 0.9077).

## Related tools

- **im6A-TS-CNN** (CNN classifier whose 11 tissue-specific models are evaluated via 5-fold cross-validation and independent testing) — https://github.com/liukeweiaway/DeepM6A_cnn
- **Keras in TensorFlow 2.0** (Framework used to build and train the tissue-specific CNNs under Python 3.6)
- **iRNA-m6A** (Existing SVM-based method used as the comparison baseline in the same 5-fold cross-validation and independent test)

## Examples

```
python im6A-TS-cnn.py -i sequence.txt -o ccc.html -s Human -t brain -ts normal
```

## Evaluation signals

- h_b 5-fold CV metrics match the reported Sn 75.35%, Sp 69.71%, Acc 72.53%, MCC 0.4523, AUC 0.8029
- r_k achieves the highest AUC: 0.9017 in 5-fold CV and 0.9077 in the independent test
- Most of the 11 tissue AUCs exceed 0.8 in both the 5-fold cross-validation and independent test
- 5-fold CV and independent-test metrics are close in magnitude, indicating model stability
- Compared AUCs against iRNA-m6A match sign/magnitude, e.g., m_b cross-validation difference −0.0026

## Limitations

- im6A-TS-CNN underperforms iRNA-m6A for mouse brain in cross-validation (AUC 0.8705 vs 0.8731) and rat brain in the independent test (0.8516 vs 0.8968)
- Threshold-dependent performance details were deferred to supplementary Table S1 rather than the main table
- Metrics depend on the CD-HIT (<80% similarity) filtered benchmark; performance on non-filtered or novel data is not established

## Evidence

- [results] AUC mostly above 0.8: [evidence span withheld — anchor status: not_found]
