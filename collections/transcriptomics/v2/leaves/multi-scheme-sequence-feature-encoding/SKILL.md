---
name: multi-scheme-sequence-feature-encoding
description: Use when raw nucleotide sequence segments (e.g., 41-bp windows centered
  on candidate m5C sites in FASTA format) must be converted into informative numeric
  features to train or apply site-prediction classifiers.
when_to_use_negative:
- Input is already a numeric feature table or embedding — encoding raw sequences again
  is unnecessary
- 'The task requires cross-species generalization with a single model: the article
  notes it ''could be difficult to develop a general model for cross-species prediction,
  and it is necessary to set up species-specific models'''
- Sequence segments shorter than the 41-bp window used by Staem5, since PSP and PCPseDNC
  assume the fixed local window structure
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Staem5
  role: Reference implementation that fuses the four encoding schemes and applies
    F-score-based feature selection (step of 5) before classifier training
  repo: https://github.com/Cxd-626/Staem5.git
- name: iLearn
  role: Used to compute the PCPseDNC (parallel correlation pseudo dinucleotide composition)
    features
- name: scikit-learn
  role: Provides the base classifiers (SVM, XGBoost, LightGBM) that consume the encoded
    feature vectors
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.omtn.2021.10.012
    title: 'Staem5: A novel computational approach for accurate prediction of m5C
      site'
  source_content_hash: sha256:b21dc9129a6c8d91912fe9428d6f43de897ebd6bd91453cd75e3d913bc144d03
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Cxd-626/Staem5.git
  tools:
  - Staem5
  - iLearn
  - mlxtend
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# multi-scheme-sequence-feature-encoding

## Summary

Encode 41-bp RNA sequence segments by fusing four feature schemes — position-specific propensity (PSP), k-mer (k=1,2,3), PCPseDNC, and PseEIIP — into a single numeric feature vector, then reduce dimension with F-score-based selection (step of 5) for machine learning classifiers.

## When to use

Use when raw nucleotide sequence segments (e.g., 41-bp windows centered on candidate m5C sites in FASTA format) must be converted into informative numeric features to train or apply site-prediction classifiers.

## When NOT to use

- Input is already a numeric feature table or embedding — encoding raw sequences again is unnecessary
- The task requires cross-species generalization with a single model: the article notes it 'could be difficult to develop a general model for cross-species prediction, and it is necessary to set up species-specific models'
- Sequence segments shorter than the 41-bp window used by Staem5, since PSP and PCPseDNC assume the fixed local window structure

## Inputs

- 41-bp RNA sequence segments in FASTA format (e.g., the balanced A. thaliana training set of 5,298 positive and 5,298 negative samples)
- Sequence labels (positive m5C site vs. negative non-m5C site) for supervised feature selection

## Outputs

- Fused numeric feature matrix (one row per sequence segment; columns from k-mer, PSP, PCPseDNC, and PseEIIP schemes)
- F-score-selected reduced feature matrix (dimension reduced iteratively with step of 5)

## How to apply

Take each 41-bp sequence segment and compute four complementary encodings: k-mer composition for k=1, 2, and 3; position-specific propensity (PSP); parallel correlation pseudo dinucleotide composition (PCPseDNC), computed with the iLearn package; and electron-ion interaction pseudo potentials of trinucleotide (PseEIIP). Concatenate the four schemes into one fused feature vector per segment — the rationale is that no single encoding captures all signal, and Staem5's feature fusion leveraged these informatic sequence profiles. Because the fused vector is high-dimensional, apply F-score feature selection to reduce the dimension and computing time, using a small reduction step of 5 as described in the article. Feed the selected features to downstream classifiers (SVM, XGBoost, LightGBM, ExtraTree, GBDT combinations) within the stacking framework, evaluating with 10-fold cross-validation metrics (Sn, Sp, Acc, MCC, AUC).

## Related tools

- **Staem5** (Reference implementation that fuses the four encoding schemes and applies F-score-based feature selection (step of 5) before classifier training) — https://github.com/Cxd-626/Staem5.git
- **iLearn** (Used to compute the PCPseDNC (parallel correlation pseudo dinucleotide composition) features)
- **scikit-learn** (Provides the base classifiers (SVM, XGBoost, LightGBM) that consume the encoded feature vectors)

## Examples

```
python m5C_predict.py --input query_segments.fasta --output predictions.csv --species Arabidopsis
```

## Evaluation signals

- The fused encoding of the balanced A. thaliana set (5,298 positive / 5,298 negative 41-bp segments) followed by F-score selection (step of 5) yields 10-fold CV accuracy of approximately 73.89% with the SVM/XGBoost/LightGBM stack
- MCC of the downstream stacked model is approximately 0.479 and AUC approximately 0.810 on the same training set
- The reduced feature set retains discriminative power: per-classifier 10-fold CV accuracy should improve or remain stable relative to the full fused dimensionality
- Feature dimensions match the expected composition: k-mer counts for k=1,2,3 plus PSP, PCPseDNC, and PseEIIP blocks concatenated without overlap

## Limitations

- No general cross-species model is available; species-specific encoding-tuned models are required for A. thaliana vs. M. musculus
- Hyperparameters downstream of encoding were pre-set according to previous experience rather than systematically derived; optimized values appear only in supplementary Table S2
- Detailed comparison metrics for encoding/classifier combinations are available only in supplementary Tables S1, S3, and S5–S7, not the main text
- Staem5's encoding+stack pipeline underperformed iRNA-m5C_SVM and m5CPred-SVM on the independent A. thaliana test set

## Evidence
