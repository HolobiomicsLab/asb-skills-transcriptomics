---
name: xgboost-classifier-training
description: Use when labeled mRNA+CDS and lncRNA FASTA training sets (human, mouse,
  or a model organism) are available and a binary coding-potential classifier must
  be trained and serialized for LncDC-style prediction.
when_to_use_negative:
- The goal is prediction on unlabeled transcripts with an already-trained model —
  use lncDC.py instead of training.
- The species has too few well-annotated lncRNAs for training and no transfer from
  a close model organism (zebrafish, Drosophila, C. elegans, Arabidopsis, etc.) is
  planned.
- ViennaRNA is not installed and -r is required, since RNAfold-based SSF training
  would fail or be prohibitively slow.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: LncDC
  role: Provides lncDC-train.py, which orchestrates feature extraction, SMOTE balancing,
    RFECV selection, and XGBoost model training
  repo: https://github.com/lim74/LncDC
- name: XGBoost
  role: Classifier algorithm trained on SIF/PF/SSF features; outperformed LR, DT,
    SVM, and RF in the paper's comparison
- name: scikit-learn
  role: Supplies RFECV recursive feature elimination and the imputer/scaler objects
    serialized alongside the model
- name: imbalanced-learn
  role: SMOTE oversampling so recursive elimination is not biased toward the majority
    mRNA class
- name: ViennaRNA
  role: RNAfold computes secondary structures used for SASS k-mer score features when
    training with -r
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-22082-7
    title: 'LncDC: a machine learning-based tool for long non-coding RNA detection
      from RNA-Seq data'
  source_content_hash: sha256:18f0e70133f13af7de76f1999f0bf793222aed3f7bcce925446538f88361f535
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
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# xgboost-classifier-training

## Summary

Train an XGBoost-based lncRNA/mRNA classification model with LncDC's lncDC-train.py, which internally applies SMOTE class balancing, RFECV feature selection over 57 sequence/structure/protein features, and serializes the hexamer table, model, imputer, and scaler needed for downstream prediction.

## When to use

Use when labeled mRNA+CDS and lncRNA FASTA training sets (human, mouse, or a model organism) are available and a binary coding-potential classifier must be trained and serialized for LncDC-style prediction.

## When NOT to use

- The goal is prediction on unlabeled transcripts with an already-trained model — use lncDC.py instead of training.
- The species has too few well-annotated lncRNAs for training and no transfer from a close model organism (zebrafish, Drosophila, C. elegans, Arabidopsis, etc.) is planned.
- ViennaRNA is not installed and -r is required, since RNAfold-based SSF training would fail or be prohibitively slow.

## Inputs

- mRNA transcript sequences in FASTA format (-m, plain or gzip-compressed)
- CDS sequences of the mRNAs in FASTA format (-c; same order and number as the mRNA file)
- lncRNA transcript sequences in FASTA format (-l)
- Optional -r flag enabling secondary structure feature training via RNAfold

## Outputs

- Hexamer table CSV (e.g. self_mouse_hexamer_table.csv)
- Serialized XGBoost model (e.g. self_mouse_xgb_model_SIF_PF.pkl)
- Imputer pickle (e.g. self_mouse_imputer_SIF_PF.pkl)
- Scaler pickle (e.g. self_mouse_scaler_SIF_PF.pkl)
- With -r: five secondary structure k-mer tables (prefix_ss_table_k1.csv through k5.csv)

## How to apply

Assemble three FASTA inputs: mRNA transcripts, their CDS sequences in the same order and number as the mRNAs (via -c), and lncRNA transcripts (via -l). Run lncDC-train.py with an output prefix (-o) and thread count (-t); training balances classes with SMOTE (imbalanced-learn 0.9.1) and performs RFECV with an XGBoost classifier (xgboost 1.7.1, scikit-learn 1.1.3) — on H-Train the 28-feature subset (6 SSFs + 14 SIFs + 8 PFs) reached tenfold CV accuracy 0.9823. Add the -r flag to include SASS k-mer secondary structure features, which invokes RNAfold (ViennaRNA) and generates five ss k-mer tables (k1–k5). XGBoost was chosen over LR, DT, SVM, and RF in the paper's five-algorithm comparison. Set up in a python>=3.9 conda env with pandas 1.5, numpy 1.23, and verify with test_requirements.py before training.

## Related tools

- **LncDC** (Provides lncDC-train.py, which orchestrates feature extraction, SMOTE balancing, RFECV selection, and XGBoost model training) — https://github.com/lim74/LncDC
- **XGBoost** (Classifier algorithm trained on SIF/PF/SSF features; outperformed LR, DT, SVM, and RF in the paper's comparison)
- **scikit-learn** (Supplies RFECV recursive feature elimination and the imputer/scaler objects serialized alongside the model)
- **imbalanced-learn** (SMOTE oversampling so recursive elimination is not biased toward the majority mRNA class)
- **ViennaRNA** (RNAfold computes secondary structures used for SASS k-mer score features when training with -r)

## Examples

```
python bin/lncDC-train.py -m mrna_mouse.fasta -c cds_mouse.fasta -l lncrna_mouse.fasta -o self_mouse -t 8
```

## Evaluation signals

- test_requirements.py passes in the lncdc conda environment (python>=3.9, scikit-learn 1.1.3, xgboost 1.7.1, imbalanced-learn 0.9.1).
- Four output files are produced with the -o prefix: hexamer table CSV, XGB model pkl, imputer pkl, and scaler pkl.
- With -r, exactly five ss k-mer tables (k1–k5) are generated alongside the other outputs.
- Selected-feature RFECV subset on H-Train reproduces tenfold CV accuracy ≈0.9823 (28 features: 6 SSFs + 14 SIFs + 8 PFs).
- Self-trained mouse model evaluated on M-Test reaches accuracy ≈0.9758 and MCC ≈0.9516 (human model ≈0.9723/0.9446).

## Limitations

- Training with -r is slow because secondary structure calculation by RNAfold (ViennaRNA) is time-consuming.
- For non-model organisms lacking well-annotated lncRNAs, training may be unreliable; README recommends training on data from evolutionarily close model organisms instead.
- Adding secondary structure features yields only marginal gains — the paper notes the enhancement 'seems not a giant step' because SIFs and PFs already achieve high accuracy.

## Evidence

- [discussion] Five popular algorithms, LR, DT, SVM, RF, and XGBoost were compared in this study. Because XGBoost outperformed other algorithms in performance evaluation, it was used in the LncDC program: "Five popular algorithms, LR, DT, SVM, RF, and XGBoost were compared in this study."
- [intro] RFECV and SMOTE rationale: "We used recursive feature elimination with cross-validation (RFECV) for feature selection"
- [readme] Training command: "python bin/lncDC-train.py -m mrna_mouse.fasta -c cds_mouse.fasta -l lncrna_mouse.fasta -o self_mouse -t 8"
- [readme] CDS ordering constraint: "The order and number of the CDS sequences should be the same as the mRNA sequences."
- [other] RFECV accuracy outcome: [evidence span withheld — anchor status: not_found]
