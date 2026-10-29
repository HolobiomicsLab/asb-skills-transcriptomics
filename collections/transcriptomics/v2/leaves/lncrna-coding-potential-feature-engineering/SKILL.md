---
name: lncrna-coding-potential-feature-engineering
description: Use when building a coding-potential classifier for RNA transcripts and
  you must design the feature space (SIFs, SSFs, PFs, ORF-type features) or trim it
  down with recursive feature elimination while preserving accuracy.
when_to_use_negative:
- Prediction-only runs where a prebuilt/self-trained model already exists (python
  lncDC.py with -m model.pkl) — feature engineering and RFECV are training-time activities,
  not needed per prediction
- Input is a single expression matrix or a feature table that is already numeric and
  complete — skip re-engineering of SIFs/SSFs/PFs from raw FASTA
- Transcripts cannot be aligned to training data or lack CDS annotations for the mRNA
  class (cds.fa must match mrna.fa in order and number for lncDC-train.py)
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: LncDC
  role: XGBoost-based lncRNA detection tool whose 57-feature training matrix (SIFs,
    SSFs, PFs) this skill engineers and reduces; lncDC-train.py generates the hexamer
    table, ss k-mer tables, model, imputer, and scaler
  repo: https://github.com/lim74/LncDC
- name: XGBoost
  role: Classifier used inside RFECV to score feature subsets with tenfold-cross-validated
    accuracy; chosen after outperforming LR, DT, SVM, and RF
- name: scikit-learn
  role: Provides sklearn.feature_selection.RFECV for recursive feature elimination
    (one feature per iteration) with accuracy scoring
- name: imbalanced-learn
  role: SMOTE over-sampling applied to the 57-feature H-Train matrix before RFECV
    so elimination is not biased toward the majority class
- name: ViennaRNA
  role: RNAfold computes the secondary structures from which SSF k-mer score features
    are derived; optional dependency, only needed when -r is set
  repo: https://www.tbi.univie.ac.at/RNA/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-022-22082-7
    title: 'LncDC: a machine learning-based tool for long non-coding RNA detection
      from RNA-Seq data'
  source_content_hash: sha256:4e4ac2f6ac06300b18608f2c4aa0bfa78c15263fec69144f492abadbef193d78
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lim74/LncDC
  tools:
  - LncDC
  - Python
  - ViennaRNA
  - XGBoost
  - imbalanced-learn
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# lncrna-coding-potential-feature-engineering

## Summary

Engineering and reducing the numeric feature matrix that an XGBoost-based lncRNA/mRNA classifier consumes: sequence intrinsic features (SIFs), secondary structure features (SSFs), and protein features (PFs) are extracted from RNA transcripts, then a compact subset is selected by RFECV on the SMOTE-balanced H-Train matrix.

## When to use

Use when building a coding-potential classifier for RNA transcripts and you must design the feature space (SIFs, SSFs, PFs, ORF-type features) or trim it down with recursive feature elimination while preserving accuracy.

## When NOT to use

- Prediction-only runs where a prebuilt/self-trained model already exists (python lncDC.py with -m model.pkl) — feature engineering and RFECV are training-time activities, not needed per prediction
- Input is a single expression matrix or a feature table that is already numeric and complete — skip re-engineering of SIFs/SSFs/PFs from raw FASTA
- Transcripts cannot be aligned to training data or lack CDS annotations for the mRNA class (cds.fa must match mrna.fa in order and number for lncDC-train.py)

## Inputs

- mRNA FASTA file (with matching CDS FASTA in identical order, e.g. mrna.fa / cds.fa) and lncRNA FASTA file
- 57-feature H-Train feature matrix (SIFs, SSFs, PFs) with mRNA/lncRNA class labels
- Hexamer table and secondary-structure k-mer tables (e.g. train_ss_table_k1.csv through _k5.csv) built by lncDC-train.py
- Transcript secondary structures (or RNAfold predictions via ViennaRNA) for SSF computation

## Outputs

- feature_selection_results.csv containing selected feature names, feature-group labels (SSF/SIF/PF), and the tenfold CV accuracy-versus-feature-count curve
- The selected 28-feature subset (6 SSFs + 14 SIFs + 8 PFs) with its tenfold cross-validation accuracy (0.9823)
- Serialized model artifacts from training (e.g. XGB_model_SIF_PF.pkl, imputer_SIF_PF.pkl, scaler_SIF_PF.pkl)

## How to apply

