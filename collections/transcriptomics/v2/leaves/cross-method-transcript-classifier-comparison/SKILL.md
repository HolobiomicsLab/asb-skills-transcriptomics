---
name: cross-method-transcript-classifier-comparison
description: Use when you need to reproduce or extend head-to-head evaluation of lncRNA
  coding-potential classifiers on sORF-enriched test sets, computing PRE, SEN, SPE,
  ACC, F1, MCC, and AUC per species.
when_to_use_negative:
- The goal is training a new model rather than comparing existing classifiers on held-out
  test data (use the LncCat training pipeline with CD-HIT 80% redundancy reduction
  and 70/10/20 split instead)
- Input transcripts are generic long-ORF benchmark sets from species like wheat or
  chicken rather than the sORF test sets — different reported MCC ranges (e.g. wheat
  MCC 0.8672) apply
- Only one method is being assessed with no competitor baseline, since the skill's
  core is the eight-method head-to-head comparison
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3308
tools:
- name: LncCat
  role: CatBoost-based classifier under evaluation; produces lncRNA/PCT prediction
    scores with cutoff 0.5
  repo: https://github.com/a525076133/LncCat
- name: CPAT
  role: Comparison method (v3.0.0 logistic regression with Fickett TESTCODE and hexamer
    scores)
  repo: http://cpat.readthedocs.io/en/latest
- name: CNCI
  role: Comparison method (version 2, SVM with adjoining nucleotide triplets)
  repo: http://www.bioinfo.org/software/cnci
- name: PLEK
  role: Comparison method (v1.2, improved k-mer scheme with SVM)
  repo: http://sourceforge.net/projects/plek
- name: CPC2
  role: Comparison method (v1.0.1, alignment-free sequence intrinsic features)
  repo: http://cpc2.cbi.pku.edu.cn
- name: LncFinder
  role: Comparison method (v1.1.5, SVM with secondary structure and EIIP-derived features)
  repo: http://github.com/HAN-Siyu/LncFinder
- name: CPPred
  role: Comparison method (SVM-based coding potential predictor)
  repo: http://www.rnabinding.com/CPPred
- name: mRNN
  role: Comparison method (recursive neural network on one-hot encoded RNA)
  repo: http://github.com/hendrixlab/mRNN
- name: RNAsamba
  role: Comparison method (neural network using whole-sequence and ORF information;
    classify mode on FASTA input)
  repo: http://github.com/apcamargo/RNAsamba
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.02.012
    title: 'LncCat: An ORF attention model to identify LncRNA based on ensemble learning
      strategy and fused sequence information'
  source_content_hash: sha256:bbe45aa07ea9b026eb0d62a1091d6bccaff102638f0cf51db5d23a6469e2b1c8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/a525076133/LncCat
  repo_url: https://github.com/a525076133/LncCat
  tools:
  - CNCI
  - CPAT
  - CPC2
  - CPPred
  - LncCat
  - LncFinder
  - PLEK
  - RNAsamba
  - mRNN
  edam_operation: http://edamontology.org/operation_0436
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# cross-method-transcript-classifier-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark LncCat (CatBoost with codon-related, GC-related, sequence-related, peptide-related, and ORF-attention features, cutoff 0.5) against eight lncRNA identification methods (CPAT, CNCI, PLEK, CPC2, LncFinder, CPPred, mRNN, RNAsamba) on independent small-ORF (sORF) test datasets from human, mouse, and zebrafish.

## When to use

Use when you need to reproduce or extend head-to-head evaluation of lncRNA coding-potential classifiers on sORF-enriched test sets, computing PRE, SEN, SPE, ACC, F1, MCC, and AUC per species.

## When NOT to use

- The goal is training a new model rather than comparing existing classifiers on held-out test data (use the LncCat training pipeline with CD-HIT 80% redundancy reduction and 70/10/20 split instead)
- Input transcripts are generic long-ORF benchmark sets from species like wheat or chicken rather than the sORF test sets — different reported MCC ranges (e.g. wheat MCC 0.8672) apply
- Only one method is being assessed with no competitor baseline, since the skill's core is the eight-method head-to-head comparison

## Inputs

