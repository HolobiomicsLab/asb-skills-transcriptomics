---
name: gradient-boosting-model-training-catboost
description: Use when a tabular feature matrix encoded from transcript sequences (e.g.
  Fickett score, GC1/GC2/GC3, k-mer/CTD, Mw/PI/Gravy, BERT ORF-attention vectors)
  must be fit with a gradient-boosting classifier for binary lncRNA vs PCT prediction.
when_to_use_negative:
- The input is raw transcript FASTA without precomputed features — ORF extraction
  (Orfipy, transl_table=1) and the five feature types must be computed before CatBoost
  can be trained
- ncRNAs shorter than 200 nt or datasets with >80% sequence similarity not yet reduced
  by CD-HIT — the model was built and validated only on filtered, >=200 nt datasets
- The task requires per-species hyperparameter tuning — the paper does not report
  CatBoost hyperparameter settings or software versions, so claims of tuned per-species
  optimization are unsupported
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0659
tools:
- name: LncCat
  role: End-to-end pipeline fusing the five feature types with CatBoost to identify
    lncRNAs; provides precomputed feature matrices and datasets
  repo: https://github.com/a525076133/LncCat
- name: CatBoost
  role: Gradient-boosting classifier that builds the lncRNA/PCT prediction model;
    supports GPU execution
  repo: https://github.com/a525076133/LncCat
- name: Orfipy
  role: Extracts ORFs using Standard Code transl_table=1 (start codons ATG/TTG/CTG)
    upstream of ORF-attention feature computation
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2023.02.012
    title: 'LncCat: An ORF attention model to identify LncRNA based on ensemble learning
      strategy and fused sequence information'
  source_content_hash: sha256:6adf10ba76a79bad615dbac34fed311824d5c2bf96893d58b2f94f1f79b28134
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
  - CatBoost
  - LncCat
  - Orfipy
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# gradient-boosting-model-training-catboost

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a CatBoost gradient-boosting decision-tree classifier (as instantiated in LncCat) on five fused transcript feature types — codon-related, GC-related, sequence-related, peptide-related, and ORF-attention features — to discriminate lncRNAs from protein-coding transcripts (PCTs), including sORF-containing transcripts where competing methods underperform.

## When to use

Use when a tabular feature matrix encoded from transcript sequences (e.g. Fickett score, GC1/GC2/GC3, k-mer/CTD, Mw/PI/Gravy, BERT ORF-attention vectors) must be fit with a gradient-boosting classifier for binary lncRNA vs PCT prediction.

## When NOT to use

- The input is raw transcript FASTA without precomputed features — ORF extraction (Orfipy, transl_table=1) and the five feature types must be computed before CatBoost can be trained
- ncRNAs shorter than 200 nt or datasets with >80% sequence similarity not yet reduced by CD-HIT — the model was built and validated only on filtered, >=200 nt datasets
- The task requires per-species hyperparameter tuning — the paper does not report CatBoost hyperparameter settings or software versions, so claims of tuned per-species optimization are unsupported

## Inputs

- Tabular training/validation/test feature matrices of transcript sequences (70/10/20 split), with five feature types: codon-related (stop codon count/frequency, Fickett score), GC-related (GC1/GC2/GC3 and frame scores), sequence-related (length, k-mer, CTD, hexamer score), peptide-related (Mw, PI, Gravy, instability index), and ORF-attention features
- Binary class labels: lncRNA (positive) vs PCT (negative), per transcript
- Transcript FASTA sequences from NCBI RefSeq, GENCODE, and Ensembl for feature extraction (CD-HIT filtered at 80%, length >= 200 nt)

## Outputs

- Trained CatBoost classification model (per-species or integrated multi-species)
- Predicted probabilities on test transcripts, thresholded at 0.5 into lncRNA/PCT calls
- Evaluation table of PRE, SEN, SPE, ACC, F1, MCC, AUC per species or per benchmark (e.g. lnccat_species_metrics.csv, integrated_model_evaluation.csv)

## How to apply

Encode each transcript with the five LncCat feature types, extracting ORFs with Orfipy using Standard Code (transl_table=1) with start codons ATG, TTG, CTG, and embedding ORF peptides with a pre-trained BERT model. Train the CatBoost classifier on the 70% training split, keeping 10% for validation and 20% for testing, with GPU execution supported by CatBoost for large merged multi-species matrices. At inference, apply the fixed cutoff of 0.5 — prediction scores >= 0.5 are labeled lncRNA and < 0.5 labeled PCT. Evaluate with seven metrics (ACC, SEN, SPE, F1, PRE, MCC, AUC) on held-out test sets, benchmarking against methods such as CPAT, CNCI, PLEK, CPC2, LncFinder, CPPred, mRNN, and RNAsamba. Optionally merge training data across human, mouse, zebrafish, wheat, and chicken to build an integrated model (expected integrated MCC 0.9236; sORF-integrated MCC 0.8346).

## Related tools

- **LncCat** (End-to-end pipeline fusing the five feature types with CatBoost to identify lncRNAs; provides precomputed feature matrices and datasets) — https://github.com/a525076133/LncCat
- **CatBoost** (Gradient-boosting classifier that builds the lncRNA/PCT prediction model; supports GPU execution) — https://github.com/a525076133/LncCat
- **Orfipy** (Extracts ORFs using Standard Code transl_table=1 (start codons ATG/TTG/CTG) upstream of ORF-attention feature computation)

## Examples

```
from catboost import CatBoostClassifier; model = CatBoostClassifier(task_type='GPU'); model.fit(X_train, y_train, eval_set=(X_val, y_val)); preds = model.predict_proba(X_test)[:,1] >= 0.5
```

## Evaluation signals

- Per-species test MCC should approach the reported values: 0.9503 (human), 0.9219 (mouse), 0.8591 (zebrafish), 0.8672 (wheat), 0.9047 (chicken); human F1 of 0.9742
- Integrated five-species model should reach MCC 0.9236 (F1 0.9537, Table 14) and the sORF-integrated model MCC 0.8346 (F1 0.9245, Table 15)
- MCC improvements over the eight competing methods should fall in the reported ranges (e.g. 1.90%–7.82% on human, up to 51.64% on wheat)
- Prediction output must be binarized with the fixed cutoff of 0.5 — scores >= 0.5 as lncRNA, < 0.5 as PCT
- UMAP visualization of features should show lncRNAs and PCTs as separable distributions, particularly in ORF-attention feature space

## Limitations

- CatBoost hyperparameter tuning settings are not reported, and no version of CatBoost or BERT is specified, hindering exact reproduction
- Cross-species models transfer poorly — human, mouse, and zebrafish models achieve only ~0.63–0.67 MCC on wheat
- Species may lack annotations, making it impossible to build models for all species

## Evidence
