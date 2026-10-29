---
name: cnn-training-early-stopping
description: Use when training a CNN classifier on one-hot encoded nucleotide sequences
  (e.g., 41-nt m6A windows) and you need a stable training regime with 5-fold cross-validation
  and early stopping.
when_to_use_negative:
- The input is not short nucleotide sequence windows (e.g., already a feature table
  or genomic intervals), since this skill depends on one-hot encoding of 41-nt RNA
  segments
- The goal is baseline SVM classification (e.g., reproducing iRNA-m6A) rather than
  CNN model construction
- Sequence similarity within the dataset has not been reduced below 80% with CD-HIT,
  which the benchmark construction requires
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3869
tools:
- name: im6A-TS-CNN
  role: 'The CNN model being constructed and trained: one-hot encoded input, 200 filters
    of kernel size 6, max-pooling 4, dropout 0.16, fully connected 164 to 32 units,
    softmax output'
  repo: https://github.com/liukeweiaway/DeepM6A_cnn
- name: Keras in TensorFlow 2.0
  role: Framework used to implement, train (SGD, lr 0.001, up to 2,000 epochs), and
    early-stop (patience 50, min_delta 0.001) the CNN
- name: Python 3.6
  role: Runtime environment for the Keras/TensorFlow 2.0 training pipeline
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_003
  source_papers:
  - doi: 10.1016/j.omtn.2020.07.034
    title: 'im6A-TS-CNN: Identifying the N6-Methyladenine Site in Multiple Tissues
      by Using the Convolutional Neural Network'
  source_content_hash: sha256:594920125a0a563df2b9605ead561db9a27ab0e984987ebf966defb9929b77c7
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3869
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cnn-training-early-stopping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train tissue-specific 1D CNN classifiers (as in im6A-TS-CNN) on one-hot encoded 41-nt RNA m6A sequences in Keras/TensorFlow 2.0, using SGD with early stopping (patience 50, min_delta 0.001) to produce per-tissue models evaluated by Sn, Sp, Acc, MCC, and AUC.

## When to use

Use when training a CNN classifier on one-hot encoded nucleotide sequences (e.g., 41-nt m6A windows) and you need a stable training regime with 5-fold cross-validation and early stopping.

## When NOT to use

- The input is not short nucleotide sequence windows (e.g., already a feature table or genomic intervals), since this skill depends on one-hot encoding of 41-nt RNA segments
- The goal is baseline SVM classification (e.g., reproducing iRNA-m6A) rather than CNN model construction
- Sequence similarity within the dataset has not been reduced below 80% with CD-HIT, which the benchmark construction requires

## Inputs

- Tissue-specific RNA m6A benchmark datasets of 41-nt positive/negative FASTA sequences (11 species-tissue sets: h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, r_l)
- One-hot encoded 4-l dimensional sequence vectors for model input

## Outputs

- Trained Keras/TensorFlow 2.0 CNN model artifact per tissue dataset
- 5-fold cross-validation and independent test metrics (Sn, Sp, Acc, MCC, AUC) per tissue

## How to apply

Encode each 41-nt RNA segment with the one-hot scheme (A=(1,0,0,0), U=(0,1,0,0), C=(0,0,1,0), G=(0,0,0,1)) to obtain 4-l dimensional vectors. In Keras on TensorFlow 2.0 under Python 3.6, build a CNN with a convolutional layer of 200 filters (kernel size 6, ReLU), a max-pooling layer of size 4, dropout rate 0.16, then fully connected layers compressing 164 to 32 units with a softmax output. Train with the SGD optimizer (learning rate 0.001) and categorical cross-entropy loss for up to 2,000 epochs, applying early stopping with patience 50 and min_delta 0.001 to prevent overfitting while allowing slow convergence. Use 5-fold cross-validation on the training split and hold out an independent test set; repeat per species-tissue dataset (h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, r_l).

## Related tools

- **im6A-TS-CNN** (The CNN model being constructed and trained: one-hot encoded input, 200 filters of kernel size 6, max-pooling 4, dropout 0.16, fully connected 164 to 32 units, softmax output) — https://github.com/liukeweiaway/DeepM6A_cnn
- **Keras in TensorFlow 2.0** (Framework used to implement, train (SGD, lr 0.001, up to 2,000 epochs), and early-stop (patience 50, min_delta 0.001) the CNN)
- **Python 3.6** (Runtime environment for the Keras/TensorFlow 2.0 training pipeline)

## Examples

```
python im6A-TS-cnn.py -i sequence.txt -o ccc.html -s Human -t brain -ts normal
```

## Evaluation signals

- A trained model is accepted when it reproduces reported tissue-level independent-test performance (e.g., h_b: Sn 75.35%, Sp 69.71%, Acc 72.53%, MCC 0.4523, AUC 0.8029)
- Most tissue AUCs exceed 0.8 in both the 5-fold cross-validation test and independent test (e.g., r_k AUC 0.9017 cross-validation / 0.9077 independent)
- Training terminates via early stopping well before the 2,000-epoch cap, consistent with patience 50 and min_delta 0.001
- Per-tissue metric values match the aggregated 11-dataset validation table across both cross-validation and independent tests

## Limitations

- CNN does not outperform the SVM-based iRNA-m6A for mouse and rat brain AUC (e.g., m_b cross-validation delta −0.0026; r_b independent-test delta −0.0452), so it is not universally superior per tissue
- Threshold performance with different specificity options (low/normal/high) was deferred to supplementary Table S1, so exact threshold behavior is not fully specified in the main text
- The README pins specific dependency versions (python3.6, pandas 1.0.3, tensorflow-cpu 2.1.0, numpy 1.18.3); other environments may not reproduce results

## Evidence

- [abstract] In im6A-TS-CNN, the samples were encoded by using the one-hot encoding scheme.: "In im6A-TS-CNN, the samples were encoded by using the one-hot encoding scheme."
- [methods] in an RNA segment, A is represented as (1,0,0,0), U as (0,1,0,0), C as (0,0,1,0), and G as (0,0,0,1): "in an RNA segment, A is represented as (1,0,0,0), U as (0,1,0,0), C as (0,0,1,0), and G as (0,0,0,1)"
- [results] the Keras in TensorFlow 2.0 under Python 3.6 was used to perform the predictions: "the Keras in TensorFlow 2.0 under Python 3.6 was used to perform the predictions"
- [results] h_b	75.35	69.71	72.53	0.4523	0.8029	75.17	70.20	72.69	0.4543	0.8056: "h_b	75.35	69.71	72.53	0.4523	0.8029	75.17	70.20	72.69	0.4543	0.8056"
- [readme] python im6A-TS-cnn.py -i sequence.txt -o ccc.html -s Human -t brain  -ts normal: "python im6A-TS-cnn.py -i sequence.txt -o ccc.html -s Human -t brain  -ts normal"
