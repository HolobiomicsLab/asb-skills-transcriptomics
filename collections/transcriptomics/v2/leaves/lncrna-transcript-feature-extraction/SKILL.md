---
name: lncrna-transcript-feature-extraction
description: Use when you have raw transcript FASTA files (mRNA sequences, their matching
  CDS sequences, and lncRNA sequences) and need to build a labeled feature matrix
  for training or reproducing an lncRNA/mRNA classification model such as LncDC's.
when_to_use_negative:
- You only need to classify unknown transcripts with an existing model — use lncDC.py
  prediction instead of re-extracting a training feature matrix
- You lack matched mRNA/CDS/lncRNA training FASTAs for the organism and cannot obtain
  well-annotated lncRNAs from an evolutionarily close model organism
- You need alignment- or homology-based lncRNA identification (e.g. CPC/CPAT-style
  coding-potential searches against protein databases) rather than alignment-free
  feature-based classification
tools:
- name: LncDC
  role: Provides lncDC-train.py, the feature-extraction and training pipeline that
    reads the mRNA/CDS/lncRNA FASTAs and emits the hexamer table, model, imputer,
    and scaler
  repo: https://github.com/lim74/LncDC
- name: ViennaRNA
  role: RNAfold computes RNA secondary structures needed to derive SASS k-mer (SSF)
    tables when the -r flag is enabled
  repo: https://www.tbi.univie.ac.at/RNA/
- name: XGBoost
  role: Trains the classifier on the extracted feature matrix (paired model pickles
    XGB_model_SIF_PF.pkl); version pinned to 1.7.1 in the README
  repo: https://github.com/lim74/LncDC
- name: Python
  role: Runtime environment (Python >= 3.9 conda env) in which lncDC-train.py performs
    feature extraction and model serialization
  repo: https://github.com/lim74/LncDC
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-22082-7
    title: 'LncDC: a machine learning-based tool for long non-coding RNA detection
      from RNA-Seq data'
  source_content_hash: sha256:bd41c2f169ac068dcebfa95ebea6ed8ab0d864787f02c921247144abed7f0f1f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lim74/LncDC
  tools:
  - LncDC
  - Python
  - ViennaRNA
  - XGBoost
  - numpy
  - pandas
  - scikit-learn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# lncrna-transcript-feature-extraction

## Summary

Extracts an alignment-free machine learning feature matrix from transcript FASTA sequences (mRNA, CDS, lncRNA) using the LncDC pipeline, combining sequence intrinsic features (SIFs), flexible ORF features, protein features (PFs), and optional sequence-and-secondary-structure (SASS) k-mer features. It is the preprocessing step that produces the training matrices and support files (hexamer table, imputer, scaler, SS k-mer tables) consumed by LncDC's XGBoost model.

## When to use

Use when you have raw transcript FASTA files (mRNA sequences, their matching CDS sequences, and lncRNA sequences) and need to build a labeled feature matrix for training or reproducing an lncRNA/mRNA classification model such as LncDC's.

## When NOT to use

- You only need to classify unknown transcripts with an existing model — use lncDC.py prediction instead of re-extracting a training feature matrix
- You lack matched mRNA/CDS/lncRNA training FASTAs for the organism and cannot obtain well-annotated lncRNAs from an evolutionarily close model organism
- You need alignment- or homology-based lncRNA identification (e.g. CPC/CPAT-style coding-potential searches against protein databases) rather than alignment-free feature-based classification

## Inputs

- mRNA transcript sequences in FASTA format (regular text or gzip-compressed *.gz)
- CDS sequences of the mRNAs in FASTA format, in the same order and number as the mRNA sequences
- lncRNA transcript sequences in FASTA format
- ViennaRNA installation (optional, required only when -r enables secondary structure feature extraction)

## Outputs

- hexamer table CSV (e.g. self_mouse_hexamer_table.csv)
- trained XGBoost model pickle (e.g. self_mouse_xgb_model_SIF_PF.pkl)
- imputer pickle (e.g. self_mouse_imputer_SIF_PF.pkl)
- scaler pickle (e.g. self_mouse_scaler_SIF_PF.pkl)
- with -r: five secondary structure k-mer tables (ss_table_k1.csv ... ss_table_k5.csv)
- feature matrix over sequence intrinsic features, ORF features, protein features, and (optionally) SSFs for SMOTE-balanced training

## How to apply

