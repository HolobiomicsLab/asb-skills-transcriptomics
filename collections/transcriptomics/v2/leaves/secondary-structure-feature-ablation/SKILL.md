---
name: secondary-structure-feature-ablation
description: Use when asked whether a specific feature group (e.g. the 6 SSFs) contributes
  to a trained XGBoost model's performance and you need a full-vs-reduced-feature
  ablation table with accuracy, precision, MCC and specificity on a held-out FASTA
  set.
when_to_use_negative:
- The goal is de novo lncRNA prediction on new RNA-Seq assemblies rather than evaluating
  which features drive the classifier's performance
- You need to train new models from your own mRNA/CDS/lncRNA FASTA data (use the lncDC-train.py
  workflow instead of running the prebuilt ablation)
- The comparison targets a non-human organism without prebuilt SS k-mer tables, since
  self-training with -r requires time-consuming RNAfold secondary structure calculation
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0097
tools:
- name: LncDC
  role: XGBoost-based lncRNA/mRNA classifier whose prebuilt models (XGB_model_SIF_PF_SSF.pkl,
    XGB_model_SIF_PF.pkl) are invoked under different feature-subset conditions for
    the ablation
  repo: https://github.com/lim74/LncDC
- name: XGBoost
  role: Underlying classifier of the LncDC models being compared in the ablation conditions
- name: ViennaRNA
  role: Provides RNAfold, invoked when -r enables secondary structure feature extraction
    for SSF-based predictions
- name: scikit-learn
  role: Used to compute accuracy, precision, MCC and specificity for each ablation
    condition
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-022-22082-7
    title: 'LncDC: a machine learning-based tool for long non-coding RNA detection
      from RNA-Seq data'
  source_content_hash: sha256:9ddbfc7a64b317367c4111c92f91c939767c8a9eb72d794bd219b409d39ddd04
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0097
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# secondary-structure-feature-ablation

## Summary

Ablation protocol for the LncDC lncRNA classifier that quantifies the contribution of the 6 selected secondary structure features (SSFs) by comparing full-model prediction (SIF+PF+SSF) against a SIF+PF-only model and an SSF-only model on the human H-Test FASTA dataset.

## When to use

Use when asked whether a specific feature group (e.g. the 6 SSFs) contributes to a trained XGBoost model's performance and you need a full-vs-reduced-feature ablation table with accuracy, precision, MCC and specificity on a held-out FASTA set.

## When NOT to use

- The goal is de novo lncRNA prediction on new RNA-Seq assemblies rather than evaluating which features drive the classifier's performance
- You need to train new models from your own mRNA/CDS/lncRNA FASTA data (use the lncDC-train.py workflow instead of running the prebuilt ablation)
- The comparison targets a non-human organism without prebuilt SS k-mer tables, since self-training with -r requires time-consuming RNAfold secondary structure calculation

## Inputs

- human H-Test RNA transcript sequences in FASTA format (regular text or gzip *.gz)
- prebuilt LncDC model artifacts: XGB_model_SIF_PF_SSF.pkl, XGB_model_SIF_PF.pkl, imputer_SIF_PF*.pkl, scaler_SIF_PF*.pkl
- hexamer table (train_hexamer_table.csv) and sequence/secondary-structure k-mer tables (prefix train_ss_table, k1-k5)

## Outputs

- per-condition prediction CSVs labeling each transcript as 'lncrna' or 'mrna'
- ablation_results.csv with accuracy, precision, MCC and specificity for the full (SIF+PF+SSF), SIF+PF-only, and SSF-only conditions

## How to apply

Load the human H-Test FASTA sequences and the prebuilt LncDC artifacts (XGB_model_SIF_PF_SSF.pkl, imputer, scaler, hexamer table, ss k-mer tables). Run the full-feature baseline with XGBoost, enabling secondary structure features via the -r flag (which invokes RNAfold from ViennaRNA and is slower), then re-run with the SIF+PF-only model (XGB_model_SIF_PF.pkl) to remove the 6 SSFs, and finally run using only the 6 selected SSFs. Compute accuracy, precision, MCC and specificity in scikit-learn for each condition and assemble them into ablation_results.csv. Expect the SIF+PF-only drop (accuracy 0.9799 to 0.9783, precision 0.9786 to 0.9746, MCC 0.9598 to 0.9567) to be small because SIFs and PFs already achieve high accuracy, while the SSF-only model should reach accuracy 0.8680 with precision 0.8929.

## Related tools

- **LncDC** (XGBoost-based lncRNA/mRNA classifier whose prebuilt models (XGB_model_SIF_PF_SSF.pkl, XGB_model_SIF_PF.pkl) are invoked under different feature-subset conditions for the ablation) — https://github.com/lim74/LncDC
- **XGBoost** (Underlying classifier of the LncDC models being compared in the ablation conditions)
- **ViennaRNA** (Provides RNAfold, invoked when -r enables secondary structure feature extraction for SSF-based predictions)
- **scikit-learn** (Used to compute accuracy, precision, MCC and specificity for each ablation condition)

## Examples

```
python bin/lncDC.py -i H-Test.fasta -o lncdc_SSF_full.csv -r -k data/train_ss_table -m data/XGB_model_SIF_PF_SSF.pkl -p data/imputer_SIF_PF_SSF.pkl -s data/scaler_SIF_PF_SSF.pkl -t 8
```

## Evaluation signals

- SIF+PF-only condition reproduces the reported metrics: accuracy 0.9783, precision 0.9746, MCC 0.9567 (vs full-model baseline accuracy 0.9799, precision 0.9786, MCC 0.9598)
- SSF-only condition reaches accuracy 0.8680, precision 0.8929 and specificity 0.8998 on H-Test
- Ablation results table contains all four metrics (accuracy, precision, MCC, specificity) for each of the three conditions, written to ablation_results.csv
- Full-feature baseline (with -r) reproduces the reference accuracy of 0.9799 before any features are removed
- Prediction output CSV labels every transcript as 'lncrna' or 'mrna' and covers the same transcript IDs as the H-Test FASTA input

## Limitations

- Secondary structure prediction with RNAfold (ViennaRNA) makes SSF-based runs substantially slower than SIF+PF-only runs
- The enhancement from the 6 selected SSFs is acknowledged as marginal because SIFs and PFs had already achieved a high accuracy and further improvement became difficult
- Secondary structure features require ViennaRNA to be installed (optional dependency, e.g. conda install -c bioconda viennarna) and confirmed via 'import RNA'

## Evidence

- [discussion] the LncDC model with only the 6 selected SSFs performed well in H-Test (precision:0.8929, specificity:0.8998): "the LncDC model with only the 6 selected SSFs performed well in H-Test (precision:0.8929, specificity:0.8998)"
- [discussion] SIFs and PFs already achieve high accuracy: "This might be because SIFs and PFs had already achieved a high accuracy and further improvement became difficult"
- [readme] README SSF prediction example: "python bin/lncDC.py -i test/mouse_test_toy.fasta -o lncdc_mouse.csv -r -t 8"
- [readme] README runtime caveat for SSF features: "This may take a longer time because secondary structure calculation by RNAfold (The main program of ViennaRNA) is time-consuming."
- [intro] novel SSF design rationale: "We also designed novel secondary structure features that integrate both primary sequence and secondary structure information from RNA transcripts"
