---
name: orf-extraction-with-orfipy
description: Use when transcript sequences (e.g. CD-HIT-reduced lncRNA/PCT FASTA from
  RefSeq, Ensembl, or GENCODE) must be scanned for ORFs to compute ORF-derived coding-potential
  features or ORF-attention encodings.
when_to_use_negative:
- Input is already a feature table or precomputed coding-potential scores — ORF extraction
  would be redundant
- Sequences are raw genomic DNA rather than mature transcript sequences, since ORF
  coverage and dominance are defined relative to transcript length
- Datasets are not redundancy-reduced (CD-HIT 80%) or contain ncRNAs shorter than
  200 nt, which LncCat's pipeline explicitly filters out
edam_operation: http://edamontology.org/operation_2451
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0659
tools:
- name: Orfipy
  role: Extracts ORFs from transcript sequences under Standard Code (transl_table=1)
    to supply ORF-related and ORF-attention features
  repo: https://github.com/a525076133/LncCat
- name: BERT
  role: Encodes the longest-ORF translated peptide sequences into contextual vector
    representations (ORF-attention features)
  repo: https://github.com/a525076133/LncCat
- name: LncCat
  role: Downstream CatBoost-based classifier consuming the ORF-derived features to
    distinguish lncRNAs from PCTs
  repo: https://github.com/a525076133/LncCat
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.02.012
    title: 'LncCat: An ORF attention model to identify LncRNA based on ensemble learning
      strategy and fused sequence information'
  source_content_hash: sha256:a6de22c171cf43fe45eb2ede2c9261e92c8a9a6b33eb6b05dcd6985084d3684c
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
  - LncCat
  - Orfipy
  edam_operation: http://edamontology.org/operation_2451
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# ORF extraction with Orfipy

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts open reading frames (ORFs) from transcript FASTA sequences using Orfipy under the Standard Code (transl_table=1), retaining the top three longest ORFs per transcript as the basis for LncCat's ORF-related and ORF-attention features.

## When to use

Use when transcript sequences (e.g. CD-HIT-reduced lncRNA/PCT FASTA from RefSeq, Ensembl, or GENCODE) must be scanned for ORFs to compute ORF-derived coding-potential features or ORF-attention encodings.

## When NOT to use

- Input is already a feature table or precomputed coding-potential scores — ORF extraction would be redundant
- Sequences are raw genomic DNA rather than mature transcript sequences, since ORF coverage and dominance are defined relative to transcript length
- Datasets are not redundancy-reduced (CD-HIT 80%) or contain ncRNAs shorter than 200 nt, which LncCat's pipeline explicitly filters out

## Inputs

- CD-HIT-reduced transcript FASTA sequences (80% identity threshold) for lncRNAs and PCTs from NCBI RefSeq, Ensembl, and GENCODE (human, mouse, zebrafish, wheat, chicken)
- Orfipy default parameters with Standard Code (transl_table=1) settings

## Outputs

- Per-transcript ORF coordinates and sequences (top three longest ORFs per transcript)
- Longest-ORF putative peptide sequences (one-third length of the ORF nucleotide sequence) for BERT encoding
- ORF length, ORF coverage, and ORF-dominance values feeding the 599-dimensional orf_attention_features.csv matrix

## How to apply

Run Orfipy on the CD-HIT-reduced (80% identity threshold) transcript FASTA using default parameters with the Standard Code (transl_table=1), which recognizes ATG, TTG, and CTG as start codons and TAA, TAG, and TGA as stop codons. Retain the top three longest ORFs per transcript, then compute codon-related, GC-related, sequence-related, and peptide-related features on both the full transcript and the selected ORFs. Translate the longest ORF to its putative peptide (roughly one-third of the nucleotide length) and encode it with a pre-trained BERT model to obtain the ORF-attention vectors, alongside ORF length, ORF coverage (longest ORF length / transcript length), and the ORF-dominance feature. The rationale is that ORF-level signals — especially for transcripts with small ORFs (sORFs) — are what let LncCat outperform alignment-free competitors like CPAT, CNCI, PLEK, CPC2, LncFinder, CPPred, mRNN, and RNAsamba.

## Related tools

- **Orfipy** (Extracts ORFs from transcript sequences under Standard Code (transl_table=1) to supply ORF-related and ORF-attention features) — https://github.com/a525076133/LncCat
- **BERT** (Encodes the longest-ORF translated peptide sequences into contextual vector representations (ORF-attention features)) — https://github.com/a525076133/LncCat
- **LncCat** (Downstream CatBoost-based classifier consuming the ORF-derived features to distinguish lncRNAs from PCTs) — https://github.com/a525076133/LncCat

## Examples

```
orfipy transcripts.fa --pep --orfflags -n 3 --table 1 > orfs.tsv
```

## Evaluation signals

- Every input transcript yields ORF output (one row per transcript in the downstream feature matrix)
- Retained ORF set matches the top three longest ORFs per transcript under transl_table=1 (ATG/TTG/CTG starts; TAA/TAG/TGA stops)
- ORF coverage (longest ORF length / transcript length) falls in [0, 1] and translated peptide length is approximately one-third of the longest ORF nucleotide length
- Concatenated whole-transcript plus ORF-attention features form the 599-dimensional vector consistent with Supplementary Table S1-1
- Downstream CatBoost classifier reaches the reported performance, e.g. MCC of 0.9503 on the human test dataset

## Limitations

- Standard Code start/stop codon definitions may miss ORFs using non-standard genetic codes; LncCat applied transl_table=1 only
- Only the top three longest ORFs per transcript are retained, so shorter sORFs outside the top three are not directly encoded
- No BERT or CatBoost versions are specified, complicating exact reproducibility; species lacking annotations cannot be modeled for all species

## Evidence
