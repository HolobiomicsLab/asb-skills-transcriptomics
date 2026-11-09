---
name: class-imbalance-smote-oversampling
description: Use when building an lncRNA/mRNA classifier from a training set where
  the two transcript classes are imbalanced (e.g. H-Train or a self-trained model
  via lncDC-train.py), prior to running RFECV feature selection or XGBoost training.
when_to_use_negative:
- Never apply SMOTE to the test set (H-Test, M-Test) or to prediction inputs — over-sampling
  evaluation data invalidates reported metrics
- 'Do not substitute random under-sampling: the article explicitly chose SMOTE to
  reduce loss of potentially valuable data'
- Skip if the input is already class-balanced or if you are only re-running prediction
  with a prebuilt model (lncDC.py) rather than training
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LncDC
  role: Training pipeline (lncDC-train.py) that applies SMOTE class balancing internally
    before RFECV feature selection and XGBoost model fitting
  repo: https://github.com/lim74/LncDC
- name: imbalanced-learn
  role: Provides the SMOTE over-sampling implementation (version 0.9.1) used to balance
    the lncRNA/mRNA training matrix
- name: XGBoost
  role: Classifier trained on the SMOTE-balanced matrix; downstream consumer of the
    balanced data (xgboost 1.7.1)
- name: scikit-learn
  role: RFECV feature selection and model/pipeline machinery applied to the SMOTE-balanced
    training data (scikit-learn 1.1.3)
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-22082-7
    title: 'LncDC: a machine learning-based tool for long non-coding RNA detection
      from RNA-Seq data'
  source_content_hash: sha256:3b4ffb0f52c226210da4c9a1b68c95bc95acfdd283c037821078edf434d5be2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lim74/LncDC
  tools:
  - COME
  - CPAT
  - CPC2
  - CPPred
  - LncDC
  - LncFinder
  - PLEK
  - Python
  - ViennaRNA
  - XGBoost
  - imbalanced-learn
  - numpy
  - pandas
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# Class imbalance handling with SMOTE over-sampling

## Summary

Balance lncRNA/mRNA training matrices with SMOTE (imbalanced-learn 0.9.1) before feature selection and model training in LncDC, so recursive feature elimination is not biased toward the majority class and no minority-class transcripts are discarded.

## When to use

Use when building an lncRNA/mRNA classifier from a training set where the two transcript classes are imbalanced (e.g. H-Train or a self-trained model via lncDC-train.py), prior to running RFECV feature selection or XGBoost training.

## When NOT to use

- Never apply SMOTE to the test set (H-Test, M-Test) or to prediction inputs — over-sampling evaluation data invalidates reported metrics
- Do not substitute random under-sampling: the article explicitly chose SMOTE to reduce loss of potentially valuable data
- Skip if the input is already class-balanced or if you are only re-running prediction with a prebuilt model (lncDC.py) rather than training

## Inputs

- mRNA transcripts FASTA (with CDS FASTA in matching order) and lncRNA transcripts FASTA, e.g. mrna_mouse.fasta, cds_mouse.fasta, lncrna_mouse.fasta
- Extracted 57-feature H-Train matrix (SIFs, SSFs, PFs) with mRNA/lncRNA class labels

## Outputs

- Balanced training feature matrix used internally by lncDC-train.py
- Fitted imputer and scaler pickles (e.g. self_mouse_imputer_SIF_PF.pkl, self_mouse_scaler_SIF_PF.pkl) paired with the XGBoost model pickle

## How to apply

Apply SMOTE inside the LncDC training pipeline (lncDC-train.py) rather than as a separate manual step: the training inputs are mRNA FASTA with matching CDS FASTA (same order and number) plus lncRNA FASTA. The rationale is two-fold: SMOTE over-sampling keeps all minority-class transcripts so no potentially valuable data is lost (versus random under-sampling used by earlier tools), and it must run BEFORE sklearn RFECV with XGBoost so the one-feature-per-iteration elimination on the 57-feature H-Train matrix (SIFs, SSFs, PFs) is not biased toward the majority class. Use imbalanced-learn 0.9.1 in a python 3.9 environment (pandas 1.5, numpy 1.23, scikit-learn 1.1.3, xgboost 1.7.1). Fit imputer and scaler on the balanced training data and ship them alongside the model (imputer_SIF_PF.pkl, scaler_SIF_PF.pkl) so prediction-time transforms match. Do not over-sample evaluation data: metrics on H-Test/H-Test-equivalents must be computed on the original class distribution.

## Related tools

- **LncDC** (Training pipeline (lncDC-train.py) that applies SMOTE class balancing internally before RFECV feature selection and XGBoost model fitting) — https://github.com/lim74/LncDC
- **imbalanced-learn** (Provides the SMOTE over-sampling implementation (version 0.9.1) used to balance the lncRNA/mRNA training matrix)
- **XGBoost** (Classifier trained on the SMOTE-balanced matrix; downstream consumer of the balanced data (xgboost 1.7.1))
- **scikit-learn** (RFECV feature selection and model/pipeline machinery applied to the SMOTE-balanced training data (scikit-learn 1.1.3))

## Examples

```
python bin/lncDC-train.py -m mrna_mouse.fasta -c cds_mouse.fasta -l lncrna_mouse.fasta -o self_mouse -t 8
```

## Evaluation signals

- Class balance after over-sampling: mRNA and lncRNA counts in the training matrix are equal, and no minority-class transcripts were discarded
- RFECV on the balanced 57-feature H-Train matrix reproduces the reported accuracy curve with the top-28-feature subset at tenfold CV accuracy ≈0.9823 (top-53 subset ≈0.9824)
- The fitted imputer and scaler pickles (imputer_SIF_PF.pkl, scaler_SIF_PF.pkl) are emitted alongside the XGBoost model and hexamer table after lncDC-train.py completes
- Environment check via test_requirements.py passes with imbalanced-learn == 0.9.1 installed
- Prediction with the self-trained model on the unbalanced test set (e.g. H-Test) yields XGBoost accuracy ≈0.9800 and AUC ≈0.9974, confirming the balanced training data did not distort evaluation

## Limitations

- SMOTE balancing only applies at training time; prediction (lncDC.py) receives raw transcripts and outputs lncrna/mrna labels on the original distribution
- The article notes SIFs and PFs already achieved high accuracy, so class balancing plus feature work yielded only marginal further improvement
- For non-model organisms with few annotated lncRNAs, the README suggests training on data from evolutionarily close model organisms instead — balance cannot be achieved purely by over-sampling a tiny minority class

## Evidence

- [intro] SMOTE chosen to avoid data loss: "we used synthetic minority over-sampling technique (SMOTE) to reduce loss of potentially valuable data"
- [full_text] SMOTE applied so RFECV is not biased toward the majority class: [evidence span withheld — anchor status: not_found]
- [readme] imbalanced-learn version pin in the LncDC environment: "pandas == 1.5; numpy == 1.23; scikit-learn == 1.1.3; xgboost == 1.7.1; imbalanced-learn == 0.9.1"
- [methods] Training emits model, imputer and scaler artifacts from balanced data: "Run lncDC-train.py to obtain the hexamer table of your own training data"
