---
name: mcc-f1-metric-computation
description: Use when benchmarking lncRNA identification methods (or any binary sequence
  classifiers) on the human/mouse/zebrafish sORF test datasets and you need MCC/F1
  values to compare against a published reference table (e.g. Tables 8–9, S12–S14
  of LncCat).
when_to_use_negative:
- Data are unlabeled transcripts with no lncRNA/PCT ground truth — confusion-matrix-based
  metrics (MCC, F1, SEN, SPE) cannot be computed
- Multi-class or regression prediction tasks — MCC/F1 as applied here assume a binary
  lncRNA-vs-PCT classification with a specified positive class
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0099
tools:
- name: LncCat
  role: CatBoost-based classifier with codon-related, GC-related, sequence-related,
    peptide-related, and ORF-attention features; its scores at cutoff 0.5 are evaluated
    against the eight comparison methods
  repo: https://github.com/a525076133/LncCat
- name: CPAT
  role: Logistic regression comparison method (v3.0.0) whose predictions are scored
    with the same metric suite
  repo: http://cpat.readthedocs.io/en/latest
- name: CNCI
  role: SVM-based comparison method (version 2) evaluated on the same sORF datasets
  repo: http://www.bioinfo.org/software/cnci
- name: PLEK
  role: Improved k-mer SVM comparison method (v1.2) evaluated on the same sORF datasets
  repo: http://sourceforge.net/projects/plek
- name: CPC2
  role: Alignment-free sequence-feature comparison method (v1.0.1) evaluated on the
    same sORF datasets
  repo: http://cpc2.cbi.pku.edu.cn
- name: LncFinder
  role: SVM comparison method with secondary-structure and EIIP-derived features (v1.1.5)
    evaluated on the same sORF datasets
  repo: http://github.com/HAN-Siyu/LncFinder
- name: CPPred
  role: SVM comparison method evaluated on the same sORF datasets
  repo: http://www.rnabinding.com/CPPred
- name: mRNN
  role: Recursive neural network comparison method evaluated on the same sORF datasets
  repo: http://github.com/hendrixlab/mRNN
- name: RNAsamba
  role: Neural network comparison method evaluated on the same sORF datasets
  repo: http://github.com/apcamargo/RNAsamba
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.02.012
    title: 'LncCat: An ORF attention model to identify LncRNA based on ensemble learning
      strategy and fused sequence information'
  source_content_hash: sha256:cda312cde68a5c9ab7312b76de4a44e18fc0b84c66a82bb8904f2a1e3d124e28
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# mcc-f1-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute PRE, SEN, SPE, ACC, F1, MCC, and AUC for binary lncRNA/PCT classifiers on sORF test datasets, treating lncRNAs as the positive class, to quantify performance gaps between LncCat (CatBoost + ORF-attention features) and eight competing methods (CPAT, CNCI, PLEK, CPC2, LncFinder, CPPred, mRNN, RNAsamba).

## When to use

Use when benchmarking lncRNA identification methods (or any binary sequence classifiers) on the human/mouse/zebrafish sORF test datasets and you need MCC/F1 values to compare against a published reference table (e.g. Tables 8–9, S12–S14 of LncCat).

## When NOT to use

- Data are unlabeled transcripts with no lncRNA/PCT ground truth — confusion-matrix-based metrics (MCC, F1, SEN, SPE) cannot be computed
- Multi-class or regression prediction tasks — MCC/F1 as applied here assume a binary lncRNA-vs-PCT classification with a specified positive class

## Inputs

- FASTA transcript sequences for the three sORF test datasets (human-sORF: 639 lncRNAs / 641 PCTs; mouse-sORF: 993 lncRNAs / 846 PCTs; zebrafish-sORF: 497 lncRNAs / 387 PCTs, Ensembl-derived)
- Per-transcript prediction scores or labels from each classifier (LncCat, CPAT, CNCI, PLEK, CPC2, LncFinder, CPPred, mRNN, RNAsamba)
- Ground-truth class labels (lncRNA as positive, PCT as negative)

## Outputs

