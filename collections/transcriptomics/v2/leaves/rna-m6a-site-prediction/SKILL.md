---
name: rna-m6a-site-prediction
description: Use when you need to identify m6A modification sites from RNA sequences
  for specific tissues (brain, liver, kidney, heart, testis) in human, mouse, or rat,
  starting from FASTA sequences or the 11 benchmark datasets (h_b, h_k, h_l, m_b,
  m_h, m_k, m_l, m_t, r_b, r_k, r_l).
when_to_use_negative:
- The query is outside the three supported species or the five supported tissue types
  (e.g., plant or yeast m6A prediction).
- Sequences have not been CD-HIT-filtered to <80% similarity and you expect benchmark-level
  metrics, since redundancy inflates cross-validation performance.
- You need quantitative threshold-performance data not in Table S1 or versioned details
  of the tool not stated by the article (e.g., no version number for i6mA-TS-CNN is
  given).
edam_operation: http://edamontology.org/operation_0262
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3474
tools:
- name: im6A-TS-CNN
  role: 'The CNN-based predictor itself: one-hot encoded 41-nt RNA segments are classified
    as m6A-positive/negative per tissue.'
  repo: https://github.com/liukeweiaway/DeepM6A_cnn
- name: Keras in TensorFlow 2.0
  role: Framework used to construct and train the tissue-specific CNN classifiers
    (200 filters, kernel size 6, dropout 0.16, SGD lr 0.001).
- name: Python 3.6
  role: Runtime environment (with pandas==1.0.3, tensorflow-cpu==2.1.0, numpy==1.18.3
    per the README) for training and the command-line tool.
- name: iRNA-m6A
  role: SVM-based baseline (Dao et al.) used for head-to-head AUC comparison on the
    same training/testing splits.
- name: CD-HIT
  role: Filtering program used to enforce <80% sequence similarity within benchmark
    datasets before encoding and training.
provenance:
  source_task_ids:
  - task_005
  - task_004
  - task_006
  - task_003
  source_papers:
  - doi: 10.1016/j.omtn.2020.07.034
    title: 'im6A-TS-CNN: Identifying the N6-Methyladenine Site in Multiple Tissues
      by Using the Convolutional Neural Network'
  source_content_hash: sha256:4b6fcb0bd05748297abd7032801e88cbc5b88a03e89c2f79a5a3bebddc787bc4
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
  - CD-HIT
  - Keras in TensorFlow 2.0
  - Python 3.6
  - iRNA-m6A
  - im6A-TS-CNN
  edam_operation: http://edamontology.org/operation_0262
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# rna-m6a-site-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A CNN-based skill for predicting N6-methyladenine (m6A) sites in tissue-specific RNA sequences across human, mouse, and rat, using one-hot encoded 41-nt segments classified by im6A-TS-CNN implemented in Keras/TensorFlow 2.0.

## When to use

Use when you need to identify m6A modification sites from RNA sequences for specific tissues (brain, liver, kidney, heart, testis) in human, mouse, or rat, starting from FASTA sequences or the 11 benchmark datasets (h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, r_l).

## When NOT to use

- The query is outside the three supported species or the five supported tissue types (e.g., plant or yeast m6A prediction).
- Sequences have not been CD-HIT-filtered to <80% similarity and you expect benchmark-level metrics, since redundancy inflates cross-validation performance.
- You need quantitative threshold-performance data not in Table S1 or versioned details of the tool not stated by the article (e.g., no version number for i6mA-TS-CNN is given).

## Inputs

- FASTA-format RNA sequence file (input.txt) containing query segments
- 11 tissue-specific benchmark datasets of balanced 41-nt positive/negative RNA sequences (h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, r_l)
- Species choice (Human/Mouse/Rat) and tissue choice (e.g., brain/liver/kidney/heart/testis)

## Outputs

- Per-tissue classification of candidate m6A sites, saved as output.html in the results folder
- Performance metrics per species-tissue combination: Sn, Sp, Acc, MCC, and AUC for 5-fold cross-validation and independent test (e.g., performance_metrics.csv)
- Trained Keras CNN model artifacts per tissue dataset

## How to apply

First, obtain balanced positive/negative 41-nt RNA segments, filtered with CD-HIT to <80% sequence similarity. One-hot encode each segment (A=(1,0,0,0), U=(0,1,0,0), C=(0,0,1,0), G=(0,0,0,1)) into 4-l dimensional vectors. Build the CNN in Keras on TensorFlow 2.0 under Python 3.6: a convolutional layer with 200 filters of kernel size 6, max-pooling of size 4, dropout rate 0.16, fully connected layers of 164 then 32 units with softmax output; train with SGD (learning rate 0.001) and categorical cross-entropy for up to 2,000 epochs with early stopping (patience 50, min_delta 0.001). Evaluate with 5-fold cross-validation on the training split and on a held-out independent test set, recording Sn, Sp, Acc, MCC, and AUC; optionally compare AUC against the SVM-based iRNA-m6A baseline per tissue. For command-line use, run the packaged script with species, tissue, and threshold options (low/normal/high specificity).

## Related tools

- **im6A-TS-CNN** (The CNN-based predictor itself: one-hot encoded 41-nt RNA segments are classified as m6A-positive/negative per tissue.) — https://github.com/liukeweiaway/DeepM6A_cnn
- **Keras in TensorFlow 2.0** (Framework used to construct and train the tissue-specific CNN classifiers (200 filters, kernel size 6, dropout 0.16, SGD lr 0.001).)
- **Python 3.6** (Runtime environment (with pandas==1.0.3, tensorflow-cpu==2.1.0, numpy==1.18.3 per the README) for training and the command-line tool.)
- **iRNA-m6A** (SVM-based baseline (Dao et al.) used for head-to-head AUC comparison on the same training/testing splits.)
- **CD-HIT** (Filtering program used to enforce <80% sequence similarity within benchmark datasets before encoding and training.)

## Examples

```
python im6A-TS-cnn.py -i sequence.txt -o ccc.html -s Human -t brain -ts normal
```

## Evaluation signals

- Most per-tissue AUCs exceed 0.8 in both the 5-fold cross-validation test and the independent test.
- Reproduced h_b metrics match the reported values (e.g., cross-validation Sn 75.35%, Sp 69.71%, Acc 72.53%, MCC 0.4523, AUC 0.8029; independent test AUC 0.8056).
- r_k reaches the highest AUC of 0.9017 (cross-validation) and 0.9077 (independent test); im6A-TS-CNN outperforms iRNA-m6A in AUC for every tissue except mouse and rat brain (e.g., m_b cross-validation delta −0.0026, r_b independent-test delta −0.0452).
- Cross-validation and independent-test results are consistent, indicating stability of the trained models.
- Schema checks: comparison_table.csv contains per-tissue AUC deltas with reported significance markers, and performance_metrics.csv covers all 11 species-tissue combinations.

## Limitations

- iRNA-m6A achieves higher AUC for mouse brain in cross-validation (0.8731 vs 0.8705) and for rat brain in the independent test (0.8968 vs 0.8516), so the CNN is not uniformly superior across all tissues.
- The command-line tool offers only three specificity thresholds (low/normal/high) with full per-threshold performance deferred to supplementary Table S1, and no version number was stated for i6mA-TS-CNN.
- The tool is restricted to brain, liver, kidney, heart (mouse), and testis (mouse) tissues of human, mouse, and rat as covered by the 11 benchmark datasets.

## Evidence
