---
name: lncrna-coding-potential-classification
description: Use when you need to distinguish lncRNAs from PCTs in FASTA transcript
  sequences across species (human, mouse, zebrafish, wheat, chicken), especially for
  transcripts containing sORFs where existing coding-potential tools perform poorly.
when_to_use_negative:
- Input transcripts lack species annotations and the target species has no suitable
  training data, making model construction infeasible (the article notes 'Species
  may lack annotations, and it is impossible to build models for all species.')
- You need sORF-translated peptide functional annotation, which is out of scope of
  this method (future work per the discussion)
- A feature table is already provided and only model scoring is needed rather than
  the full five-type feature extraction pipeline
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: LncCat
  role: CatBoost-based classifier with ORF-attention features performing the lncRNA
    identification
  repo: https://github.com/a525076133/LncCat
- name: CatBoost
  role: Gradient-boosting decision-tree model underlying LncCat classification
  repo: https://github.com/a525076133/LncCat
- name: RNAsamba
  role: Competing neural-network coding-potential method used as a baseline for benchmarking
  repo: http://github.com/apcamargo/RNAsamba
- name: CPAT
  role: Competing logistic regression method (v3.0.0) used as a baseline for benchmarking
  repo: http://cpat.readthedocs.io/en/latest
- name: LncFinder
  role: Competing SVM-based method (v1.1.5) used as a baseline for benchmarking
  repo: http://github.com/HAN-Siyu/LncFinder
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2023.02.012
    title: 'LncCat: An ORF attention model to identify LncRNA based on ensemble learning
      strategy and fused sequence information'
  source_content_hash: sha256:f557fd8021c837bd539a5649aea92dff11ad2ff5ce606325443b54ce935f4970
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
  - COME
  - CPAT
  - CPC2
  - CPPred
  - CatBoost
  - LncCat
  - LncDC
  - LncFinder
  - PLEK
  - Python
  - RNAsamba
  - mRNN
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# lncrna-coding-potential-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classify transcripts as lncRNA vs. protein-coding transcripts (PCTs) using CatBoost-based ensemble learning with fused codon-related, GC-related, sequence-related, peptide-related, and ORF-attention features, following the LncCat method. Particularly effective for lncRNAs with small ORFs (sORFs) where CPAT, CNCI, PLEK, CPC2, LncFinder, CPPred, mRNN, and RNAsamba underperform.

## When to use

Use when you need to distinguish lncRNAs from PCTs in FASTA transcript sequences across species (human, mouse, zebrafish, wheat, chicken), especially for transcripts containing sORFs where existing coding-potential tools perform poorly.

## When NOT to use

- Input transcripts lack species annotations and the target species has no suitable training data, making model construction infeasible (the article notes 'Species may lack annotations, and it is impossible to build models for all species.')
- You need sORF-translated peptide functional annotation, which is out of scope of this method (future work per the discussion)
- A feature table is already provided and only model scoring is needed rather than the full five-type feature extraction pipeline

## Inputs

- Transcript sequences in FASTA format (lncRNA and PCT classes) from species such as human, mouse, zebrafish, wheat, or chicken
- sORF test datasets (e.g., human: 639 lncRNAs / 641 PCTs; mouse: 993 lncRNAs / 846 PCTs; zebrafish: 497 lncRNAs / 387 PCTs from Ensembl)
- Precomputed five-type feature matrices (codon-related, GC-related, sequence-related, peptide-related, ORF-attention) from the LncCat repository

## Outputs

- Per-transcript prediction scores with binary classification (>= 0.5 as lncRNA, < 0.5 as PCT)
- Evaluation metrics (ACC, SEN, SPE, F1, PRE, MCC, AUC) per species or benchmark dataset
- Benchmark comparison tables against competing methods (CPAT, CNCI, PLEK, CPC2, LncFinder, CPPred, mRNN, RNAsamba)

## How to apply

Encode each transcript with five feature types: codon-related (stop codon count/frequency, Fickett score), GC-related (GC1/GC2/GC3 and frame scores), sequence-related (length, k-mer, CTD, hexamer score), peptide-related (Mw, PI, Gravy, instability index), and ORF-attention features extracted with Orfipy using Standard Code transl_table=1 (start codons ATG/TTG/CTG) and encoded with a pre-trained BERT model. Train the CatBoost gradient-boosting classifier on training features, applying a fixed decision cutoff of 0.5 (score >= 0.5 predicted as lncRNA, < 0.5 as PCT). Datasets should be pre-filtered with CD-HIT at 80% similarity and transcripts shorter than 200 nt removed, then split 70/10/20 into training/validation/test. Evaluate with PRE, SEN, SPE, ACC, F1, MCC, and AUC using lncRNAs as the positive class; correct application on the human benchmark yields MCC 0.9503 and F1 0.9742.

## Related tools

- **LncCat** (CatBoost-based classifier with ORF-attention features performing the lncRNA identification) — https://github.com/a525076133/LncCat
- **CatBoost** (Gradient-boosting decision-tree model underlying LncCat classification) — https://github.com/a525076133/LncCat
- **RNAsamba** (Competing neural-network coding-potential method used as a baseline for benchmarking) — http://github.com/apcamargo/RNAsamba
- **CPAT** (Competing logistic regression method (v3.0.0) used as a baseline for benchmarking) — http://cpat.readthedocs.io/en/latest
- **LncFinder** (Competing SVM-based method (v1.1.5) used as a baseline for benchmarking) — http://github.com/HAN-Siyu/LncFinder

## Examples

```
rnasamba classify results.tsv sorf_test.fa full_length_weights.hdf5
```

## Evaluation signals

- MCC on the human benchmark test set should reach 0.9503 (F1 0.9742), with mouse 0.9219, zebrafish 0.8591, wheat 0.8672, and chicken 0.9047
- On sORF test datasets, MCC should substantially exceed the best competing method (human-sORF MCC 0.8858, mouse-sORF MCC 0.7971, zebrafish-sORF MCC 0.8735 per task cards)
- Prediction scores partition transcripts with the fixed cutoff: score >= 0.5 is lncRNA, score < 0.5 is PCT
- UMAP visualization of the feature space should show lncRNAs and PCTs significantly separated, especially in ORF-attention features
- Expected MCC improvement ranges over the eight competing methods: 1.90%~7.82% (human), 1.49%~17.63% (mouse), 6.11%~21.50% (zebrafish)

## Limitations

- CatBoost hyperparameter tuning settings and the versions of CatBoost and BERT are not reported, limiting exact reproducibility
- The published MCC-improvement figures for sORF datasets differ between article text (at least 11.90%, 12.96%, 42.61%) and the repository README (6.88%, 22.98%, 39.43%)
- Results from the deployed web server (http://cczubio.top/lnccat/) are returned only via user email rather than directly in the interface
- Cross-species models degrade on distant species: human, mouse, and zebrafish models score only MCC 0.67, 0.63, and 0.66 on wheat

## Evidence

- [readme] LncCat dramatically improves the MCC by at least 6.88%, 22.98% and 39.43% on sORFs test datasets of human, mouse and zebrafish: [evidence span withheld — anchor status: external_doc]
