---
name: roc-auc-performance-evaluation
description: Use when a trained binary classifier (e.g., CNN or SVM on 41-nt RNA segments)
  must be scored on balanced positive/negative test sets and reported via Sn, Sp,
  Acc, MCC, and ROC AUC.
when_to_use_negative:
- Datasets are heavily imbalanced or unlabeled — the balanced positive/negative design
  underpins Sn/Sp/Acc/MCC interpretability here.
- The task is site localization in a full transcriptome rather than classification
  of pre-extracted 41-nt segments.
- Sequences have not passed CD-HIT filtering to <80% similarity, so metrics would
  be inflated by redundant homologs.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: im6A-TS-CNN
  role: CNN predictor whose 5-fold cross-validation and independent-test outputs are
    scored with Sn, Sp, Acc, MCC, and AUC
  repo: https://github.com/liukeweiaway/DeepM6A_cnn
- name: iRNA-m6A
  role: SVM-based baseline evaluated on the same splits for per-tissue AUC comparison
- name: Keras in TensorFlow 2.0
  role: Framework that produces the softmax probabilities from which metrics and ROC
    curves are computed
- name: Python 3.6
  role: Runtime environment for metric computation and ROC plotting
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_005
  source_papers:
  - doi: 10.1016/j.omtn.2020.07.034
    title: 'im6A-TS-CNN: Identifying the N6-Methyladenine Site in Multiple Tissues
      by Using the Convolutional Neural Network'
  source_content_hash: sha256:4ebe5e380d8c8a8850da21166b9ccc91db944e914bd4ad72d619b78f0954441f
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# roc-auc-performance-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluate tissue-specific m6A-site predictors such as im6A-TS-CNN using 5-fold cross-validation and independent-test Sn, Sp, Acc, MCC, and AUC, with ROC curves plotted per species-tissue dataset.

## When to use

Use when a trained binary classifier (e.g., CNN or SVM on 41-nt RNA segments) must be scored on balanced positive/negative test sets and reported via Sn, Sp, Acc, MCC, and ROC AUC.

## When NOT to use

- Datasets are heavily imbalanced or unlabeled — the balanced positive/negative design underpins Sn/Sp/Acc/MCC interpretability here.
- The task is site localization in a full transcriptome rather than classification of pre-extracted 41-nt segments.
- Sequences have not passed CD-HIT filtering to <80% similarity, so metrics would be inflated by redundant homologs.

## Inputs

- Balanced 41-nt positive/negative RNA sequence benchmark datasets per species-tissue (FASTA-like, one-hot encodable)
- Trained model predicted probabilities (softmax) for 5-fold cross-validation folds and independent test sets
- True binary labels (m6A site vs. non-site) for each fold and test set

## Outputs

- Per-dataset metric table of Sn, Sp, Acc, MCC, and AUC for both 5-fold cross-validation and independent tests (e.g., performance_metrics.csv)
- ROC curves per species-tissue combination (Figure 2-style plots)
- Head-to-head AUC comparison table versus a baseline predictor (e.g., comparison_table.csv with per-tissue deltas)

## How to apply

Split each tissue-specific benchmark (h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, r_l) into training and independent testing sets, then run 5-fold cross-validation and evaluate the held-out independent test. One-hot encode 41-nt segments (A=(1,0,0,0), U=(0,1,0,0), C=(0,0,1,0), G=(0,0,0,1)) before prediction, and compute Sn, Sp, Acc, MCC, and AUC from the predicted softmax probabilities against true labels. Sweep the decision threshold (the im6A-TS-CNN command-line offers low/normal/high specificity thresholds) to build ROC curves for both the 5-fold cross-validation test and the independent test, and plot them as in Figure 2. Compare against baselines (e.g., iRNA-m6A) by per-dataset AUC deltas; the paper's validation pattern is that most AUCs exceed 0.8 (e.g., h_b 0.8029 CV / 0.8056 independent; r_k 0.9017 CV / 0.9077 independent).

## Related tools

- **im6A-TS-CNN** (CNN predictor whose 5-fold cross-validation and independent-test outputs are scored with Sn, Sp, Acc, MCC, and AUC) — https://github.com/liukeweiaway/DeepM6A_cnn
- **iRNA-m6A** (SVM-based baseline evaluated on the same splits for per-tissue AUC comparison)
- **Keras in TensorFlow 2.0** (Framework that produces the softmax probabilities from which metrics and ROC curves are computed)
- **Python 3.6** (Runtime environment for metric computation and ROC plotting)

## Examples

```
python im6A-TS-cnn.py -i sequence.txt -o ccc.html -s Human -t brain -ts normal
```

## Evaluation signals

- Most reported AUCs exceed 0.8 in both 5-fold cross-validation and the independent test
- h_b reference values: Sn 75.35%, Sp 69.71%, Acc 72.53%, MCC 0.4523, AUC 0.8029 (CV) and AUC 0.8056 (independent)
- r_k achieves the highest AUC: 0.9017 CV / 0.9077 independent test
- CV and independent-test AUCs for the same dataset differ only slightly (e.g., h_b 0.8029 vs 0.8056), indicating a stable model
- Comparison deltas reproduce the paper's pattern: im6A-TS-CNN beats iRNA-m6A in AUC for every tissue except mouse and rat brain (m_b CV delta −0.0026; r_b independent delta −0.0452)

## Limitations

- iRNA-m6A outperforms im6A-TS-CNN in AUC for mouse and rat brain, so per-tissue superiority is not universal
- Threshold-dependent performance differs (low/normal/high specificity options), with details deferred to supplementary Table S1
- Metrics are tied to the balanced 41-nt benchmark design and CD-HIT <80% similarity filtering; they may not transfer to raw transcriptome-wide scans

## Evidence
