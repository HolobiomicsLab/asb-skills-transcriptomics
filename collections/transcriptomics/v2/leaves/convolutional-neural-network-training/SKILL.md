---
name: convolutional-neural-network-training
description: Use when classifying tissue-specific RNA m6A modification sites from
  41-nt nucleotide sequences and you need a CNN trained on one-hot encoded inputs
  with cross-validation and independent-test evaluation.
when_to_use_negative:
- The task is predicting m6A in only brain tissue across species without tissue specificity
  — a non-tissue-specific predictor such as iRNA-m6A may suffice and cross-tissue
  transfer is out of scope
- Sequences have not been deduplicated — the benchmark requires CD-HIT filtering so
  that sequence similarity is less than 80% before training
- The input is genomic signal data or a feature matrix rather than raw 41-nt RNA segments
  amenable to one-hot encoding
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3512
tools:
- name: im6A-TS-CNN
  role: The CNN-based method trained per tissue to identify m6A sites; provides the
    architecture and command-line interface
  repo: https://github.com/liukeweiaway/DeepM6A_cnn
- name: Keras in TensorFlow 2.0
  role: Framework used to build and train the convolutional neural network
- name: Python 3.6
  role: Runtime environment for training and prediction (pandas 1.0.3, tensorflow-cpu
    2.1.0, numpy 1.18.3)
- name: iRNA-m6A
  role: SVM-based comparison method whose AUCs are benchmarked against the trained
    CNN
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.omtn.2020.07.034
    title: 'im6A-TS-CNN: Identifying the N6-Methyladenine Site in Multiple Tissues
      by Using the Convolutional Neural Network'
  source_content_hash: sha256:866cdfcf129056a3523f8bf86ec83c1a5becc727f423d0c93dbe876352be8d6f
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
  edam_operation: http://edamontology.org/operation_0436
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# convolutional-neural-network-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a 1D CNN (Keras in TensorFlow 2.0 under Python 3.6) to classify 41-nt RNA segments as m6A sites or non-sites across 11 species-tissue combinations, using one-hot encoded sequences and reporting Sn, Sp, Acc, MCC, and AUC under 5-fold cross-validation and independent testing.

## When to use

Use when classifying tissue-specific RNA m6A modification sites from 41-nt nucleotide sequences and you need a CNN trained on one-hot encoded inputs with cross-validation and independent-test evaluation.

## When NOT to use

- The task is predicting m6A in only brain tissue across species without tissue specificity — a non-tissue-specific predictor such as iRNA-m6A may suffice and cross-tissue transfer is out of scope
- Sequences have not been deduplicated — the benchmark requires CD-HIT filtering so that sequence similarity is less than 80% before training
- The input is genomic signal data or a feature matrix rather than raw 41-nt RNA segments amenable to one-hot encoding

## Inputs

- Balanced positive/negative 41-nt RNA segment FASTA/text files for each species-tissue combination (h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, r_l), e.g. h_b with 4,605 positives and 4,605 negatives
- Independent testing sets with matched positive/negative counts (e.g. h_b 4,604/4,604)
- Training hyperparameters: 200 filters, kernel size 6, max-pooling 4, dropout 0.16, FC 164/32, SGD lr 0.001

## Outputs

- Trained per-tissue CNN models (Keras/TF 2.0)
- Table of Sn, Sp, Acc, MCC, AUC for each dataset under 5-fold cross-validation and independent test (e.g. performance_metrics.csv)
- Plotted ROC curves for both test conditions
- HTML-formatted prediction results from the command-line tool

## How to apply

Load the 11 balanced tissue-specific benchmark datasets (h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, r_l) of 41-nt RNA segments, one-hot encoding A=(1,0,0,0), U=(0,1,0,0), C=(0,0,1,0), G=(0,0,0,1) into 4-l dimensional vectors. Build a CNN with a convolutional layer of 200 filters (kernel size 6), max-pooling of size 4, dropout rate 0.16, and fully connected layers of 164 then 32 units, optimized with SGD (learning rate 0.001) and categorical cross-entropy for up to 2,000 epochs with early stopping (patience 50, min_delta 0.001). Run 5-fold cross-validation on each training set, then evaluate on the corresponding independent testing set, recording Sn, Sp, Acc, MCC, and AUC. Optionally compare AUCs against the SVM-based iRNA-m6A baseline and perform cross-species/cross-tissue validation. Accept the result when most AUCs exceed 0.8 (e.g., h_b 5-fold AUC 0.8029, r_k independent-test AUC 0.9077) and values match the reported table.

## Related tools

- **im6A-TS-CNN** (The CNN-based method trained per tissue to identify m6A sites; provides the architecture and command-line interface) — https://github.com/liukeweiaway/DeepM6A_cnn
- **Keras in TensorFlow 2.0** (Framework used to build and train the convolutional neural network)
- **Python 3.6** (Runtime environment for training and prediction (pandas 1.0.3, tensorflow-cpu 2.1.0, numpy 1.18.3))
- **iRNA-m6A** (SVM-based comparison method whose AUCs are benchmarked against the trained CNN)

## Examples

```
python im6A-TS-cnn.py -i sequence.txt -o ccc.html -s Human -t brain -ts normal
```

## Evaluation signals

- Most AUCs exceed 0.8 in both 5-fold cross-validation and independent test; h_b 5-fold AUC should be ~0.8029 and r_k independent-test AUC ~0.9077
- Reproduced per-dataset metrics match Table 1, e.g. h_b: Sn 75.35%, Sp 69.71%, Acc 72.53%, MCC 0.4523 in 5-fold cross-validation
- Dataset balance is preserved: each dataset has equal positive and negative counts (e.g. h_k 2,634/2,634) and CD-HIT-enforced sequence similarity <80%
- Training stops via early stopping (patience 50, min_delta 0.001) before the 2,000-epoch cap
- Cross-validation vs independent-test results are consistent, indicating model stability

## Limitations

- iRNA-m6A slightly outperforms im6A-TS-CNN for mouse brain cross-validation (AUC 0.8731 vs 0.8705) and rat brain independent test (0.8968 vs 0.8516), so the CNN is not uniformly best
- Performance depends on threshold selection; only three thresholds (low/normal/high) are exposed in the command-line tool, with detailed threshold performance deferred to supplementary Table S1
- Support is limited to brain, liver, kidney, heart, and testis tissues of human, mouse, and rat; other species/tissues require retraining

## Evidence

- [methods] One-hot encoding with CNN architecture: "in an RNA segment, A is represented as (1,0,0,0), U as (0,1,0,0), C as (0,0,1,0), and G as (0,0,0,1)"
- [results] Framework: "the Keras in TensorFlow 2.0 under Python 3.6 was used to perform the predictions"
- [results] Benchmark result: "h_b	75.35	69.71	72.53	0.4523	0.8029	75.17	70.20	72.69	0.4543	0.8056"
- [results] Comparison caveat: "Except for the identification of the m6A sites from the brain of mouse and rat, im6A-TS-CNN outperforms iRNA-m6A"
- [methods] Similarity filter: "The CD-HIT program25 was used to make sure that the sequence similarity of the dataset was less than 80%."
- [readme] Repository README: "im6A-TS-cnn is able to identify multi-tissues in Human, Mouse and Rat."
