---
name: sequence-feature-encoding-pseudonucleotide-composition
description: Use when building RNA modification (m5C) predictors that need fused informatic
  sequence profiles from raw FASTA-like training sequences before classifier training
  or F-score feature selection.
when_to_use_negative:
- Input is already a numeric feature table — encoding would be redundant
- The goal is cross-species prediction with a single general model — the paper found
  species-specific models are necessary ('it could be difficult to develop a general
  model for cross-species prediction')
- Predicting m5C in species other than A. thaliana or M. musculus, since only 'Arabidopsis'
  or 'Mouse' are accepted by the released predictor
edam_operation: http://edamontology.org/operation_0267
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3474
tools:
- name: Staem5
  role: Provides the prediction pipeline and repository containing the encoding schemes,
    F-score-selected feature subsets, and stacked model
  repo: https://github.com/Cxd-626/Staem5.git
- name: iLearn
  role: Toolkit used to compute the sequence encodings (PCPseDNC, PSP, k-mer, PseEIIP)
- name: m5C_predict.py (Staem5)
  role: CLI entry point that accepts FASTA input and species and emits m5C predictions
    using the encoded-feature stacked model
  repo: https://github.com/Cxd-626/Staem5.git
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.omtn.2021.10.012
    title: 'Staem5: A novel computational approach for accurate prediction of m5C
      site'
  source_content_hash: sha256:34fe4759f1e4e23dbaaaadefd6575f5a57a56ecfa3ff60e4af887482b9684a04
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Cxd-626/Staem5.git
  tools:
  - LightGBM
  - Python
  - Staem5
  - iLearn
  - m5C_predict.py (Staem5)
  - mlxtend
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0267
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# sequence-feature-encoding-pseudonucleotide-composition

## Summary

Encode RNA sequences into numeric feature vectors using four complementary schemes — position-specific propensity (PSP), k-mer (k=1,2,3), parallel correlation pseudo dinucleotide composition (PCPseDNC), and electron-ion interaction pseudo potentials of trinucleotide (PseEIIP) — as the fused input representation for the Staem5 m5C site prediction model.

## When to use

Use when building RNA modification (m5C) predictors that need fused informatic sequence profiles from raw FASTA-like training sequences before classifier training or F-score feature selection.

## When NOT to use

- Input is already a numeric feature table — encoding would be redundant
- The goal is cross-species prediction with a single general model — the paper found species-specific models are necessary ('it could be difficult to develop a general model for cross-species prediction')
- Predicting m5C in species other than A. thaliana or M. musculus, since only 'Arabidopsis' or 'Mouse' are accepted by the released predictor

## Inputs

- Positive and negative RNA sequence training sets (e.g., M. musculus m5C: 4,563 positive / 4,563 negative sequences)
- FASTA-formatted query RNA sequences for prediction
- Species indicator ('Arabidopsis' or 'Mouse') to select the appropriate model

## Outputs

- Fused numeric feature matrix combining PSP, k-mer (k=1,2,3), PCPseDNC, and PseEIIP encodings
- F-score-selected reduced feature subset (e.g., 185 features for M. musculus) used by the stacked model
- Per-sequence m5C prediction labels/probabilities written to an output file

## How to apply

Take the positive and negative training sequences (e.g., the M. musculus m5C set of 4,563 positive and 4,563 negative sequences, or A. thaliana sets) and compute four encoding schemes: PSP, k-mer with k=1, 2, and 3, PCPseDNC, and PseEIIP; fuse the resulting vectors into a single feature matrix. This fused representation is intentionally high-dimensional and redundant, so follow it with F score-based feature selection (reducing dimension with a small step of 5) to cut feature dimension and computing time before training. The selected features (e.g., an F-score-selected 185-feature subset for M. musculus) then feed the stacked ensemble of SVM, XGBoost, and LightGBM with a logistic regression meta-classifier. Encoding can be performed with the iLearn package, and the Staem5 repository provides the pipeline and precomputed F-score-selected feature subsets.

## Related tools

- **Staem5** (Provides the prediction pipeline and repository containing the encoding schemes, F-score-selected feature subsets, and stacked model) — https://github.com/Cxd-626/Staem5.git
- **iLearn** (Toolkit used to compute the sequence encodings (PCPseDNC, PSP, k-mer, PseEIIP))
- **m5C_predict.py (Staem5)** (CLI entry point that accepts FASTA input and species and emits m5C predictions using the encoded-feature stacked model) — https://github.com/Cxd-626/Staem5.git

## Examples

```
python m5C_predict.py --input query.fa --output predictions.txt --species Mouse
```

## Evaluation signals

- 10-fold cross-validation on the M. musculus training set should reproduce approximately accuracy 77.42%, MCC 0.549, AUC 0.855 for the stacked model on encoded features
- A. thaliana stacked model should reach approximately 73.89% accuracy and 0.479 MCC
- The reduced feature matrix dimension should match the F-score-selected subset size (e.g., 185 features for M. musculus)
- Encoding must cover all four schemes — PSP, k-mer (k=1,2,3), PCPseDNC, and PseEIIP — since the model 'integrates four kinds of encoding schemes'
- Running m5C_predict.py --help should succeed in an environment with the pinned versions (sklearn 0.21.3, xgboost 0.90, lightgbm 2.3.0, mlxtend 0.17.3)

## Limitations

- Species-specific models only: no general cross-species model is feasible, and the released predictor accepts only 'Arabidopsis' or 'Mouse'
- Staem5 was inferior to iRNA-m5C_SVM and m5CPred-SVM on the independent test set of A. thaliana, so encoding choices do not guarantee superiority on all independent datasets
- Selected hyperparameters and detailed feature-selection results are only in supplementary Tables S2 and S5–S7, not the main text
- The README pins exact package versions (e.g., sklearn 0.21.3, tensorflow 1.14, python 3.7); other versions may not run

## Evidence