Run lncDC-train.py with -m (mRNA FASTA), -c (CDS FASTA, where order and number of CDS sequences must match the mRNA sequences), and -l (lncRNA FASTA), plus an output prefix and thread count; this yields four files: a hexamer table, an XGBoost model .pkl, an imputer .pkl, and a scaler .pkl. Add -r to also extract secondary structure features, which invokes RNAfold via ViennaRNA (much slower) and additionally generates five SASS k-mer tables (ss_table_k1 through k5). Features fall into three families defined by the paper: sequence intrinsic features (SIFs), secondary structure features (SSFs) that integrate primary sequence and RNAfold structure, and protein features (PFs); four ORF types (following FEELnc) are defined to fully capture CDS characteristics. For human H-Train, the extracted 57-feature matrix should be balanced with SMOTE over-sampling rather than random under-sampling so no minority-class transcripts are discarded. Any gaps in feature values are handled by the imputer, and features are standardized by the scaler paired with the trained model.

## Related tools

- **LncDC** (Provides lncDC-train.py, the feature-extraction and training pipeline that reads the mRNA/CDS/lncRNA FASTAs and emits the hexamer table, model, imputer, and scaler) — https://github.com/lim74/LncDC
- **ViennaRNA** (RNAfold computes RNA secondary structures needed to derive SASS k-mer (SSF) tables when the -r flag is enabled) — https://www.tbi.univie.ac.at/RNA/
- **XGBoost** (Trains the classifier on the extracted feature matrix (paired model pickles XGB_model_SIF_PF.pkl); version pinned to 1.7.1 in the README) — https://github.com/lim74/LncDC
- **Python** (Runtime environment (Python >= 3.9 conda env) in which lncDC-train.py performs feature extraction and model serialization) — https://github.com/lim74/LncDC

## Examples

```
python bin/lncDC-train.py -m mrna_mouse.fasta -c cds_mouse.fasta -l lncrna_mouse.fasta -o self_mouse -t 8
```

## Evaluation signals

- The output prefix directory contains exactly the four expected files: hexamer table CSV, XGB model .pkl, imputer .pkl, and scaler .pkl (e.g. self_mouse_hexamer_table.csv, self_mouse_xgb_model_SIF_PF.pkl, ...)
- With -r enabled, five additional SS k-mer tables (ss_table_k1.csv through ss_table_k5.csv) are generated alongside the standard outputs
- The CDS FASTA line up with the mRNA FASTA: identical order and identical number of sequences, as required by the -c argument contract
- Extracted features separate the classes: the paper reports that SIFs and PFs 'displayed two distinguished clusters for mRNAs and lncRNAs in H-Train'
- The 57-feature H-Train matrix, after SMOTE balancing and tenfold-CV tuning, should support an XGBoost model reaching accuracy 0.9800 and AUC 0.9974 on H-Test when reproducing the model-selection comparison

## Limitations

- Secondary structure feature extraction is time-consuming because RNAfold calculation is slow; leave -r off when SSFs are not needed
- For non-model organisms lacking well-annotated lncRNAs, a model must be trained on data from an evolutionarily close organism instead
- SSFs gave only a marginal improvement: the 6 selected SSFs 'performed well in H-Test' but the enhancement 'seems not a giant step' because SIFs and PFs had already achieved high accuracy
- Dependency versions are pinned (pandas 1.5, numpy 1.23, scikit-learn 1.1.3, xgboost 1.7.1, imbalanced-learn 0.9.1); ViennaRNA is optional but required for SS-feature prediction

## Evidence

- [intro] Feature families integrated by LncDC: "we developed an alignment-free, machine learning-based tool named LncDC, which integrates sequence intrinsic features (SIFs), secondary structure features (SSFs), and protein features (PFs)"
- [methods] Training pipeline invocation: "python bin/lncDC-train.py -m mrna_mouse.fasta -c cds_mouse.fasta -l lncrna_mouse.fasta -o SS_mouse -r -t 8"
- [methods] Hexamer table generation from training data: "Run lncDC-train.py to obtain the hexamer table of your own training data"
- [readme] CDS input contract: "The order and number of the CDS sequences should be the same as the mRNA sequences."
- [readme] SASS k-mer table outputs: "five ss tables will also be generated, including SS_mouse_ss_table_k1.csv, SS_mouse_ss_table_k2.csv, SS_mouse_ss_table_k3.csv, SS_mouse_ss_table_k4.csv and SS_mouse_ss_table_k5.csv."
- [discussion] ORF features rationale: "different types of ORF and their associated features effectively improved the ability of LncDC in capturing CDS characteristics of mRNAs and discriminated them from lncRNAs"
