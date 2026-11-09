---
name: peptide-physicochemical-feature-calculation
description: Use when encoding transcript sequences for coding-potential prediction
  and putative peptide sequences translated from ORFs need to be converted into peptide-related
  numerical features (MW, pI, Gravy, instability index) for a classifier.
when_to_use_negative:
- Input consists of raw nucleotide transcripts without any ORF/peptide information
  (e.g. the study notes future work requires collecting sORF-translated peptide sequences
  that are not yet available)
- Input is already a numeric feature table — re-computing peptide descriptors from
  scratch would be redundant
- Non-Standard genetic-code organisms, since Orfipy is configured with Standard Code
  transl_table=1 (ATG/TTG/CTG starts, TAA/TAG/TGA stops)
edam_operation: http://edamontology.org/operation_0253
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Orfipy
  role: Extracts ORFs (top three longest per transcript) with Standard Code transl_table=1
    whose translation yields the putative peptide sequences
- name: BERT
  role: Pre-trained model providing contextual vector representations of the ORF peptide
    sequences used as ORF-attention features
  repo: https://github.com/a525076133/LncCat
- name: LncCat
  role: Pipeline that fuses the peptide physicochemical features with codon-, GC-,
    sequence- and ORF-attention features into the 599-dim vector
  repo: https://github.com/a525076133/LncCat
- name: CatBoost
  role: Downstream classifier that consumes the fused feature vector to distinguish
    lncRNAs from PCTs
  repo: https://github.com/a525076133/LncCat
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.02.012
    title: 'LncCat: An ORF attention model to identify LncRNA based on ensemble learning
      strategy and fused sequence information'
  source_content_hash: sha256:1527c649e2191ccc0b79a6c1a5eace9acf07d6c3626c00b6748fbda3e193e6b1
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
  - BERT
  - CatBoost
  - LncCat
  - Orfipy
  edam_operation: http://edamontology.org/operation_0253
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# peptide-physicochemical-feature-calculation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes peptide-related physicochemical features (molecular weight, theoretical isoelectric point, Gravy hydrophilicity, instability index) from ORF-derived putative peptide sequences, which LncCat fuses with codon-, GC-, sequence- and ORF-attention features into the 599-dimensional vector fed to CatBoost for lncRNA vs. PCT classification.

## When to use

Use when encoding transcript sequences for coding-potential prediction and putative peptide sequences translated from ORFs need to be converted into peptide-related numerical features (MW, pI, Gravy, instability index) for a classifier.

## When NOT to use

- Input consists of raw nucleotide transcripts without any ORF/peptide information (e.g. the study notes future work requires collecting sORF-translated peptide sequences that are not yet available)
- Input is already a numeric feature table — re-computing peptide descriptors from scratch would be redundant
- Non-Standard genetic-code organisms, since Orfipy is configured with Standard Code transl_table=1 (ATG/TTG/CTG starts, TAA/TAG/TGA stops)

## Inputs

- Transcript FASTA sequences (CD-HIT reduced at 80% identity) from NCBI RefSeq, Ensembl, or GENCODE
- Putative peptide sequences translated from the longest ORF of each transcript (Orfipy, transl_table=1)

## Outputs

- Per-transcript peptide-related feature vector (molecular weight, theoretical isoelectric point, Gravy hydrophilicity, instability index)
- BERT contextual vector representations of ORF peptide sequences (ORF-attention features)
- Merged 599-dimensional feature matrix with transcript identifiers and class labels written to orf_attention_features.csv (lncRNA positive, PCT negative)

## How to apply

First extract ORFs from each CD-HIT-reduced (80% identity) transcript with Orfipy using Standard Code (transl_table=1), recognizing ATG, TTG, CTG as start codons and TAA, TAG, TGA as stop codons, and translate the longest ORF into its putative peptide sequence. Compute four peptide-related descriptors per peptide: molecular weight, theoretical isoelectric point, Gravy hydrophilicity, and instability index. In parallel, encode the peptide with a pre-trained BERT model whose contextual vector representations capture the ORF-derived sequence information, contributing to the ORF-attention features. Concatenate these peptide features with the whole-transcript features (stop codon count/frequency, Fickett TESTCODE score, GC content and GC1/GC2/GC3 frame scores, length/k-mer/CTD/hexamer usage) and ORF-dominance, ORF length and ORF coverage terms into a single 599-dimensional vector per transcript, then write orf_attention_features.csv for the CatBoost classifier.

## Related tools

- **Orfipy** (Extracts ORFs (top three longest per transcript) with Standard Code transl_table=1 whose translation yields the putative peptide sequences)
- **BERT** (Pre-trained model providing contextual vector representations of the ORF peptide sequences used as ORF-attention features) — https://github.com/a525076133/LncCat
- **LncCat** (Pipeline that fuses the peptide physicochemical features with codon-, GC-, sequence- and ORF-attention features into the 599-dim vector) — https://github.com/a525076133/LncCat
- **CatBoost** (Downstream classifier that consumes the fused feature vector to distinguish lncRNAs from PCTs) — https://github.com/a525076133/LncCat

## Evaluation signals

- The peptide descriptors computed for each transcript are exactly molecular weight, theoretical isoelectric point, Gravy hydrophilicity, and instability index, matching the paper's peptide-related feature list
- The peptide input is the translation of the longest ORF, reducing sequence length to roughly one-third of the transcript
- The merged feature matrix has one row per input transcript and a 599-dimensional feature layout consistent with Supplementary Table S1-1
- Feeding the encoded features to CatBoost reproduces reported performance, e.g. MCC of 0.9503 on the human test dataset
- UMAP visualization of the features should show lncRNAs and PCTs as distinguishable distributions in ORF-attention feature space

## Limitations

- sORF-translated peptide sequences for large-scale functional annotation are not available in this study and are deferred to future work
- Cross-species transfer degrades: human, mouse, and zebrafish models perform poorly on wheat (MCC 0.67, 0.63, 0.66)
- No version of BERT is specified, limiting reproducibility of the peptide contextual encodings

## Evidence
