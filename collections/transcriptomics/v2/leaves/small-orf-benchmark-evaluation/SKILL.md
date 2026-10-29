---
name: small-orf-benchmark-evaluation
description: Use when you need to test whether an lncRNA/coding-potential classifier
  handles transcripts with small ORFs, by benchmarking it against established methods
  on the species-specific sORF test sets from the LncCat study.
when_to_use_negative:
- Input transcripts contain long ORFs only — use the five-species long-ORF benchmark
  datasets instead, where LncCat's improvements are 1.90%–21.50% rather than sORF-specific
- The target species lacks sufficient annotation to build a species model (the article
  notes species may lack annotations, making it impossible to build models for all
  species)
- The task is functional annotation of sORF-translated peptides — the study explicitly
  defers collecting sORF peptide sequences to future work
edam_operation: http://edamontology.org/operation_0464
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3512
tools:
- name: LncCat
  role: CatBoost-based classifier with ORF-attention features being benchmarked on
    sORF test sets; cutoff 0.5
  repo: https://github.com/a525076133/LncCat
- name: CPAT
  role: Comparison method; logistic regression model with Fickett TESTCODE score and
    hexamer score, v3.0.0
  repo: http://cpat.readthedocs.io/en/latest
- name: CNCI
  role: Comparison method; SVM with adjoining nucleotide triplets and codon bias,
    version 2
  repo: http://www.bioinfo.org/software/cnci
- name: PLEK
  role: Comparison method; improved k-mer scheme with SVM, v1.2
  repo: http://sourceforge.net/projects/plek
- name: CPC2
  role: Comparison method; alignment-free sequence intrinsic features, v1.0.1
  repo: http://cpc2.cbi.pku.edu.cn
- name: LncFinder
  role: Comparison method; SVM with sequence intrinsic, secondary-structure, and EIIP
    physicochemical features, v1.1.5
  repo: http://github.com/HAN-Siyu/LncFinder
- name: CPPred
  role: Comparison method using SVM
  repo: http://www.rnabinding.com/CPPred
- name: mRNN
  role: Comparison method; recursive neural network with one-hot encoded RNA sequences
  repo: http://github.com/hendrixlab/mRNN
- name: RNAsamba
  role: Comparison method; neural network using whole-sequence and ORF information
    via rnasamba classify
  repo: http://github.com/apcamargo/RNAsamba
- name: CatBoost
  role: Category-boosting algorithm underlying the LncCat classification model
  repo: https://github.com/a525076133/LncCat
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.02.012
    title: 'LncCat: An ORF attention model to identify LncRNA based on ensemble learning
      strategy and fused sequence information'
  source_content_hash: sha256:319eb82ead9e9a7ff86206eb8fdc64cf035705a0a756859521abbf2062d78eba
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
  edam_operation: http://edamontology.org/operation_0464
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# small-orf-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark protocol for evaluating whether lncRNA identification methods underperform on transcripts with small ORFs (sORFs), by running LncCat and eight comparison methods (CPAT, CNCI, PLEK, CPC2, LncFinder, CPPred, mRNN, RNAsamba) on human-, mouse-, and zebrafish-sORF test datasets and computing PRE, SEN, SPE, ACC, F1, MCC, and AUC.

## When to use

Use when you need to test whether an lncRNA/coding-potential classifier handles transcripts with small ORFs, by benchmarking it against established methods on the species-specific sORF test sets from the LncCat study.

## When NOT to use

- Input transcripts contain long ORFs only — use the five-species long-ORF benchmark datasets instead, where LncCat's improvements are 1.90%–21.50% rather than sORF-specific
- The target species lacks sufficient annotation to build a species model (the article notes species may lack annotations, making it impossible to build models for all species)
- The task is functional annotation of sORF-translated peptides — the study explicitly defers collecting sORF peptide sequences to future work

## Inputs

- Human-sORF test dataset FASTA (639 lncRNAs, 641 PCTs, Ensembl-derived)
- Mouse-sORF test dataset FASTA (993 lncRNAs, 846 PCTs)
- Zebrafish-sORF test dataset FASTA (497 lncRNAs, 387 PCTs)
- LncCat model/code from https://github.com/a525076133/LncCat
- Pre-trained RNAsamba weights (full_length_weights.hdf5 or partial_length_weights.hdf5)

## Outputs

- Per-method per-dataset metrics table of PRE, SEN, SPE, ACC, F1, MCC, AUC with lncRNAs as the positive class
- MCC and F1 comparison table mirroring article Tables 8 and 9 and Supplementary Tables S12–S14
- Relative MCC-improvement percentages of the best method over the second-best on each sORF dataset

