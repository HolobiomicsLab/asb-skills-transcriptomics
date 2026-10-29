---
name: tissue-specific-dataset-handling
description: Use when benchmarking or deploying m6A site predictors on tissue-specific
  41-nt RNA segment datasets from human, mouse, or rat, or when cross-tissue/cross-species
  comparisons require consistent dataset splits and encodings.
when_to_use_negative:
- Input sequences are not 41-nt tissue-specific RNA segments centered on a candidate
  m6A site (the model is trained only on this window and tissue scope)
- Species or tissue requested is outside the supported set (Human brain/liver/kidney/heart/testis,
  Mouse brain/liver/kidney, Rat brain/liver/kidney)
- Sequences have not been CD-HIT filtered below 80% similarity, risking inflated performance
  from near-duplicate training/testing samples
edam_operation: http://edamontology.org/operation_0433
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3512
tools:
- name: im6A-TS-CNN
  role: CNN-based predictor applied per species-tissue combination on one-hot encoded
    41-nt RNA segments, with low/normal/high threshold options
  repo: https://github.com/liukeweiaway/DeepM6A_cnn
- name: Keras in TensorFlow 2.0
  role: Implementation framework used to perform the predictions under Python 3.6
  repo: https://github.com/liukeweiaway/DeepM6A_cnn
- name: Python 3.6
  role: Runtime environment required by the im6A-TS-cnn command-line script (pandas
    1.0.3, tensorflow-cpu 2.1.0, numpy 1.18.3)
  repo: https://github.com/liukeweiaway/DeepM6A_cnn
- name: CD-HIT
  role: Sequence clustering program used to ensure dataset sequence similarity is
    less than 80% before model training
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.omtn.2020.07.034
    title: 'im6A-TS-CNN: Identifying the N6-Methyladenine Site in Multiple Tissues
      by Using the Convolutional Neural Network'
  source_content_hash: sha256:56e4875ea455313cca1a7b94ed4d752e874e5c79555988e5d0ec82ee9df15b86
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
  edam_operation: http://edamontology.org/operation_0433
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# tissue-specific-dataset-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Handling the 11 species-tissue m6A benchmark datasets (human brain/kidney/liver, mouse brain/heart/kidney/liver/testis, rat brain/kidney/liver) built by Dao et al., including CD-HIT filtering to <80% sequence similarity, one-hot encoding, and threshold-specific deployment of im6A-TS-CNN.

## When to use

Use when benchmarking or deploying m6A site predictors on tissue-specific 41-nt RNA segment datasets from human, mouse, or rat, or when cross-tissue/cross-species comparisons require consistent dataset splits and encodings.

## When NOT to use

- Input sequences are not 41-nt tissue-specific RNA segments centered on a candidate m6A site (the model is trained only on this window and tissue scope)
- Species or tissue requested is outside the supported set (Human brain/liver/kidney/heart/testis, Mouse brain/liver/kidney, Rat brain/liver/kidney)
- Sequences have not been CD-HIT filtered below 80% similarity, risking inflated performance from near-duplicate training/testing samples

## Inputs

- FASTA-format 41-nt RNA segment sequences (positive and negative m6A site examples)
- Tissue-specific benchmark datasets: h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, r_l
- User-selected species (-s Human/Mouse/Rat), tissue (-t), and threshold (-ts low/normal/high)

## Outputs

- One-hot encoded 4-l dimensional feature vectors per RNA segment
- Per-tissue predictions written to an output.html results file
- Performance metrics (Sn, Sp, Acc, MCC, AUC) for 5-fold cross-validation and independent tests, e.g. h_b AUC 0.8029/0.8056, r_k AUC 0.9017/0.9077

## How to apply

Collect the 11 tissue-specific benchmark datasets (h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, r_l) of 41-nt positive/negative RNA sequences from Dao et al., then run CD-HIT so sequence similarity is less than 80%. Encode each RNA segment with the one-hot scheme (A=(1,0,0,0), U=(0,1,0,0), C=(0,0,1,0), G=(0,0,0,1)) to produce 4-l dimensional vectors. When invoking the pretrained command-line tool, select the species (Human/Mouse/Rat) and the tissue matching the dataset (brain/liver/kidney/heart/testis as available), and choose one of three specificity thresholds (low/normal/high) based on the desired specificity level. Evaluate with both 5-fold cross-validation and independent dataset tests to confirm stability across tissues, reporting Sn, Sp, Acc, MCC, and AUC.

## Related tools

- **im6A-TS-CNN** (CNN-based predictor applied per species-tissue combination on one-hot encoded 41-nt RNA segments, with low/normal/high threshold options) — https://github.com/liukeweiaway/DeepM6A_cnn
- **Keras in TensorFlow 2.0** (Implementation framework used to perform the predictions under Python 3.6) — https://github.com/liukeweiaway/DeepM6A_cnn
- **Python 3.6** (Runtime environment required by the im6A-TS-cnn command-line script (pandas 1.0.3, tensorflow-cpu 2.1.0, numpy 1.18.3)) — https://github.com/liukeweiaway/DeepM6A_cnn
- **CD-HIT** (Sequence clustering program used to ensure dataset sequence similarity is less than 80% before model training)

## Examples

```
python im6A-TS-cnn.py -i sequence.txt -o ccc.html -s Human -t brain -ts normal
```

## Evaluation signals

- CD-HIT output confirms maximum pairwise sequence similarity <80% within each tissue dataset
- Encoded matrix has width 4 x 41 = 164 per sample, matching the fully connected layer of 164 units in the CNN
- Most per-tissue AUCs exceed 0.8 in both 5-fold cross-validation and independent tests (e.g. r_k: 0.9017 and 0.9077)
- Cross-validation and independent-test metrics are close to each other, indicating the model is stable for identifying m6A sites
- Command-line run writes output.html under the results folder for the chosen species/tissue/threshold combination

## Limitations

- The command-line tool supports only Mouse(brain/liver/kidney), Rat(brain/liver/kidney), and Human(brain/liver/kidney/heart/testis); other tissue-species pairs are out of scope
- Three fixed threshold options (low/normal/high) are offered based on differences in specificity; detailed threshold performance is deferred to supplementary Table S1
- Preprocessing choices (filter steps, threshold selection) are not named per section in the discussion, so per-dataset provenance may need to be re-derived

## Evidence