Extract SIFs (including SASS k-mer score features with hexamer tables), SSFs via RNAfold/ViennaRNA (generating ss_table k1–k5 k-mer tables that integrate primary sequence and structure), and PFs plus flexible ORF features from four ORF types to capture CDS characteristics. Assemble the 57-feature H-Train matrix (SIFs, SSFs, PFs) with mRNA/lncRNA labels in pandas (pandas 1.5, numpy 1.23), then apply SMOTE (imbalanced-learn 0.9.1) so recursive elimination is not biased toward the majority class. Run sklearn.feature_selection.RFECV (scikit-learn 1.1.3) with an XGBoost classifier (xgboost 1.7.1), tenfold cross-validation, and accuracy as the scoring metric, eliminating one feature per iteration to build the accuracy-versus-feature-count curve. When the peak (53 features, CV accuracy 0.9824) and a much smaller subset (28 features, 0.9823) differ by only ~0.0001, prefer the compact 28-feature subset: 6 SSFs + 14 SIFs + 8 PFs. Write selected feature names, group labels, and the CV curve to feature_selection_results.csv.

## Related tools

- **LncDC** (XGBoost-based lncRNA detection tool whose 57-feature training matrix (SIFs, SSFs, PFs) this skill engineers and reduces; lncDC-train.py generates the hexamer table, ss k-mer tables, model, imputer, and scaler) — https://github.com/lim74/LncDC
- **XGBoost** (Classifier used inside RFECV to score feature subsets with tenfold-cross-validated accuracy; chosen after outperforming LR, DT, SVM, and RF)
- **scikit-learn** (Provides sklearn.feature_selection.RFECV for recursive feature elimination (one feature per iteration) with accuracy scoring)
- **imbalanced-learn** (SMOTE over-sampling applied to the 57-feature H-Train matrix before RFECV so elimination is not biased toward the majority class)
- **ViennaRNA** (RNAfold computes the secondary structures from which SSF k-mer score features are derived; optional dependency, only needed when -r is set) — https://www.tbi.univie.ac.at/RNA/

## Examples

```
from sklearn.feature_selection import RFECV; from xgboost import XGBClassifier; sel = RFECV(XGBClassifier(), step=1, cv=10, scoring='accuracy').fit(X_train_57feat_smote, y_labels)
```

## Evaluation signals

- The selected subset must contain exactly 6 secondary structure features, 14 sequence intrinsic features, and 8 protein features (28 total)
- Tenfold CV accuracy of the 28-feature subset must reproduce the reported 0.9823 within ~0.001
- The RFECV accuracy-versus-feature-count curve should peak at 53 features with 0.9824, confirming the 28-feature choice is only 0.0001 lower
- SMOTE must be applied before RFECV so class balancing precedes recursive elimination
- Trained artifacts (hexamer table, model .pkl, imputer .pkl, scaler .pkl; plus ss tables k1–k5 when -r is used) must all be produced by lncDC-train.py

## Limitations

- Secondary-structure feature extraction is time-consuming because RNAfold structure calculation dominates runtime; ViennaRNA is an optional dependency required only when -r is specified
- The enhancement from the 6 selected SSFs is acknowledged as marginal, and further accuracy improvement was difficult because SIFs and PFs already achieved high accuracy
- Feature engineering is training-data-dependent: for non-model organisms without well-annotated lncRNAs, train on data from evolutionary close model organisms instead

## Evidence

- [intro] We used recursive feature elimination with cross-validation (RFECV) for feature selection ... we used synthetic minority over-sampling technique (SMOTE) to reduce loss of potentially valuable data: [evidence span withheld — anchor status: not_found]
- [intro] Integration of SIFs, SSFs, PFs into an alignment-free tool: "we developed an alignment-free, machine learning-based tool named LncDC, which integrates sequence intrinsic features (SIFs), secondary structure features (SSFs), and protein features (PFs)"
- [intro] Four ORF types for CDS capture: "In order to fully capture the characteristics of CDS, we defined four types of ORF similar to those described in FEELnc"
- [intro] Novel SSFs integrating sequence and structure: "We also designed novel secondary structure features that integrate both primary sequence and secondary structure information from RNA transcripts"
- [discussion] XGBoost chosen over four alternative algorithms: "Five popular algorithms, LR, DT, SVM, RF, and XGBoost were compared in this study. Because XGBoost outperformed other algorithms in performance evaluation, it was used in the LncDC program"
- [readme] Pinned library versions for the feature-engineering stack: "pandas == 1.5; numpy == 1.23; scikit-learn == 1.1.3; xgboost == 1.7.1; imbalanced-learn == 0.9.1"
