---
name: orf-attention-feature-scoring
description: Use when classifying transcript FASTA sequences (especially transcripts
  with small ORFs) as lncRNA or PCT, or when benchmarking coding-potential methods
  on sORF test sets where CPAT/CNCI/PLEK/CPC2/LncFinder/CPPred/mRNN/RNAsamba underperform.
when_to_use_negative:
- When the goal is de novo annotation of sORF-translated peptide functions — the article
  states such peptide sequences were not collected ('In the following work, we will
  collect the sORF-translated peptide sequences')
- When building a model for poorly annotated species — the article warns 'Species
  may lack annotations, and it is impossible to build models for all species.'
- When input sequences are protein sequences or already-encoded feature tables rather
  than nucleotide transcript FASTA
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: LncCat
  role: CatBoost-based classifier fusing codon-, GC-, sequence-, peptide-, and ORF-attention
    features to score transcripts as lncRNA or PCT
  repo: https://github.com/a525076133/LncCat
- name: CatBoost
  role: Gradient-boosting ensemble that builds the LncCat prediction model from the
    fused five-type feature vectors
- name: CPAT
  role: Comparison method (v3.0.0 logistic regression model) benchmarked on the same
    sORF test datasets
  repo: http://cpat.readthedocs.io/en/latest
- name: CNCI
  role: Comparison method (version 2, SVM with adjoining nucleotide triplets) benchmarked
    on the same sORF test datasets
  repo: http://www.bioinfo.org/software/cnci
- name: CPC2
  role: Comparison method (v1.0.1, alignment-free intrinsic features) benchmarked
    on the same sORF test datasets
  repo: http://cpc2.cbi.pku.edu.cn
- name: LncFinder
  role: Comparison method (v1.1.5, SVM with secondary-structure and EIIP features)
    benchmarked on the same sORF test datasets
  repo: http://github.com/HAN-Siyu/LncFinder
- name: PLEK
  role: Comparison method (v1.2, improved k-mer SVM) benchmarked on the same sORF
    test datasets
  repo: http://sourceforge.net/projects/plek
- name: CPPred
  role: Comparison method (SVM) benchmarked on the same sORF test datasets
  repo: http://www.rnabinding.com/CPPred
- name: mRNN
  role: Comparison method (Recursive Neural Network on one-hot encoded sequences)
    benchmarked on the same sORF test datasets
  repo: http://github.com/hendrixlab/mRNN
- name: RNAsamba
  role: Comparison method (neural network using whole-sequence and ORF information)
    benchmarked on the same sORF test datasets
  repo: http://github.com/apcamargo/RNAsamba
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.02.012
    title: 'LncCat: An ORF attention model to identify LncRNA based on ensemble learning
      strategy and fused sequence information'
  source_content_hash: sha256:c6667a68de6244da2f2c4b9cd5e92e51e4b03e7df661f9846f6398c38b8e8326
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
  - CatBoost
  - LncCat
  - LncFinder
  - PLEK
  - RNAsamba
  - mRNN
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# orf-attention-feature-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores transcript sequences for lncRNA vs. protein-coding-transcript (PCT) identity using LncCat's five fused feature types — codon-related, GC-related, sequence-related, peptide-related, and ORF-attention features — with a CatBoost ensemble classifier, emphasizing ORF-derived attention representations that separate lncRNAs with small ORFs (sORFs) from PCTs.

## When to use

Use when classifying transcript FASTA sequences (especially transcripts with small ORFs) as lncRNA or PCT, or when benchmarking coding-potential methods on sORF test sets where CPAT/CNCI/PLEK/CPC2/LncFinder/CPPred/mRNN/RNAsamba underperform.

## When NOT to use

- When the goal is de novo annotation of sORF-translated peptide functions — the article states such peptide sequences were not collected ('In the following work, we will collect the sORF-translated peptide sequences')
- When building a model for poorly annotated species — the article warns 'Species may lack annotations, and it is impossible to build models for all species.'
- When input sequences are protein sequences or already-encoded feature tables rather than nucleotide transcript FASTA

## Inputs

- Transcript sequences in FASTA format (five species datasets: human, mouse, zebrafish, wheat, chicken)
- sORF test datasets from the LncCat deposit (human-sORF: 639 lncRNAs / 641 PCTs; mouse-sORF: 993 lncRNAs / 846 PCTs; zebrafish-sORF: 497 lncRNAs / 387 PCTs, derived from Ensembl)
- Pre-trained BERT model for ORF peptide encoding
- CatBoost classifier (LncCat model weights)

## Outputs

- Per-transcript prediction score with >= 0.5 classified as lncRNA and < 0.5 as PCT
- Per-dataset metric table of PRE, SEN, SPE, ACC, F1, MCC, and AUC (lncRNAs as positive class)
- UMAP visualization of lncRNA/PCT separation in ORF-attention feature space

## How to apply

Extract ORFs from each transcript with Orfipy using the Standard Code (transl_table=1) with start codons ATG, TTG, CTG and stop codons TAA, TAG, TGA, then encode the ORF peptide sequences with a pre-trained BERT model to obtain contextual vector representations (the ORF-attention features). Combine these with codon-related, GC-related, sequence-related, and peptide-related features computed from the transcript sequence, and feed the fused five-type feature vector into the LncCat CatBoost classifier. Apply a prediction-score cutoff of 0.5: scores >= 0.5 call lncRNA, scores < 0.5 call PCT. Evaluate against lncRNAs-as-positive-class metrics (PRE, SEN, SPE, ACC, F1, MCC, AUC) on independent sORF test sets, expecting LncCat MCCs near 0.8858 (human-sORF), 0.7971 (mouse-sORF), and 0.8735 (zebrafish-sORF).

## Related tools

- **LncCat** (CatBoost-based classifier fusing codon-, GC-, sequence-, peptide-, and ORF-attention features to score transcripts as lncRNA or PCT) — https://github.com/a525076133/LncCat
- **CatBoost** (Gradient-boosting ensemble that builds the LncCat prediction model from the fused five-type feature vectors)
- **CPAT** (Comparison method (v3.0.0 logistic regression model) benchmarked on the same sORF test datasets) — http://cpat.readthedocs.io/en/latest
- **CNCI** (Comparison method (version 2, SVM with adjoining nucleotide triplets) benchmarked on the same sORF test datasets) — http://www.bioinfo.org/software/cnci
- **CPC2** (Comparison method (v1.0.1, alignment-free intrinsic features) benchmarked on the same sORF test datasets) — http://cpc2.cbi.pku.edu.cn
- **LncFinder** (Comparison method (v1.1.5, SVM with secondary-structure and EIIP features) benchmarked on the same sORF test datasets) — http://github.com/HAN-Siyu/LncFinder
- **PLEK** (Comparison method (v1.2, improved k-mer SVM) benchmarked on the same sORF test datasets) — http://sourceforge.net/projects/plek
- **CPPred** (Comparison method (SVM) benchmarked on the same sORF test datasets) — http://www.rnabinding.com/CPPred
- **mRNN** (Comparison method (Recursive Neural Network on one-hot encoded sequences) benchmarked on the same sORF test datasets) — http://github.com/hendrixlab/mRNN
- **RNAsamba** (Comparison method (neural network using whole-sequence and ORF information) benchmarked on the same sORF test datasets) — http://github.com/apcamargo/RNAsamba

## Examples

```
orfipy transcripts.fasta --pep orfs.pep --table 1 --min 1  # then encode ORF peptides with pre-trained BERT and score fused features with the LncCat CatBoost model (cutoff 0.5)
```

## Evaluation signals

- Prediction scores dichotomized at the documented cutoff 0.5 reproduce the reported sORF MCCs: LncCat human-sORF MCC 0.8858, mouse-sORF MCC 0.7971, zebrafish-sORF MCC 0.8735
- On sORF test sets, LncCat's MCC exceeds the second-best method by at least 11.90% (human), 12.95-12.96% (mouse), and 42.61% (zebrafish)
- On full five-species datasets, expect MCC values of 0.9503 (human), 0.9219 (mouse), 0.8591 (zebrafish), 0.8672 (wheat), and 0.9047 (chicken)
- UMAP visualization of the ORF-attention features should show clear separation between lncRNA and PCT clusters
- Cross-species sanity check: human/mouse/zebrafish models score well on each other's datasets but degrade on wheat (MCC ~0.67, 0.63, 0.66)

## Limitations

- CatBoost and BERT versions, and CatBoost hyperparameter tuning settings, are not reported, limiting exact reproducibility
- Results on the deployed web server (http://cczubio.top/lnccat/) are returned only via user email rather than directly in the interface
- sORF-translated peptide sequences were not collected in this study, so downstream functional annotation of sORF peptides is out of scope
- Species may lack annotations, making it impossible to build LncCat models for all species

## Evidence
