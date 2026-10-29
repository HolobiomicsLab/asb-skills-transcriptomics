---
name: orf-feature-extraction
description: Use when encoding transcript sequences (e.g. lncRNA/PCT FASTA filtered
  to ≥200 nt) into numeric feature vectors where ORF content, not just global sequence
  composition, discriminates coding potential.
when_to_use_negative:
- Input sequences are shorter than 200 nt or the goal is prediction without sequence
  features (e.g. classification of already-annotated transcripts)
- Input is already a computed feature table — re-running ORF extraction is unnecessary
- The species transcriptome lacks annotations/ORF signal, since ORF features depend
  on detectable ORFs
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3474
tools:
- name: Orfipy
  role: Extracts ORFs from transcripts using Standard Code (transl_table=1) with start
    codons ATG/TTG/CTG and stop codons TAA/TAG/TGA
- name: BERT
  role: Pre-trained model that encodes the longest-ORF peptide sequences into contextual
    ORF-attention feature vectors
- name: LncCat
  role: Prediction framework that combines the extracted five feature types and uses
    CatBoost to identify lncRNAs
  repo: https://github.com/a525076133/LncCat
- name: CD-HIT
  role: Reduces dataset redundancy at an 80% similarity threshold before feature extraction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.02.012
    title: 'LncCat: An ORF attention model to identify LncRNA based on ensemble learning
      strategy and fused sequence information'
  source_content_hash: sha256:d41fb9599321af748f1b0bb5aa220da82612eb5b7bde7842de901e2f27aa6356
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
  - CD-HIT
  - LncCat
  - Orfipy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# orf-feature-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract ORF-derived features (codon-related, GC-related, sequence-related, peptide-related, and ORF-attention features) from transcript sequences to encode lncRNAs vs protein-coding transcripts, as used by LncCat. ORF-attention features are obtained by encoding the longest-ORF peptide with a pre-trained BERT model after ORF extraction with Orfipy.

## When to use

Use when encoding transcript sequences (e.g. lncRNA/PCT FASTA filtered to ≥200 nt) into numeric feature vectors where ORF content, not just global sequence composition, discriminates coding potential.

## When NOT to use

- Input sequences are shorter than 200 nt or the goal is prediction without sequence features (e.g. classification of already-annotated transcripts)
- Input is already a computed feature table — re-running ORF extraction is unnecessary
- The species transcriptome lacks annotations/ORF signal, since ORF features depend on detectable ORFs

## Inputs

- Deduplicated transcript FASTA datasets (lncRNAs and PCTs from NCBI RefSeq, Ensembl, GENCODE; CD-HIT 80%, length ≥200 nt)
- Pre-trained BERT model for peptide sequence vector representation

## Outputs

- Per-transcript ORF-attention feature vectors (BERT embeddings of longest-ORF peptides)
- Combined feature matrices of codon-related, GC-related, sequence-related, peptide-related, and ORF-attention features
- Contrast base feature matrix lacking ORF-attention features

## How to apply

Start from deduplicated transcript datasets (CD-HIT at 80% threshold, transcripts ≥200 nt) from NCBI RefSeq, Ensembl, or GENCODE. Extract ORFs from all transcripts with Orfipy using the Standard Code (transl_table=1) with start codons ATG/TTG/CTG and stop codons TAA/TAG/TGA under default parameters. For the top three longest ORFs, compute codon-related, GC-related, sequence-related, and peptide-related features; additionally translate the longest ORF and encode its peptide sequence with a pre-trained BERT model to obtain contextual ORF-attention vectors. Concatenate all feature types to form the full transcript encoding, or omit the ORF-attention block to construct the base (without-ORF-attention) contrast feature set. The rationale is that ORF-attention features capture coding-potential signal that discriminates lncRNAs from PCTs even when sORFs are present.

## Related tools

- **Orfipy** (Extracts ORFs from transcripts using Standard Code (transl_table=1) with start codons ATG/TTG/CTG and stop codons TAA/TAG/TGA)
- **BERT** (Pre-trained model that encodes the longest-ORF peptide sequences into contextual ORF-attention feature vectors)
- **LncCat** (Prediction framework that combines the extracted five feature types and uses CatBoost to identify lncRNAs) — https://github.com/a525076133/LncCat
- **CD-HIT** (Reduces dataset redundancy at an 80% similarity threshold before feature extraction)

## Examples

```
orfipy transcripts.fa --procs 8 --table 1 --orffile orfs.tsv --peptide --min 30 --start ATG,TTG,CTG --stop TAA,TAG,TGA
```

## Evaluation signals

- UMAP 2D embedding of ORF-attention features should separate lncRNAs and PCTs into two adequately divided clusters, whereas the base (without-ORF-attention) features show poor separation
- ORF extraction parameters must match transl_table=1 with start codons ATG/TTG/CTG and stop codons TAA/TAG/TGA
- Features are computed for the top three longest ORFs per transcript, not just the single longest
- Downstream CatBoost models using the fused features should reach MCC values near 0.9503/0.9219/0.8591 on human/mouse/zebrafish datasets
- Transcript inputs must already be deduplicated with CD-HIT at 80% and filtered to ≥200 nt before feature computation

## Limitations

- No version of CatBoost or BERT is specified for reproducibility
- Species may lack annotations, making it impossible to build ORF-based models for all species
- The LncCat repository only exposes a web server; fine-grained feature-extraction code may not be directly reusable (results returned via email)

## Evidence
