---
name: cross-species-cross-tissue-model-validation
description: Use when a tissue-specific RNA modification predictor (e.g., im6A-TS-CNN
  on 41-nt m6A segments) must be shown to generalize across species-tissue combinations
  before deployment or comparison.
when_to_use_negative:
- The goal is comparison against another predictor rather than assessing cross-species/cross-tissue
  transfer — use the dedicated iRNA-m6A comparison workflow instead
- Input sequences are not 41-nt m6A benchmark segments from human, mouse, or rat brain/liver/kidney/heart/testis
  tissues
- Only a single species-tissue dataset is available, so no cross-species or cross-tissue
  transfer assessment is possible
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: im6A-TS-CNN
  role: CNN-based m6A site predictor whose cross-species and cross-tissue performance
    is being validated
  repo: https://github.com/liukeweiaway/DeepM6A_cnn
- name: Keras in TensorFlow 2.0
  role: Framework used to train and evaluate the CNN models under the validation protocol
- name: Python 3.6
  role: Runtime environment for the Keras/TensorFlow validation pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.omtn.2020.07.034
    title: 'im6A-TS-CNN: Identifying the N6-Methyladenine Site in Multiple Tissues
      by Using the Convolutional Neural Network'
  source_content_hash: sha256:a15b7bc3fe9618a9a018c1ebd9d0b89c7b8f96ff5cb5a5d660333cb4f9b6c574
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
  - im6A-TS-CNN
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# cross-species-cross-tissue-model-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates that the im6A-TS-CNN m6A-site predictor trained on one species-tissue benchmark transfers effectively to other species (human, mouse, rat) and tissues (brain, liver, kidney, heart, testis), demonstrating method universality.

## When to use

Use when a tissue-specific RNA modification predictor (e.g., im6A-TS-CNN on 41-nt m6A segments) must be shown to generalize across species-tissue combinations before deployment or comparison.

## When NOT to use

- The goal is comparison against another predictor rather than assessing cross-species/cross-tissue transfer — use the dedicated iRNA-m6A comparison workflow instead
- Input sequences are not 41-nt m6A benchmark segments from human, mouse, or rat brain/liver/kidney/heart/testis tissues
- Only a single species-tissue dataset is available, so no cross-species or cross-tissue transfer assessment is possible

## Inputs

- 11 tissue-specific RNA m6A benchmark datasets (41-nt positive and negative sequences for h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, r_l) from the DeepM6A_cnn repository
- Trained im6A-TS-CNN model(s) per species-tissue dataset
- Held-out independent test sets per species-tissue combination

## Outputs

- Aggregated validation table of Sn, Sp, Acc, MCC, and AUC for all 11 species-tissue combinations under both 5-fold cross-validation and independent testing
- ROC curves for cross-validation and independent tests

## How to apply

Load the 11 tissue-specific RNA m6A benchmark datasets (h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, r_l) of 41-nt positive/negative sequences from the DeepM6A_cnn repository. One-hot encode each segment (A=(1,0,0,0), U=(0,1,0,0), C=(0,0,1,0), G=(0,0,0,1)) and train im6A-TS-CNN in Keras/TensorFlow 2.0 under Python 3.6 using 200 convolutional filters of kernel size 6, max-pooling of 4, dropout 0.16, FC layers of 164 and 32 units, SGD (lr 0.001), categorical cross-entropy, and early stopping (patience 50, min_delta 0.001) for up to 2,000 epochs. Evaluate each model with 5-fold cross-validation and on its held-out independent test set, computing Sn, Sp, Acc, MCC, and AUC, then aggregate all 11 species-tissue results into one validation table. Judge success by whether most AUCs exceed 0.8 in both tests and per-dataset values match reported results (e.g., h_b AUC 0.8029/0.8056; r_k AUC 0.9017/0.9077).

## Related tools

- **im6A-TS-CNN** (CNN-based m6A site predictor whose cross-species and cross-tissue performance is being validated) — https://github.com/liukeweiaway/DeepM6A_cnn
- **Keras in TensorFlow 2.0** (Framework used to train and evaluate the CNN models under the validation protocol)
- **Python 3.6** (Runtime environment for the Keras/TensorFlow validation pipeline)

## Examples

```
python im6A-TS-cnn.py -i sequence.txt -o ccc.html -s Human -t brain -ts normal
```

## Evaluation signals

- Most AUCs exceed 0.8 in both the 5-fold cross-validation test and the independent test
- Per-dataset metrics match reported values, e.g., h_b: Sn 75.35%, Sp 69.71%, Acc 72.53%, MCC 0.4523, AUC 0.8029 (cross-validation) and AUC 0.8056 (independent test)
- r_k achieves the highest AUC (0.9017 cross-validation / 0.9077 independent test)
- Cross-validation and independent-test values are close (method stable), indicating no overfitting
- All 11 species-tissue combinations appear in the aggregated validation table with complete Sn/Sp/Acc/MCC/AUC entries

## Limitations

- iRNA-m6A slightly outperformed im6A-TS-CNN for mouse brain in cross-validation (0.8731 vs 0.8705) and for rat brain in the independent test (0.8968 vs 0.8516), so transfer superiority is not uniform across all combinations
- Threshold-specific performance details are deferred to supplementary Table S1 rather than the main text
- Validation is limited to the three species (human, mouse, rat) and five tissues covered by the 11 benchmark datasets

## Evidence
