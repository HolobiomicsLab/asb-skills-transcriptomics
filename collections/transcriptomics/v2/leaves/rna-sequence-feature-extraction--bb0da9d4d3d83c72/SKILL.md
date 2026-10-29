---
name: rna-sequence-feature-extraction
description: Use when raw RNA transcript sequences in FASTA format must be converted
  into the numeric feature set (SIFs, SS k-mer scores, ORF, protein features) that
  LncDC's XGBoost model consumes for lncRNA prediction, or when reproducing its SSF
  ablation experiments.
when_to_use_negative:
- Input is already a numeric feature table rather than raw transcript FASTA sequences
  — no sequence feature extraction is needed
- Secondary structure features are required but ViennaRNA is not installed (RNAfold
  is only available when the optional dependency is present)
- Only full-model default human prediction on well-annotated data is needed and runtime
  matters — skip -r since RNAfold is time-consuming and SSF gains are marginal
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3474
tools:
- name: LncDC
  role: Main tool implementing SIF/SSF/PF feature extraction and XGBoost prediction/training
    over RNA FASTA
  repo: https://github.com/lim74/LncDC
- name: XGBoost
  role: Classifier consuming the extracted features; model shipped as XGB_model_SIF_PF.pkl
    / XGB_model_SIF_PF_SSF.pkl
  repo: https://github.com/lim74/LncDC
- name: ViennaRNA
  role: Optional dependency; RNAfold computes RNA secondary structures used to build
    the SASS ss k-mer tables (required only with -r)
- name: scikit-learn
  role: Imputer and scaler pickles applied to extracted features before classification;
    metric computation for ablation
  repo: https://github.com/lim74/LncDC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-022-22082-7
    title: 'LncDC: a machine learning-based tool for long non-coding RNA detection
      from RNA-Seq data'
  source_content_hash: sha256:ca0742a23de8bf1b2f58633e9ad0613636dc585fe6ef5bab3a02946a868fb45e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lim74/LncDC
  tools:
  - LncDC
  - Python
  - ViennaRNA
  - XGBoost
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# rna-sequence-feature-extraction

## Summary

Extract sequence-derived features (sequence intrinsic features/SIFs, hexamer scores, flexible ORF features, and secondary structure features/SSFs via ViennaRNA k-mer tables) from RNA FASTA transcripts to feed LncDC's XGBoost lncRNA-vs-mRNA classifier.

## When to use

Use when raw RNA transcript sequences in FASTA format must be converted into the numeric feature set (SIFs, SS k-mer scores, ORF, protein features) that LncDC's XGBoost model consumes for lncRNA prediction, or when reproducing its SSF ablation experiments.

## When NOT to use

- Input is already a numeric feature table rather than raw transcript FASTA sequences — no sequence feature extraction is needed
- Secondary structure features are required but ViennaRNA is not installed (RNAfold is only available when the optional dependency is present)
- Only full-model default human prediction on well-annotated data is needed and runtime matters — skip -r since RNAfold is time-consuming and SSF gains are marginal

## Inputs

- RNA transcript sequences in FASTA format (regular text or gzip *.gz)
- CDS FASTA (paired in order with mRNA FASTA) when training
- Prebuilt hexamer table CSV (e.g. train_hexamer_table.csv)
- Prebuilt model, imputer, and scaler pickles (e.g. XGB_model_SIF_PF_SSF.pkl, imputer_SIF_PF_SSF.pkl, scaler_SIF_PF_SSF.pkl)
- Secondary structure k-mer tables ss_table_k1.csv–k5.csv (prefix passed via -k)

## Outputs

- CSV prediction file labeling each transcript as 'lncrna' or 'mrna' (e.g. lncdc_human.csv)
- Hexamer table CSV, XGBoost model pickle, imputer pickle, and scaler pickle when training
- Five secondary structure k-mer tables (ss_table_k1.csv–ss_table_k5.csv) when training with -r

## How to apply

