---
name: protein-physicochemical-feature-computation
description: Use when building an LncDC-style lncRNA-vs-mRNA feature matrix and protein-level
  descriptors (e.g. ProtParam properties from translated ORFs) must be added to sequence
  and secondary-structure features.
when_to_use_negative:
- The input is already a computed feature matrix (e.g. a populated 57-column csv)
  — PF computation belongs only to the raw-feature extraction stage
- Only secondary structure features (SSFs) are being added via the -r flag; PFs are
  computed independently of the ViennaRNA/RNAfold SSF path
- Input records lack a mappable mRNA/CDS pair, since the CDS-matched ORF definition
  drives the translated ORFs used for ProtParam features
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3474
tools:
- name: LncDC
  role: Tool whose feature-extraction module computes PFs from translated ORFs alongside
    SIFs and SSFs
  repo: https://github.com/lim74/LncDC
- name: Python
  role: Implementation language in which LncDC's ProtParam-based PF extraction is
    written and run
  repo: https://github.com/lim74/LncDC
- name: XGBoost
  role: Downstream classifier consuming the PF-augmented feature matrix for lncRNA/mRNA
    prediction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-22082-7
    title: 'LncDC: a machine learning-based tool for long non-coding RNA detection
      from RNA-Seq data'
  source_content_hash: sha256:8906a31bdf0a7b0baba59a83fbc7c322ff73dbcc137e510ab21388733e94e58c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lim74/LncDC
  tools:
  - LncDC
  - Python
  - ViennaRNA
  - XGBoost
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# protein-physicochemical-feature-computation

## Summary

Computes protein features (PFs) in LncDC by translating each transcript's ORFs and deriving ProtParam physicochemical properties, one of the three feature families (SIFs, SSFs, PFs) fed to the XGBoost lncRNA/mRNA classifier.

## When to use

Use when building an LncDC-style lncRNA-vs-mRNA feature matrix and protein-level descriptors (e.g. ProtParam properties from translated ORFs) must be added to sequence and secondary-structure features.

## When NOT to use

- The input is already a computed feature matrix (e.g. a populated 57-column csv) — PF computation belongs only to the raw-feature extraction stage
- Only secondary structure features (SSFs) are being added via the -r flag; PFs are computed independently of the ViennaRNA/RNAfold SSF path
- Input records lack a mappable mRNA/CDS pair, since the CDS-matched ORF definition drives the translated ORFs used for ProtParam features

## Inputs

- mRNA transcripts in FASTA format (plain text or gzip-compressed *.gz)
- Corresponding CDS sequences in FASTA format, in the same order and number as the mRNA records
- lncRNA sequences in FASTA format
- Translated ORF peptide sequences from the four FEELnc-style ORF types (type 0-3)

## Outputs

- Per-transcript protein feature (PF) columns within the 57-column feature matrix (e.g. h_train_feature_matrix.csv)
- Raw (pre-imputation, pre-scaling) PF values later consumed by imputer_SIF_PF.pkl and scaler_SIF_PF.pkl

## How to apply

Within LncDC's feature extraction, first define the four FEELnc-style ORF types (type 0-3) and translate the recovered ORFs from the matched mRNA/CDS FASTA pairs loaded with Biopython. Compute ProtParam protein features for each transcript's translated ORFs; the article motivates protein descriptors such as pH isoelectric point, which CPC2 also uses alongside ORF length and Fickett score. Keep the PF values as raw (unscaled) columns that contribute to the single-row-per-transcript feature matrix (57 columns total in h_train_feature_matrix.csv for H-Train); the imputer.pkl and scaler.pkl artifacts are applied only after raw feature assembly. PFs together with SIFs already reach high accuracy, so treat PF computation as part of the default (non-SSF) model path unless secondary structure features are explicitly requested with the -r flag.

## Related tools

- **LncDC** (Tool whose feature-extraction module computes PFs from translated ORFs alongside SIFs and SSFs) — https://github.com/lim74/LncDC
- **Python** (Implementation language in which LncDC's ProtParam-based PF extraction is written and run) — https://github.com/lim74/LncDC
- **XGBoost** (Downstream classifier consuming the PF-augmented feature matrix for lncRNA/mRNA prediction)

## Examples

```
python bin/lncDC-train.py -m mrna_mouse.fasta -c cds_mouse.fasta -l lncrna_mouse.fasta -o self_mouse -t 8
```

## Evaluation signals

- The assembled matrix has exactly one row per H-Train transcript and 57 populated feature columns, with PF columns populated (no empties before the imputation/scaling stage)
- PF values are retained as raw numbers — imputer.pkl and scaler.pkl are separate artifacts applied after feature assembly, not during extraction
- Each mRNA record has a matching CDS record in the same order, so translated ORFs (and hence PFs) exist for every type 0-3 ORF category
- Removing/altering PF inputs changes classifier behavior consistently with the article's finding that SIFs and PFs alone already achieve high accuracy

## Limitations

- SIFs and PFs already achieve high accuracy, so PF-based improvements over them were difficult to demonstrate in the study
- Protein features depend on CDS-matched ORF identification; the article notes 1909 RefSeq mRNA transcripts whose CDS differ from the longest ORF, which is why four ORF types rather than only the longest ORF are used
- PF computation assumes translated ORFs exist; transcripts without productive ORFs yield sparse values handled later by the imputer

## Evidence

- [intro] CPC2 is an upgraded version of CPC, which integrated a SVM model with Fickett score, ORF length, ORF integrity, and protein pH isoelectric point (PI) features to identify lncRNAs: "CPC2 is an upgraded version of CPC, which integrated a SVM model with Fickett score, ORF length, ORF integrity, and protein pH isoelectric point (PI) features to identify lncRNAs"
- [intro] we developed an alignment-free, machine learning-based tool named LncDC, which integrates sequence intrinsic features (SIFs), secondary structure features (SSFs), and protein features (PFs) with the: "we developed an alignment-free, machine learning-based tool named LncDC, which integrates sequence intrinsic features (SIFs), secondary structure features (SSFs), and protein features (PFs) with the"
- [discussion] different types of ORF and their associated features effectively improved the ability of LncDC in capturing CDS characteristics of mRNAs and discriminated them from lncRNAs: "different types of ORF and their associated features effectively improved the ability of LncDC in capturing CDS characteristics of mRNAs and discriminated them from lncRNAs"
- [readme] biopython >= 1.79: "python 3 >= 3.9; pandas == 1.5; numpy == 1.23; scikit-learn == 1.1.3; xgboost == 1.7.1; imbalanced-learn == 0.9.1; biopython >= 1.79"
- [readme] The order and number of the CDS sequences should be the same as the mRNA sequences.: "The order and number of the CDS sequences should be the same as the mRNA sequences."