## How to apply

Download the three independent sORF test datasets from the LncCat deposit (human-sORF: 639 lncRNAs / 641 PCTs; mouse-sORF: 993 lncRNAs / 846 PCTs; zebrafish-sORF: 497 lncRNAs / 387 PCTs, all derived from Ensembl). Run LncCat — a CatBoost classifier fusing codon-related, GC-related, sequence-related, peptide-related, and ORF-attention features — with the prediction-score cutoff fixed at 0.5 (score >= 0.5 = lncRNA). Run the eight comparison methods at their documented versions: CPAT v3.0.0 (logistic regression), CNCI v2 and PLEK v1.2 (SVM), CPC2 v1.0.1, LncFinder v1.1.5, CPPred (SVM), and the deep-learning methods mRNN and RNAsamba. For each method on each dataset, compute PRE, SEN, SPE, ACC, F1, MCC, and AUC treating lncRNAs as the positive class, then assemble MCC and F1 into tables mirroring Tables 8–9 and Supplementary Tables S12–S14. Validate by confirming the candidate method's MCC exceeds the second-best method by at least 11.90% (human), 12.96% (mouse), and 42.61% (zebrafish), with LncCat reference MCCs of 0.8858 (human-sORF), 0.7971 (mouse-sORF), and 0.8735 (zebrafish-sORF).

## Related tools

- **LncCat** (CatBoost-based classifier with ORF-attention features being benchmarked on sORF test sets; cutoff 0.5) — https://github.com/a525076133/LncCat
- **CPAT** (Comparison method; logistic regression model with Fickett TESTCODE score and hexamer score, v3.0.0) — http://cpat.readthedocs.io/en/latest
- **CNCI** (Comparison method; SVM with adjoining nucleotide triplets and codon bias, version 2) — http://www.bioinfo.org/software/cnci
- **PLEK** (Comparison method; improved k-mer scheme with SVM, v1.2) — http://sourceforge.net/projects/plek
- **CPC2** (Comparison method; alignment-free sequence intrinsic features, v1.0.1) — http://cpc2.cbi.pku.edu.cn
- **LncFinder** (Comparison method; SVM with sequence intrinsic, secondary-structure, and EIIP physicochemical features, v1.1.5) — http://github.com/HAN-Siyu/LncFinder
- **CPPred** (Comparison method using SVM) — http://www.rnabinding.com/CPPred
- **mRNN** (Comparison method; recursive neural network with one-hot encoded RNA sequences) — http://github.com/hendrixlab/mRNN
- **RNAsamba** (Comparison method; neural network using whole-sequence and ORF information via rnasamba classify) — http://github.com/apcamargo/RNAsamba
- **CatBoost** (Category-boosting algorithm underlying the LncCat classification model) — https://github.com/a525076133/LncCat

## Examples

```
rnasamba classify mouse_sorf_predictions.tsv mouse-sORF_test.fasta full_length_weights.hdf5
```

## Evaluation signals

- LncCat MCC on sORF test datasets reaches the reported values: 0.8858 (human-sORF), 0.7971 (mouse-sORF), 0.8735 (zebrafish-sORF)
- LncCat's MCC exceeds the second-best method by at least 11.90% (human), 12.96% (mouse), and 42.61% (zebrafish) on sORF datasets
- All seven metrics (PRE, SEN, SPE, ACC, F1, MCC, AUC) computed with lncRNAs as the positive class for each of the nine methods on each of the three datasets
- Dataset class counts match: human 639/641, mouse 993/846, zebrafish 497/387 lncRNAs/PCTs
- LncCat predictions reproduced with the fixed cutoff: score >= 0.5 assigned to lncRNA, < 0.5 to PCT

## Limitations

- The article abstract and the GitHub README disagree on the minimum sORF MCC improvements (article: 11.90%/12.96%/42.61%; README: 6.88%/22.98%/39.43%), so validation thresholds should be checked against the published tables
- CatBoost hyperparameter tuning settings and the versions of CatBoost and BERT are not reported, hindering exact reproducibility of the LncCat model
- Species with insufficient annotations cannot have models built, and sORF-translated peptide sequences were not collected in this study
- The LncCat web server (http://cczubio.top/lnccat/) returns results only via user email rather than directly in the interface

## Evidence

- [readme] README discrepancy on sORF improvements: [evidence span withheld — anchor status: external_doc]
