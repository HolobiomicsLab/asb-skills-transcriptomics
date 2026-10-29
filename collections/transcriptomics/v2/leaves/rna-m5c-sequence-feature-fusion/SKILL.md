---
name: rna-m5c-sequence-feature-fusion
description: Use when building sequence-based RNA modification (m5C) predictors that
  need richer features than a single encoding, and fused-feature dimensionality or
  training cost must be reduced with a filter method such as F score, MRMD, or PCC.
when_to_use_negative:
- Input is already a feature table from a single encoding — feature fusion would add
  redundant dimensions without an accuracy benefit
- Cross-species prediction is required — the article notes it could be difficult to
  develop a general model for cross-species prediction and species-specific models
  are necessary
- The target modification is not m5C or the sites are not centered single-nucleotide
  candidates
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3474
tools:
- name: Staem5
  role: Implements the fused-feature stacked ensemble prediction pipeline (m5C_predict.py)
    for Mouse and Arabidopsis
  repo: https://github.com/Cxd-626/Staem5.git
- name: iLearn
  role: Computes the PCPseDNC encoding from 38 physicochemical properties as part
    of feature fusion
- name: scikit-learn
  role: Provides SVM base classifier and stacking/logistic-regression framework
- name: mlxtend
  role: StackingCVClassifier combining SVM, XGBoost, and LightGBM base models with
    logistic regression meta-model
- name: XGBoost
  role: Gradient-boosted base classifier in the stacked ensemble
- name: LightGBM
  role: Light gradient boosting machine base classifier in the stacked ensemble
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.omtn.2021.10.012
    title: 'Staem5: A novel computational approach for accurate prediction of m5C
      site'
  source_content_hash: sha256:8de754c193199b7dbc965009fc78d2270f08f44afe101c7397f7c9e475dd3083
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Cxd-626/Staem5.git
  tools:
  - LightGBM
  - PCC
  - Staem5
  - XGBoost
  - iLearn
  - mlxtend
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# rna-m5c-sequence-feature-fusion

## Summary

Fuses four RNA sequence encoding schemes — position-specific propensity (PSP), k-mer (k = 1, 2, 3), parallel correlation pseudo dinucleotide composition (PCPseDNC, 38 physicochemical properties via iLearn), and PseEIIP — into a single feature matrix for m5C site prediction, then uses F score feature selection (step of 5) to control the dimensionality and computing cost of the fused set.

## When to use

Use when building sequence-based RNA modification (m5C) predictors that need richer features than a single encoding, and fused-feature dimensionality or training cost must be reduced with a filter method such as F score, MRMD, or PCC.

## When NOT to use

- Input is already a feature table from a single encoding — feature fusion would add redundant dimensions without an accuracy benefit
- Cross-species prediction is required — the article notes it could be difficult to develop a general model for cross-species prediction and species-specific models are necessary
- The target modification is not m5C or the sites are not centered single-nucleotide candidates

## Inputs

- FASTA file of RNA candidate-site sequences (query or training positives/negatives)
- Species label ('Arabidopsis' or 'Mouse') selecting the pretrained model
- Labeled m5C training dataset (e.g., 4,563 positive / 4,563 negative M. musculus samples)

## Outputs

- Fused feature matrix combining PSP, k-mer (k=1,2,3), PCPseDNC, and PseEIIP encodings
- Reduced-dimension feature sets ranked by F score (best ~185 features for M. musculus)
- m5C site prediction labels per query sequence
- 10-fold CV metrics (Sn, Sp, Acc, MCC, AUC) per feature-selection method and dimension

## How to apply

Encode each candidate m5C site sequence with four encodings: PSP, k-mer with k = 1, 2, and 3, PCPseDNC computed from 38 physicochemical properties (via the iLearn package), and PseEIIP, then concatenate them into one fused feature matrix. Rank all fused features with the F score criterion and incrementally add top-ranked features in steps of 5 to reduce feature dimension and computing time; the Staem5 paper found best 10-fold CV accuracy of 77.42% on M. musculus at 185 features with F score (vs. 77.21% with MRMD at 230 and PCC at 280 dimensions). Train a stacking ensemble (SVM, XGBoost, and LightGBM base classifiers combined via a logistic regression meta-classifier with mlxtend) at each dimension, using the training data (e.g., 4,563 positive and 4,563 negative M. musculus samples). Select the optimal base-classifier combination and dimension by 10-fold cross-validation accuracy, and validate the final model on an independent test set.

## Related tools

- **Staem5** (Implements the fused-feature stacked ensemble prediction pipeline (m5C_predict.py) for Mouse and Arabidopsis) — https://github.com/Cxd-626/Staem5.git
- **iLearn** (Computes the PCPseDNC encoding from 38 physicochemical properties as part of feature fusion)
- **scikit-learn** (Provides SVM base classifier and stacking/logistic-regression framework)
- **mlxtend** (StackingCVClassifier combining SVM, XGBoost, and LightGBM base models with logistic regression meta-model)
- **XGBoost** (Gradient-boosted base classifier in the stacked ensemble)
- **LightGBM** (Light gradient boosting machine base classifier in the stacked ensemble)

## Examples

```
python m5C_predict.py --input query.fasta --output predictions.tsv --species Mouse
```

## Evaluation signals

- Fused-feature dimension is reduced to the best F score dimension (185 features for M. musculus) with 10-fold CV accuracy ≈ 77.42%, matching MRMD (230 dims, 77.21%) and PCC (280 dims, 77.21%) within tolerance
- 10-fold CV metrics Sn, Sp, Acc, MCC, and AUC are recorded for every selection method × dimension sweep
- Stacked model uses exactly SVM, XGBoost, and LightGBM base classifiers with logistic regression as meta-model; adding further classifiers (ExtraTree, GBDT) should not improve accuracy
- Feature-selection sweep adds top-ranked features in steps of 5 until accuracy stops improving
- Parameter optimization via Bayesian optimization improves base classifier performance, with SVM showing the largest gain

## Limitations

- No general cross-species model; species-specific ('Arabidopsis' or 'Mouse') models are required
- Selected hyperparameters are only available in supplementary Table S2 and feature-selection results in Tables S5–S7, not the main text
- Staem5 underperformed iRNA-m5C_SVM and m5CPred-SVM on the independent A. thaliana test set
- Specific Python package versions are required (sklearn 0.21.3, xgboost 0.90, lightgbm 2.3.0, mlxtend 0.17.3, python 3.7), which may conflict with modern environments

## Evidence