- Human-sORF test dataset FASTA (639 lncRNAs, 641 PCTs, Ensembl-derived)
- Mouse-sORF test dataset FASTA (993 lncRNAs, 846 PCTs)
- Zebrafish-sORF test dataset FASTA (497 lncRNAs, 387 PCTs)
- Pre-trained LncCat model (CatBoost) and per-method prediction outputs (RNAsamba needs HDF5 weights such as full_length_weights.hdf5)

## Outputs

- Per-method prediction score tables for each sORF dataset (lncRNA vs PCT calls)
- Metrics table with PRE, SEN, SPE, ACC, F1, MCC, and AUC per method per species (mirroring Tables S12-S14)
- Validation summary confirming LncCat's minimum MCC margins of 11.90% (human), 12.96% (mouse), and 42.61% (zebrafish)

## How to apply

Download the three independent sORF test datasets from the LncCat deposit (human-sORF: 639 lncRNAs / 641 PCTs; mouse-sORF: 993 lncRNAs / 846 PCTs; zebrafish-sORF: 497 lncRNAs / 387 PCTs, derived from Ensembl). Run LncCat on each dataset with the prediction-score cutoff set at 0.5 (score >= 0.5 classified as lncRNA). Run the eight comparison methods using the versions documented in the article: CPAT v3.0.0 (logistic regression), CNCI v2, PLEK v1.2, CPC2 v1.0.1, LncFinder v1.1.5, CPPred (SVM), mRNN, and RNAsamba (deep learning). Compute PRE, SEN, SPE, ACC, F1, MCC, and AUC for each method on each dataset with lncRNAs as the positive class, then assemble MCC and F1 into results tables mirroring Tables 8-9 and Tables S12-S14. Validate that LncCat's MCC exceeds the second-best method by at least 11.90% (human), 12.96% (mouse), and 42.61% (zebrafish), with reference values human-sORF MCC 0.8858, mouse-sORF MCC 0.7971, zebrafish-sORF MCC 0.8735.

## Related tools

- **LncCat** (CatBoost-based classifier under evaluation; produces lncRNA/PCT prediction scores with cutoff 0.5) — https://github.com/a525076133/LncCat
- **CPAT** (Comparison method (v3.0.0 logistic regression with Fickett TESTCODE and hexamer scores)) — http://cpat.readthedocs.io/en/latest
- **CNCI** (Comparison method (version 2, SVM with adjoining nucleotide triplets)) — http://www.bioinfo.org/software/cnci
- **PLEK** (Comparison method (v1.2, improved k-mer scheme with SVM)) — http://sourceforge.net/projects/plek
- **CPC2** (Comparison method (v1.0.1, alignment-free sequence intrinsic features)) — http://cpc2.cbi.pku.edu.cn
- **LncFinder** (Comparison method (v1.1.5, SVM with secondary structure and EIIP-derived features)) — http://github.com/HAN-Siyu/LncFinder
- **CPPred** (Comparison method (SVM-based coding potential predictor)) — http://www.rnabinding.com/CPPred
- **mRNN** (Comparison method (recursive neural network on one-hot encoded RNA)) — http://github.com/hendrixlab/mRNN
- **RNAsamba** (Comparison method (neural network using whole-sequence and ORF information; classify mode on FASTA input)) — http://github.com/apcamargo/RNAsamba

## Examples

```
rnasamba classify predictions.tsv human-sORF_test.fa full_length_weights.hdf5
```

## Evaluation signals

- LncCat's human-sORF MCC reaches approximately 0.8858, mouse-sORF MCC 0.7971, and zebrafish-sORF MCC 0.8735
- LncCat's MCC exceeds the second-best method by at least 11.90% (human), 12.96% (mouse), and 42.61% (zebrafish) on the sORF test sets
- Metrics are computed with lncRNAs as the positive class and PRE, SEN, SPE, ACC, F1, MCC, and AUC are all present per method per dataset
- Dataset sizes match the article: human 639/641, mouse 993/846, zebrafish 497/387 (lncRNA/PCT counts)
- LncCat classification threshold is 0.5 (score >= 0.5 labeled lncRNA) rather than a tuned per-dataset cutoff

## Limitations

- CatBoost hyperparameter tuning settings and the specific versions of CatBoost and BERT are not reported, limiting reproducibility of the LncCat model itself
- No public code repository beyond a web server (results returned via email) is listed for LncCat in some sections, though a GitHub deposit exists for datasets
- Species may lack annotations, making it impossible to build comparable models for all species

## Evidence
