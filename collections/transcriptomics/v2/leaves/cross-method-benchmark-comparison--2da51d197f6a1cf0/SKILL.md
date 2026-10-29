---
name: cross-method-benchmark-comparison
description: Use when a newly proposed sequence-classification method must be validated
  against a published baseline on shared benchmark splits, and per-tissue metric deltas
  (e.g., AUC) must be tabulated with significance markers.
when_to_use_negative:
- The input is raw sequencing data needing m6A-site calling rather than benchmark
  comparison of trained classifiers
- No shared benchmark datasets or identical splits exist between the new method and
  the baseline, making AUC deltas incomparable
- The task targets m6A prediction in species or tissues outside human/mouse/rat brain,
  liver, kidney, heart, testis coverage
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3512
tools:
- name: im6A-TS-CNN
  role: CNN-based method under evaluation for tissue-specific m6A identification
  repo: https://github.com/liukeweiaway/DeepM6A_cnn
- name: iRNA-m6A
  role: SVM-based baseline whose AUC values are compared against im6A-TS-CNN
- name: Keras in TensorFlow 2.0
  role: Framework used to train and evaluate the im6A-TS-CNN model on shared splits
- name: Python 3.6
  role: Runtime environment for performing the predictions and comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.omtn.2020.07.034
    title: 'im6A-TS-CNN: Identifying the N6-Methyladenine Site in Multiple Tissues
      by Using the Convolutional Neural Network'
  source_content_hash: sha256:a09d9eb7d7515a26014439a2447f4afb0dc1693db55bb510ada2dbd1346cc378
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
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# cross-method-benchmark-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A head-to-head benchmarking skill that evaluates a new predictor (im6A-TS-CNN) against an established baseline (iRNA-m6A, SVM-based) on identical tissue-specific m6A datasets, computing per-tissue AUC deltas under both 5-fold cross-validation and independent testing.

## When to use

Use when a newly proposed sequence-classification method must be validated against a published baseline on shared benchmark splits, and per-tissue metric deltas (e.g., AUC) must be tabulated with significance markers.

## When NOT to use

- The input is raw sequencing data needing m6A-site calling rather than benchmark comparison of trained classifiers
- No shared benchmark datasets or identical splits exist between the new method and the baseline, making AUC deltas incomparable
- The task targets m6A prediction in species or tissues outside human/mouse/rat brain, liver, kidney, heart, testis coverage

## Inputs

- 11 tissue-specific benchmark datasets of 41-nt positive/negative RNA sequences (h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, r_l)
- FASTA-format input sequences for command-line prediction
- Baseline iRNA-m6A (SVM) AUC values on matching train/test splits

## Outputs

- comparison_table.csv with per-tissue AUC deltas for cross-validation and independent tests
- Per-tissue performance metrics (Sn, Sp, Acc, MCC, AUC) for both 5-fold cross-validation and independent tests
- ROC curves per tissue and test regime

## How to apply

Run both methods on the same 11 tissue-specific benchmark datasets (h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, r_l) of 41-nt sequences, using identical training and testing splits for 5-fold cross-validation and independent tests. Encode each 41-nt RNA segment one-hot (A=(1,0,0,0), U=(0,1,0,0), C=(0,0,1,0), G=(0,0,0,1)) before feeding im6A-TS-CNN (200 filters, kernel size 6, max-pooling 4, dropout 0.16, FC 164→32, softmax, SGD lr 0.001, 2,000 epochs, early stopping patience 50, min_delta 0.001) implemented in Keras/TensorFlow 2.0 under Python 3.6. Collect iRNA-m6A (SVM-based, Dao et al.) AUC values on the same splits. Compute per-tissue AUC deltas (new method minus baseline) for both test regimes and record significance markers as reported. A correct comparison reproduces the reported pattern: the new method wins in AUC everywhere except mouse and rat brain (e.g., m_b cross-validation delta −0.0026; r_b independent-test delta −0.0452), with results written to comparison_table.csv.

## Related tools

- **im6A-TS-CNN** (CNN-based method under evaluation for tissue-specific m6A identification) — https://github.com/liukeweiaway/DeepM6A_cnn
- **iRNA-m6A** (SVM-based baseline whose AUC values are compared against im6A-TS-CNN)
- **Keras in TensorFlow 2.0** (Framework used to train and evaluate the im6A-TS-CNN model on shared splits)
- **Python 3.6** (Runtime environment for performing the predictions and comparison)

## Examples

```
python im6A-TS-cnn.py -i sequence.txt -o ccc.html -s Human -t brain -ts normal
```

## Evaluation signals

- Reproduced m_b cross-validation AUC delta equals −0.0026 (im6A-TS-CNN 0.8705 vs iRNA-m6A 0.8731)
- Reproduced r_b independent-test AUC delta equals −0.0452 (0.8516 vs 0.8968) while the cross-validation delta is +0.0187 with significance marker
- im6A-TS-CNN AUC exceeds iRNA-m6A in every tissue except mouse and rat brain in both test regimes
- r_k shows the highest AUC (0.9017 cross-validation, 0.9077 independent test); most tissue AUCs exceed 0.8
- comparison_table.csv contains deltas for all 11 datasets under both 5-fold cross-validation and independent tests

## Limitations

- iRNA-m6A is better for mouse and rat brain AUC, so the comparison does not universally favor the CNN
- Datasets must be filtered with CD-HIT to <80% sequence similarity before comparison to avoid inflation
- Threshold-dependent performance details are deferred to supplementary Table S1, not the main comparison table

## Evidence

- [results] brain deltas: "Except for the identification of the m6A sites from the brain of mouse and rat, im6A-TS-CNN outperforms iRNA-m6A for the identification of m6A sites"
- [results] m_b delta: [evidence span withheld — anchor status: not_found]
- [results] comparison protocol: [evidence span withheld — anchor status: not_found]
