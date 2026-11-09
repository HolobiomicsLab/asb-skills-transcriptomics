---
name: filter-feature-ranking-mrmd-pcc-fscore
description: Use when a fused high-dimensional sequence-encoding feature matrix (e.g.,
  PSP + k-mer + PCPseDNC + PseEIIP for RNA modification data) must be ranked and reduced
  by filter selection, and when comparing F score vs MRMD vs PCC to pick the subset
  dimension.
when_to_use_negative:
- Input is already a small or low-dimensional feature table where filter ranking offers
  no dimension-reduction benefit
- The task requires a wrapper or embedded selection method rather than the three filter
  algorithms (MRMD, PCC, F score) evaluated here
- Prediction-time use of the pre-trained Staem5 model via m5C_predict.py — feature
  selection is part of training/model-building, not inference
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: Staem5
  role: Provides the fused feature matrix, training data, and stacked ensemble in
    which the filter-selected feature subsets are evaluated
  repo: https://github.com/Cxd-626/Staem5.git
- name: iLearn
  role: Generates the PCPseDNC encoding (38 physicochemical properties) that feeds
    the fused feature matrix being ranked
- name: scikit-learn
  role: Implements base classifiers and F score ranking of the fused features
- name: mlxtend
  role: Builds the stacking ensemble (logistic regression meta-classifier) used to
    score each feature dimension
- name: XGBoost
  role: Base classifier of the stacked model evaluated on each feature subset
- name: LightGBM
  role: Base classifier of the stacked model evaluated on each feature subset
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.omtn.2021.10.012
    title: 'Staem5: A novel computational approach for accurate prediction of m5C
      site'
  source_content_hash: sha256:4e3969e31e4b4e22b90c2dd1e7724be1cba0f88ec3ddc12ba2ed84132615274e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Cxd-626/Staem5.git
  tools:
  - LightGBM
  - MRMD
  - PCC
  - Staem5
  - XGBoost
  - iLearn
  - mlxtend
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# filter-feature-ranking-mrmd-pcc-fscore

## Summary

Compares three filter-type feature selection methods — F score, MRMD (maximum-relevance-maximum-distance), and PCC (Pearson correlation coefficient) — on a fused feature matrix (PSP, k-mer k=1–3, PCPseDNC, PseEIIP) for Staem5 m5C site prediction, ranking features and sweeping subset dimensions in steps of 5 to balance accuracy against dimensionality and computing time.

## When to use

Use when a fused high-dimensional sequence-encoding feature matrix (e.g., PSP + k-mer + PCPseDNC + PseEIIP for RNA modification data) must be ranked and reduced by filter selection, and when comparing F score vs MRMD vs PCC to pick the best subset dimension.

## When NOT to use

- Input is already a small or low-dimensional feature table where filter ranking offers no dimension-reduction benefit
- The task requires a wrapper or embedded selection method rather than the three filter algorithms (MRMD, PCC, F score) evaluated here
- Prediction-time use of the pre-trained Staem5 model via m5C_predict.py — feature selection is part of training/model-building, not inference

## Inputs

- Fused feature matrix from four encodings: PSP, k-mer (k=1,2,3), PCPseDNC (38 physicochemical properties, iLearn), PseEIIP
- Balanced m5C training labels (e.g., M. musculus 4,563 positive / 4,563 negative from the Staem5 repository)
- Stacked ensemble training pipeline (scikit-learn SVM, XGBoost, LightGBM; mlxtend StackingCVClassifier with logistic regression meta-classifier)

## Outputs

- Per-method ranked feature lists (F score, MRMD, PCC)
- Performance table of Sn, Sp, Acc, MCC, AUC per selection method and per feature dimension (step of 5)
- Optimal feature subset and dimension per method (paper: F score 185 features, Acc 77.42%; MRMD 230, PCC 280, Acc 77.21%)

## How to apply

Build the fused feature matrix from the four encodings (PSP, k-mer with k = 1, 2, 3, PCPseDNC with 38 physicochemical properties via iLearn, and PseEIIP) for the training set (e.g., M. musculus: 4,563 positive and 4,563 negative samples). Independently rank all fused features with each filter criterion: F score, MRMD, and PCC. For each method, incrementally add top-ranked features in steps of 5, training the stacked ensemble (SVM, XGBoost, LightGBM base classifiers with a logistic regression meta-classifier via mlxtend) at each dimension and recording Sn, Sp, Acc, MCC, and AUC under 10-fold cross-validation. Select the best dimension per method — in the paper F score reached 77.42% Acc at 185 features, while MRMD and PCC reached 77.21% at 230 and 280 dimensions — and use the F score curve to justify reduced feature dimension and computing time.

## Related tools

- **Staem5** (Provides the fused feature matrix, training data, and stacked ensemble in which the filter-selected feature subsets are evaluated) — https://github.com/Cxd-626/Staem5.git
- **iLearn** (Generates the PCPseDNC encoding (38 physicochemical properties) that feeds the fused feature matrix being ranked)
- **scikit-learn** (Implements base classifiers and F score ranking of the fused features)
- **mlxtend** (Builds the stacking ensemble (logistic regression meta-classifier) used to score each feature dimension)
- **XGBoost** (Base classifier of the stacked model evaluated on each feature subset)
- **LightGBM** (Base classifier of the stacked model evaluated on each feature subset)

## Examples

```
from sklearn.feature_selection import f_classif; F, _ = f_classif(X_fused, y); order = F.argsort()[::-1]; [train_stack(X_fused[:, order[:d]]) for d in range(5, X_fused.shape[1]+1, 5)]
```

## Evaluation signals

- Accuracy vs dimension curve peaks then plateaus: F score should reach ~77.42% Acc at 185 features, MRMD and PCC ~77.21% at 230 and 280 dimensions on the M. musculus 10-fold CV
- Per-dimension metrics table (Sn, Sp, Acc, MCC, AUC) is complete for all three methods and every step-of-5 dimension swept
- Optimal dimension from F score (185) is smaller than MRMD (230) and PCC (280), consistent with the paper's claim that F score reduces dimension and computing time
- Feature-selection evaluation results match the values reported in supplementary Tables S5–S7 within tolerance
- Each method's ranking is computed independently on the same fused matrix — identical rankings across methods indicate an implementation error

## Limitations

- Results are species-specific: the article notes it could be difficult to develop a general model for cross-species prediction, so per-species feature-selection reruns are needed
- Detailed feature-selection results are only available in supplementary Tables S5–S7, not the main text
- The comparison is restricted to three filter algorithms (MRMD, PCC, F score); no wrapper or embedded methods were benchmarked

## Evidence
