---
name: f-score-feature-selection
description: Use when a fused high-dimensional sequence-encoding feature matrix (e.g.,
  PCPseDNC + PSP + k-mer + PseEIIP) must be reduced before training classifiers and
  you need a fast filter-type ranking such as the F score used by Staem5.
when_to_use_negative:
- Input is still raw FASTA RNA sequences — encode them first with PCPseDNC/PSP/k-mer/PseEIIP,
  since the F score operates on numeric feature vectors, not sequence strings
- The goal is a general cross-species m5C predictor — the paper states it is necessary
  to set up species-specific models, so a subset selected on M. musculus should not
  be assumed to transfer to A. thaliana
- Feature dimensionality and computing time are not constraints and you want joint-subset
  (wrapper/embedded) selection — Staem5's scope is filter ranking, with MRMD and PCC
  evaluated as the alternatives (Tables S5–S7)
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: Staem5
  role: Stacked ensemble pipeline whose feature-selection stage applies the F score
    (step of 5) over the fused encoding matrix before training the SVM, XGBoost, and
    LightGBM base classifiers with a logistic regression meta-classifier
  repo: https://github.com/Cxd-626/Staem5.git
- name: iLearn
  role: Generates the fused PCPseDNC, PSP, k-mer (k=1,2,3), and PseEIIP encodings
    that form the matrix the F-score ranking operates on
- name: scikit-learn
  role: Implements the SVM (radial basis kernel) and supporting base classifiers whose
    10-fold CV accuracy guides the progressive subset-size decision
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.omtn.2021.10.012
    title: 'Staem5: A novel computational approach for accurate prediction of m5C
      site'
  source_content_hash: sha256:c97bb3b5df2edfcd4b734d760a9d9cf9484cd2cd6ee0ebb127f7587690bc69d2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Cxd-626/Staem5.git
  tools:
  - LightGBM
  - Python
  - Staem5
  - iLearn
  - mlxtend
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# f-score-feature-selection

## Summary

Applies the F score as a filter-type ranking to progressively trim the fused PCPseDNC, PSP, k-mer (k=1,2,3), and PseEIIP encoding matrix, retaining the subset (185 features for M. musculus) that maximizes stacked-model accuracy while cutting computing time.

## When to use

Use when a fused high-dimensional sequence-encoding feature matrix (e.g., PCPseDNC + PSP + k-mer + PseEIIP) must be reduced before training classifiers and you need a fast filter-type ranking such as the F score used by Staem5.

## When NOT to use

- Input is still raw FASTA RNA sequences — encode them first with PCPseDNC/PSP/k-mer/PseEIIP, since the F score operates on numeric feature vectors, not sequence strings
- The goal is a general cross-species m5C predictor — the paper states it is necessary to set up species-specific models, so a subset selected on M. musculus should not be assumed to transfer to A. thaliana
- Feature dimensionality and computing time are not constraints and you want joint-subset (wrapper/embedded) selection — Staem5's scope is filter ranking, with MRMD and PCC evaluated as the alternatives (Tables S5–S7)

## Inputs

- Balanced training sequences (e.g., M. musculus 4,563 positive / 4,563 negative m5C and non-m5C sites) encoded into the fused PCPseDNC, PSP, k-mer (k=1,2,3), and PseEIIP feature matrix
- Binary class labels (m5C site vs non-m5C site) aligned to each feature row

## Outputs

- F-score-ranked feature list with progressively reduced candidate subsets (removal step of 5)
- Optimal F-score-selected feature subset (185 features for the M. musculus stacked model)
- Reduced-dimension training matrix used to fit the SVM/XGBoost/LightGBM + logistic-regression stack and its cross-validation metrics (e.g., mouse_cv_metrics.csv)

## How to apply

Fuse the four encoding schemes — position-specific propensity (PSP), k-mer (k=1,2,3), PCPseDNC, and PseEIIP — computed on the balanced training set (e.g., 4,563 positive and 4,563 negative M. musculus m5C sequences) into one numeric matrix. Rank every feature by its F score, the criterion Staem5 adopted after evaluating MRMD, PCC, and F score as candidate selection algorithms. Reduce the dimension progressively by dropping lowest-ranked features with a smaller step of 5, retraining and scoring candidate subsets at each size. Keep the subset whose 10-fold cross-validation accuracy is maximal (185 F-score-selected features for the M. musculus stacked model) and feed that reduced matrix to the SVM (radial basis kernel), XGBoost, and LightGBM base classifiers with the logistic regression meta-classifier. The rationale is that the F score reduces the dimension of features and computing time without wrapper-style search; detailed selection results live in supplementary Tables S5–S7.

## Related tools

- **Staem5** (Stacked ensemble pipeline whose feature-selection stage applies the F score (step of 5) over the fused encoding matrix before training the SVM, XGBoost, and LightGBM base classifiers with a logistic regression meta-classifier) — https://github.com/Cxd-626/Staem5.git
- **iLearn** (Generates the fused PCPseDNC, PSP, k-mer (k=1,2,3), and PseEIIP encodings that form the matrix the F-score ranking operates on)
- **scikit-learn** (Implements the SVM (radial basis kernel) and supporting base classifiers whose 10-fold CV accuracy guides the progressive subset-size decision)

## Evaluation signals

- The retained subset size is a multiple of the removal step of 5 (M. musculus optimum = 185 features)
- 10-fold CV accuracy across progressively reduced subsets traces a curve whose maximum defines the kept subset; metrics include Sn, Sp, Acc, MCC, AUC
- Reproduced M. musculus stacked-model performance on the selected subset matches accuracy 77.42%, MCC 0.549, AUC 0.855
- The F-score choice is backed by a three-way comparison against MRMD and PCC feature selection, with details documented in supplementary Tables S5–S7
- Training/computing time on the reduced matrix decreases relative to the full fused encoding, consistent with the stated purpose of reducing computing time

## Limitations

- Detailed feature-selection results are only available in supplementary Tables S5–S7, not in the main text
- Feature subsets are species-specific: the paper notes a general cross-species model is difficult, so the 185-feature M. musculus subset does not transfer to A. thaliana without re-selection
- Some hyperparameters were pre-set according to previous experience rather than systematically derived, which can shift the apparent optimal subset size

## Evidence