- Per-method, per-dataset metrics table with PRE, SEN, SPE, ACC, F1, MCC, and AUC (mirroring Tables 8, 9, and S12–S14)
- MCC/F1 comparison table validating LncCat's improvement margins (e.g. human-sORF MCC 0.8858, mouse-sORF MCC 0.7971, zebrafish-sORF MCC 0.8735)

## How to apply

Run each classifier (LncCat with prediction-score cutoff 0.5; CPAT v3.0.0, CNCI v2, PLEK v1.2, CPC2 v1.0.1, LncFinder v1.1.5, CPPred, mRNN, RNAsamba at their documented versions) on the three independent sORF test sets: human-sORF (639 lncRNAs / 641 PCTs), mouse-sORF (993 lncRNAs / 846 PCTs), zebrafish-sORF (497 lncRNAs / 387 PCTs). Convert prediction scores to class labels using the cutoff of 0.5 (score >= 0.5 = lncRNA, < 0.5 = PCT). With lncRNAs defined as the positive class, compute PRE, SEN, SPE, ACC, F1, MCC, and AUC from the confusion matrix for each method on each dataset. Assemble MCC and F1 into a results table mirroring Tables 8 and 9 and validate that LncCat's MCC exceeds the second-best method by at least 11.90% (human), 12.96% (mouse), and 42.61% (zebrafish).

## Related tools

- **LncCat** (CatBoost-based classifier with codon-related, GC-related, sequence-related, peptide-related, and ORF-attention features; its scores at cutoff 0.5 are evaluated against the eight comparison methods) — https://github.com/a525076133/LncCat
- **CPAT** (Logistic regression comparison method (v3.0.0) whose predictions are scored with the same metric suite) — http://cpat.readthedocs.io/en/latest
- **CNCI** (SVM-based comparison method (version 2) evaluated on the same sORF datasets) — http://www.bioinfo.org/software/cnci
- **PLEK** (Improved k-mer SVM comparison method (v1.2) evaluated on the same sORF datasets) — http://sourceforge.net/projects/plek
- **CPC2** (Alignment-free sequence-feature comparison method (v1.0.1) evaluated on the same sORF datasets) — http://cpc2.cbi.pku.edu.cn
- **LncFinder** (SVM comparison method with secondary-structure and EIIP-derived features (v1.1.5) evaluated on the same sORF datasets) — http://github.com/HAN-Siyu/LncFinder
- **CPPred** (SVM comparison method evaluated on the same sORF datasets) — http://www.rnabinding.com/CPPred
- **mRNN** (Recursive neural network comparison method evaluated on the same sORF datasets) — http://github.com/hendrixlab/mRNN
- **RNAsamba** (Neural network comparison method evaluated on the same sORF datasets) — http://github.com/apcamargo/RNAsamba

## Examples

```
python compute_metrics.py --pred lnccat_human_sORF_scores.tsv --truth human_sORF_labels.tsv --cutoff 0.5 --positive-class lncRNA --metrics PRE SEN SPE ACC F1 MCC AUC
```

## Evaluation signals

- LncCat human-sORF MCC should reproduce ~0.8858, mouse-sORF ~0.7971, zebrafish-sORF ~0.8735
- LncCat's MCC on each sORF dataset must exceed the second-best method by at least 11.90% (human), 12.96% (mouse), and 42.61% (zebrafish)
- Metrics must be computed with lncRNAs as the positive class; reversing the positive class will invert SEN/SPE and change F1
- Prediction scores must be binarized at the documented cutoff of 0.5 (>= 0.5 = lncRNA, < 0.5 = PCT) before computing MCC/F1
- F1 values should be consistent with the MCC ranking per dataset (LncCat ranked first on all three sORF sets)

## Limitations

- CatBoost hyperparameter tuning settings and CatBoost/BERT versions are not reported, limiting exact reproducibility of the LncCat scores feeding the metric computation
- No public code repository for the LncCat model itself is provided (only a web server), so reference predictions may need re-training from the deposit datasets
- Cross-species generalization is limited: human/mouse/zebrafish models perform poorly on wheat (MCC 0.67, 0.63, 0.66), so metric comparisons are only valid within same-species settings

## Evidence

- [readme] README variant of the improvement claim: [evidence span withheld — anchor status: external_doc]
