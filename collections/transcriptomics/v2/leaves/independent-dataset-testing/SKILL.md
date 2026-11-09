---
name: independent-dataset-testing
description: Use when a trained model (e.g., im6A-TS-CNN on 41-nt RNA sequences) must
  be evaluated on a separate benchmark test set, and per-dataset Sn/Sp/Acc/MCC/AUC
  values must be compared against cross-validation results.
when_to_use_negative:
- You only need within-training-set performance — 5-fold cross-validation on the training
  set already covers this; the independent test is a separate held-out evaluation.
- Your sequences have pairwise similarity ≥80% — the article used CD-HIT to make sure
  sequence similarity of the dataset was less than 80%, so unfiltered test sets could
  inflate independent-test metrics through train/test leakage.
- The task requires cross-species or cross-tissue generalization (training on one
  tissue, testing on another) — the article handles that under a separate cross-species
  and cross-tissue validation step, not the standard independent dataset test.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: im6A-TS-CNN
  role: CNN classifier whose frozen trained model is applied to the independent testing
    sets to produce m6A predictions scored as Sn, Sp, Acc, MCC, AUC
  repo: https://github.com/liukeweiaway/DeepM6A_cnn
- name: Keras in TensorFlow 2.0
  role: Framework used under Python 3.6 to implement the model and run the independent
    dataset test predictions
- name: CD-HIT
  role: Filtering step applied before evaluation to ensure sequence similarity of
    the dataset is less than 80%, preventing homolog leakage into the test set
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.omtn.2020.07.034
    title: 'im6A-TS-CNN: Identifying the N6-Methyladenine Site in Multiple Tissues
      by Using the Convolutional Neural Network'
  source_content_hash: sha256:1981526e140567a8dcdd35c72ee20c11d5b31735c3e1cf0f4cf0a3c9006e4dd6
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
  - im6A-TS-CNN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# independent-dataset-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Held-out (independent) dataset evaluation of a trained tissue-specific m6A classifier, recording Sn, Sp, Acc, MCC, and AUC per species-tissue dataset (e.g., h_b, m_k, r_k) alongside 5-fold cross-validation results to demonstrate model stability and generalization.

## When to use

Use when a trained model (e.g., im6A-TS-CNN on 41-nt RNA sequences) must be evaluated on a separate benchmark test set, and per-dataset Sn/Sp/Acc/MCC/AUC values must be compared against cross-validation results.

## When NOT to use

- You only need within-training-set performance — 5-fold cross-validation on the training set already covers this; the independent test is a separate held-out evaluation.
- Your sequences have pairwise similarity ≥80% — the article used CD-HIT to make sure sequence similarity of the dataset was less than 80%, so unfiltered test sets could inflate independent-test metrics through train/test leakage.
- The task requires cross-species or cross-tissue generalization (training on one tissue, testing on another) — the article handles that under a separate cross-species and cross-tissue validation step, not the standard independent dataset test.

## Inputs

- 11 tissue-specific independent testing sets of balanced positive/negative 41-nt RNA sequences (h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, r_l)
- Trained im6A-TS-CNN model checkpoint (Keras/TensorFlow 2.0, Python 3.6)

## Outputs

- Per-dataset Sn, Sp, Acc, MCC, and AUC values for the independent dataset test across all 11 species-tissue combinations
- performance_metrics.csv compiling 5-fold cross-validation and independent-test metrics side by side
- ROC curves comparing 5-fold cross-validation and independent-test performance

## How to apply

After training the CNN (one-hot encoded 41-nt RNA segments: A=(1,0,0,0), U=(0,1,0,0), C=(0,0,1,0), G=(0,0,0,1); 200 convolutional filters of kernel size 6, max-pooling size 4, dropout 0.16, fully connected layers of 164 then 32 units, SGD with learning rate 0.001), apply the frozen model — without re-fitting — to each of the 11 independent testing sets corresponding to the species-tissue combinations (h_b, h_k, h_l, m_b, m_h, m_k, m_l, m_t, r_b, r_k, r_l). Compute the five metrics (sensitivity, specificity, accuracy, MCC, AUC) exactly as done for the 5-fold cross-validation test so values are directly comparable (e.g., h_b: Sn 75.17%, Sp 70.20%, Acc 72.69%, MCC 0.4543, AUC 0.8056). The rationale is that agreement between cross-validation and independent-test results demonstrates the model 'is stable for identifying the m6A sites'; compile all metrics for the 11 combinations into a single table (performance_metrics.csv) and plot ROC curves for both tests. Validation gate: most AUCs exceed 0.8 in both tests, and independent-test values are consistent with cross-validation values (e.g., r_k 5-fold AUC 0.9017 vs independent 0.9077).

## Related tools

- **im6A-TS-CNN** (CNN classifier whose frozen trained model is applied to the independent testing sets to produce m6A predictions scored as Sn, Sp, Acc, MCC, AUC) — https://github.com/liukeweiaway/DeepM6A_cnn
- **Keras in TensorFlow 2.0** (Framework used under Python 3.6 to implement the model and run the independent dataset test predictions)
- **CD-HIT** (Filtering step applied before evaluation to ensure sequence similarity of the dataset is less than 80%, preventing homolog leakage into the test set)

## Examples

```
python im6A-TS-cnn.py -i sequence.txt -o ccc.html -s Human -t brain -ts normal
```

## Evaluation signals

- Independent-test AUC for r_k should be ≈0.9077 (highest reported); r_k 5-fold AUC 0.9017 — a large deviation suggests test-set leakage or a training bug.
- h_b independent test should reproduce Sn ≈75.17%, Sp ≈70.20%, Acc ≈72.69%, MCC ≈0.4543, AUC ≈0.8056.
- Most of the 11 per-dataset AUCs must exceed 0.8 in the independent test, mirroring the cross-validation condition.
- Per-dataset independent-test metrics should be numerically close to the corresponding 5-fold cross-validation metrics, indicating the model is stable for identifying m6A sites.
- Output table must contain all 11 species-tissue combinations × 5 metrics for both test conditions, with balanced positive/negative sequence counts matching the benchmark dataset tables.

## Limitations

- Performance is reported per species-tissue combination; the method does not train a single universal m6A predictor across all 11 combinations.
- iRNA-m6A comparison shows im6A-TS-CNN does not win everywhere: it had slightly lower cross-validation AUC for mouse brain (0.8705 vs 0.8731) and lower independent-test AUC for rat brain (0.8516 vs 0.8968).
- Threshold-specific performance (low/normal/high specificity options) is deferred to supplementary Table S1, so the default independent-test metrics do not capture all operating points.
- The command-line release is version V0.1.2 (updated 2020-05-03) and the README lists pinned dependencies (pandas 1.0.3, tensorflow-cpu 2.1.0, numpy 1.18.3); results may differ on other versions.

## Evidence

- [results] it was found that most of the areas under the ROC curve (AUCs) are higher than 0.8 in both the 5-fold cross-validation test and independent test: [evidence span withheld — anchor status: not_found]
