---
name: feature-dimension-sweep-cross-validation
description: Use when a fused feature matrix built from multiple sequence encodings
  (e.g., PSP, k-mer, PCPseDNC, PseEIIP) must be reduced by a filter criterion (F score,
  MRMD, PCC) and swept over candidate dimensions with cross-validated metrics to pick
  the smallest accurate feature subset.
when_to_use_negative:
- The input is already a reduced feature table or the encoding set has not been fused
  — the sweep's premise is trimming a high-dimensional fused matrix.
- The goal is final independent-test benchmarking of a fixed predictor (e.g., against
  iRNA-m5C_SVM or m5CPred-SVM) rather than internal dimension tuning.
- A single species model is being deployed for new sequence prediction via m5C_predict.py
  — that uses the pre-trained model and skips the dimension sweep entirely.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Staem5
  role: Stacked ensemble framework whose feature dimension is swept; provides the
    M. musculus training dataset and SVM/XGBoost/LightGBM base classifiers
  repo: https://github.com/Cxd-626/Staem5.git
- name: iLearn
  role: Computes the PCPseDNC encoding (38 physicochemical properties) that feeds
    the fused feature matrix before the sweep
- name: scikit-learn
  role: Implements base classifiers (e.g., SVM) and the 10-fold cross-validation loop
    used to score each dimension
- name: mlxtend
  role: StackingCVClassifier-style stacking of the three base classifiers with logistic
    regression as meta-classifier
- name: XGBoost
  role: One of the three selected base classifiers evaluated at every swept dimension
- name: LightGBM
  role: One of the three selected base classifiers evaluated at every swept dimension
- name: MRMD
  role: Alternative filter selection criterion (maximum-relevance-maximum-distance)
    used to benchmark the F score sweep
- name: PCC
  role: Alternative filter selection criterion (Pearson correlation coefficient) used
    to benchmark the F score sweep
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.omtn.2021.10.012
    title: 'Staem5: A novel computational approach for accurate prediction of m5C
      site'
  source_content_hash: sha256:ebdb00c35d638d20b9292d3680cd229ab0a8c9d32733121dec06375809d569da
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# feature-dimension-sweep-cross-validation

## Summary

A filter-based feature selection protocol that ranks a fused sequence-feature matrix with F score, MRMD, or PCC, then adds top-ranked features in increments of 5 while re-training and scoring a stacked ensemble by 10-fold cross-validation to find the smallest dimension that preserves accuracy. Staem5 used it to control the dimensionality and computing cost introduced by fusing four sequence encoding schemes.

## When to use

Use when a fused feature matrix built from multiple sequence encodings (e.g., PSP, k-mer, PCPseDNC, PseEIIP) must be reduced by a filter criterion (F score, MRMD, PCC) and swept over candidate dimensions with cross-validated metrics to pick the smallest accurate feature subset.

## When NOT to use

- The input is already a reduced feature table or the encoding set has not been fused — the sweep's premise is trimming a high-dimensional fused matrix.
- The goal is final independent-test benchmarking of a fixed predictor (e.g., against iRNA-m5C_SVM or m5CPred-SVM) rather than internal dimension tuning.
- A single species model is being deployed for new sequence prediction via m5C_predict.py — that uses the pre-trained model and skips the dimension sweep entirely.

## Inputs

- M. musculus m5C training FASTA dataset (4,563 positive and 4,563 negative samples) from the Staem5 repository (https://github.com/Cxd-626/Staem5.git)
- Fused feature matrix from four encodings: PSP, k-mer (k=1,2,3), PCPseDNC (38 physicochemical properties via iLearn), and PseEIIP
- Feature rankings from filter criteria: F score, MRMD, and PCC

## Outputs

- Per-dimension 10-fold cross-validation metrics table (Sn, Sp, Acc, MCC, AUC) for each selection method (F score, MRMD, PCC)
- Optimal feature subset dimension per selection method (e.g., 185 features for F score on M. musculus)
- Final trained stacked-ensemble model (SVM + XGBoost + LightGBM base classifiers with logistic regression meta-classifier) at the chosen dimension

## How to apply

Build the fused feature matrix from the four encodings used by Staem5: position-specific propensity (PSP), k-mer (k = 1, 2, 3), PCPseDNC (38 physicochemical properties computed via iLearn), and PseEIIP. Rank all fused features with a filter criterion — F score is Staem5's choice because it reduced both feature dimension and computing time — then incrementally add top-ranked features in steps of 5. Repeat the same sweeping procedure with independently computed MRMD (maximum-relevance-maximum-distance) and PCC (Pearson correlation coefficient) rankings for comparison. At each dimension, train the stacked ensemble (SVM, XGBoost, and LightGBM base classifiers combined by a logistic regression meta-classifier via the mlxtend stacking package) and evaluate with 10-fold cross-validation, recording Sn, Sp, Acc, MCC, and AUC. Select the method/dimension with the best accuracy; on the M. musculus m5C training set (4,563 positive / 4,563 negative) this yields 77.42% accuracy at 185 F score-selected features versus 77.21% at 230 (MRMD) and 280 (PCC) dimensions, favoring the F score for cheaper, smaller models.

## Related tools

- **Staem5** (Stacked ensemble framework whose feature dimension is swept; provides the M. musculus training dataset and SVM/XGBoost/LightGBM base classifiers) — https://github.com/Cxd-626/Staem5.git
- **iLearn** (Computes the PCPseDNC encoding (38 physicochemical properties) that feeds the fused feature matrix before the sweep)
- **scikit-learn** (Implements base classifiers (e.g., SVM) and the 10-fold cross-validation loop used to score each dimension)
- **mlxtend** (StackingCVClassifier-style stacking of the three base classifiers with logistic regression as meta-classifier)
- **XGBoost** (One of the three selected base classifiers evaluated at every swept dimension)
- **LightGBM** (One of the three selected base classifiers evaluated at every swept dimension)
- **MRMD** (Alternative filter selection criterion (maximum-relevance-maximum-distance) used to benchmark the F score sweep)
- **PCC** (Alternative filter selection criterion (Pearson correlation coefficient) used to benchmark the F score sweep)

## Examples

```
python m5C_predict.py --input query.fa --output results.tsv --species Mouse
```

## Evaluation signals

- Best 10-fold cross-validation accuracy on M. musculus reaches 77.42% with F score at 185 features and 77.21% with MRMD and PCC at 230 and 280 dimensions — reproduced values should match within tolerance.
- The dimension sweep must increase in fixed increments of 5 top-ranked features per step for every selection method.
- Sn, Sp, Acc, MCC, and AUC must all be recorded per dimension per method, enabling direct comparison across F score, MRMD, and PCC.
- F score-selected models should reach peak accuracy at a smaller dimension (185) than MRMD (230) and PCC (280), consistent with the claim that F score reduces dimension and computing time.
- The stacked model at each dimension must use SVM, XGBoost, and LightGBM as base classifiers with logistic regression as the meta-classifier, as in the published configuration.

## Limitations

- Detailed feature selection results are only available in supplementary Tables S5–S7, not in the main text, so exact per-dimension values require the SI.
- Hyperparameters were pre-set according to previous experience rather than systematically derived, which may affect sweep reproducibility.
- Models are species-specific (Mouse or Arabidopsis); no general cross-species model exists, so the sweep must be repeated per species.

## Evidence
