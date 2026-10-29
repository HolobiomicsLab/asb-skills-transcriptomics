---
name: codon-usage-feature-computation
description: Use when encoding transcript FASTA sequences into fixed-length numeric
  features for coding-potential prediction, specifically when codon usage bias, stop-codon
  statistics, or GC content/position features are needed as classifier inputs.
when_to_use_negative:
- Input is already a numeric feature matrix or the downstream classifier expects raw
  sequences (e.g., deep models like mRNN that one-hot encode) rather than engineered
  codon/GC features
- ncRNA candidates shorter than 200 nt, which the LncCat pipeline explicitly removes
  before feature extraction
- Non-standard genetic code organisms where Standard Code (transl_table=1) start/stop
  codon assumptions are invalid
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3512
tools:
- name: Orfipy
  role: Extracts ORFs from transcript sequences under Standard Code (transl_table=1)
    so codon-related and GC-related features can be computed on ORFs as well as full
    transcripts
- name: LncFinder
  role: Reference implementation providing sequence intrinsic features (including
    GC content, Fickett TESTCODE score, hexamer score) reused for codon/GC feature
    computation in LncCat
  repo: http://github.com/HAN-Siyu/LncFinder
- name: CPAT
  role: Baseline tool illustrating the use of the Fickett TESTCODE score and hexamer
    score of ORF regions within a logistic regression model
  repo: http://cpat.readthedocs.io/en/latest
- name: LncCat
  role: Downstream pipeline that concatenates the codon/GC features with ORF-attention
    features into a 599-dim vector for CatBoost classification
  repo: https://github.com/a525076133/LncCat
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.02.012
    title: 'LncCat: An ORF attention model to identify LncRNA based on ensemble learning
      strategy and fused sequence information'
  source_content_hash: sha256:bd9944873830ca26dda19adf8b146784ee6bc9cf752b13143d4ee8dae84da042
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/a525076133/LncCat
  tools:
  - CPAT
  - LncCat
  - LncFinder
  - Orfipy
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# codon-usage-feature-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes codon-related and GC-related numerical features from transcript and ORF sequences — stop codon counts/frequencies, Fickett TESTCODE score, GC content, positional GC1/GC2/GC3 and GC frame scores — as part of the LncCat 599-dimensional feature vector used to distinguish lncRNAs from protein-coding transcripts.

## When to use

Use when encoding transcript FASTA sequences into fixed-length numeric features for coding-potential prediction, specifically when codon usage bias, stop-codon statistics, or GC content/position features are needed as classifier inputs.

## When NOT to use

- Input is already a numeric feature matrix or the downstream classifier expects raw sequences (e.g., deep models like mRNN that one-hot encode) rather than engineered codon/GC features
- ncRNA candidates shorter than 200 nt, which the LncCat pipeline explicitly removes before feature extraction
- Non-standard genetic code organisms where Standard Code (transl_table=1) start/stop codon assumptions are invalid

## Inputs

- CD-HIT-reduced (80% identity threshold) transcript FASTA sequences from NCBI RefSeq, Ensembl, and GENCODE
- Per-transcript ORF sequences extracted with Orfipy (top three longest, Standard Code transl_table=1)
- Class labels (lncRNA as positive, PCT as negative) per transcript

## Outputs

- Codon-related feature columns (stop codon count, stop codon frequency, Fickett TESTCODE score) per transcript and ORF
- GC-related feature columns (GC content, GC1/GC2/GC3, GC frame scores) per transcript and ORF
- Merged 599-dimensional feature matrix with transcript identifiers and labels written to orf_attention_features.csv

## How to apply

First extract ORFs from every transcript with Orfipy using Standard Code (transl_table=1), which recognizes start codons ATG, TTG, CTG and stop codons TAA, TAG, TGA, and retain the top three longest ORFs per transcript. Compute codon-related features — stop codon count and frequency and the Fickett TESTCODE score — plus GC-related features — overall GC content and positional GC1/GC2/GC3 and GC frame scores — for both the full transcript and the selected ORFs. Concatenate these codon/GC features with sequence-related, peptide-related, and ORF-attention (BERT-encoded peptide) features into a single 599-dimensional vector per transcript. The rationale is that stop codon statistics and GC composition discriminate lncRNAs from PCTs, and augmenting whole-transcript features with ORF-level features improves identification of transcripts with small ORFs. Write the merged matrix with transcript IDs and class labels (lncRNA positive, PCT negative) to orf_attention_features.csv and feed it to a CatBoost classifier.

## Related tools

- **Orfipy** (Extracts ORFs from transcript sequences under Standard Code (transl_table=1) so codon-related and GC-related features can be computed on ORFs as well as full transcripts)
- **LncFinder** (Reference implementation providing sequence intrinsic features (including GC content, Fickett TESTCODE score, hexamer score) reused for codon/GC feature computation in LncCat) — http://github.com/HAN-Siyu/LncFinder
- **CPAT** (Baseline tool illustrating the use of the Fickett TESTCODE score and hexamer score of ORF regions within a logistic regression model) — http://cpat.readthedocs.io/en/latest
- **LncCat** (Downstream pipeline that concatenates the codon/GC features with ORF-attention features into a 599-dim vector for CatBoost classification) — https://github.com/a525076133/LncCat

## Examples

```
orfipy transcripts.fa --pep orfs_pep.fa --table 1 --min-aa 1 > orfs.bed  # then compute Fickett TESTCODE, GC, GC1/GC2/GC3 features for transcript and ORF sequences and write orf_attention_features.csv
```

## Evaluation signals

- Verify the output feature matrix has exactly one row per input transcript and the 599-dimensional layout consistent with Supplementary Table S1-1
- Check stop codon frequency values fall in [0, 1] and GC content in [0, 1] for every transcript and ORF
- Confirm the encoded features support the reported performance, e.g., CatBoost classifier reaching LncCat MCC of 0.9503 on the human test dataset
- Validate ORF extraction used transl_table=1 so start codons ATG/TTG/CTG and stop codons TAA/TAG/TGA were treated per the Standard Code
- Cross-check that computed Fickett TESTCODE and hexamer features are present, since CPAT-style features rely on these

## Limitations

- CatBoost and BERT versions are not specified, hindering exact reproducibility of the feature-to-prediction pipeline
- Cross-species transfer is limited: human, mouse, and zebrafish models perform well on each other's datasets but poorly on wheat, with MCC of 0.67, 0.63, and 0.66
- The distinguishing signal depends on ORF extraction settings; transcripts lacking canonical ORFs may yield less informative codon/GC features

## Evidence