Start from RNA sequences in FASTA (plain or gzip-compressed) and compute hexamer-based sequence intrinsic features, flexible ORF features from four ORF types, and protein features; when secondary structure is needed, first fold sequences with RNAfold (ViennaRNA) — verify with `import RNA` — and derive the sequence-and-secondary-structure (SASS) k-mer score tables (ss_table_k1 through k5). For prediction, pass the prebuilt artifacts explicitly: the hexamer table (-x), the XGBoost model pickle (-m, e.g. XGB_model_SIF_PF_SSF.pkl), the imputer (-p), the scaler (-s), and enable -r with the ss table prefix (-k). For training, run lncDC-train.py with -m mRNA FASTA, -c CDS FASTA (same order/count as mRNAs), -l lncRNA FASTA, and -r to also generate the five ss k-mer tables. The decision point on using -r trades runtime (RNAfold is time-consuming) for the marginal accuracy gain demonstrated by the 6 selected SSFs.

## Related tools

- **LncDC** (Main tool implementing SIF/SSF/PF feature extraction and XGBoost prediction/training over RNA FASTA) — https://github.com/lim74/LncDC
- **XGBoost** (Classifier consuming the extracted features; model shipped as XGB_model_SIF_PF.pkl / XGB_model_SIF_PF_SSF.pkl) — https://github.com/lim74/LncDC
- **ViennaRNA** (Optional dependency; RNAfold computes RNA secondary structures used to build the SASS ss k-mer tables (required only with -r))
- **scikit-learn** (Imputer and scaler pickles applied to extracted features before classification; metric computation for ablation) — https://github.com/lim74/LncDC

## Examples

```
python bin/lncDC.py -i test/human_test_toy.fasta -o lncdc_human.csv -r -t 8
```

## Evaluation signals

- On human H-Test, the SIF+PF-only condition (no SSFs) should reproduce accuracy 0.9783, precision 0.9746, MCC 0.9567; the full model gives accuracy 0.9799, precision 0.9786, MCC 0.9598
- An SSF-only model (6 selected SSFs) should reach accuracy 0.8680, precision 0.8929, specificity 0.8998 on H-Test
- Training with -r must emit exactly five ss tables: ss_table_k1.csv through ss_table_k5.csv alongside the hexamer table, model, imputer, and scaler
- Verify the optional ViennaRNA dependency: `import RNA` in Python raises no errors before enabling -r
- Output CSV labels must be exactly 'lncrna' or 'mrna' per transcript, with one row per input FASTA record

## Limitations

- Secondary structure feature computation by RNAfold (ViennaRNA) is time-consuming for prediction and training with -r
- The enhancement from the 6 selected SSFs is marginal — SIFs and PFs already achieved high accuracy (accuracy 0.9799), so further improvement is difficult
- For non-model organisms lacking well-annotated lncRNAs for training, models must be trained on data from evolutionary close model organisms

## Evidence

- [abstract] we present our tool LncDC (Long non-coding RNA detection) that is able to accurately predict lncRNAs with an XGBoost model: "we present our tool LncDC (Long non-coding RNA detection) that is able to accurately predict lncRNAs with an XGBoost model"
- [abstract] the use of sequence and secondary structure (SASS) k-mer score features and flexible ORF features improved the classification capability of LncDC: "the use of sequence and secondary structure (SASS) k-mer score features and flexible ORF features improved the classification capability of LncDC"
- [readme] The -r parameter will turn on the prediction with secondary structure features, and the '-k' parameter will provide the prefix of the requried train_ss_table(s).: "The -r parameter will turn on the prediction with secondary structure features, and the '-k' parameter will provide the prefix of the requried train_ss_table(s)."
- [methods] Run lncDC-train.py to obtain the hexamer table of your own training data: "Run lncDC-train.py to obtain the hexamer table of your own training data"
- [discussion] the LncDC model with only the 6 selected SSFs performed well in H-Test (precision:0.8929, specificity:0.8998): "the LncDC model with only the 6 selected SSFs performed well in H-Test (precision:0.8929, specificity:0.8998)"
