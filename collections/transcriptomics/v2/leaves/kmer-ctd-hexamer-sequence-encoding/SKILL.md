---
name: kmer-ctd-hexamer-sequence-encoding
description: Use when raw transcript FASTA sequences (lncRNAs and PCTs) must be encoded
  into sequence-intrinsic numeric features (k-mer, CTD, hexamer scores) to feed a
  classifier such as CatBoost, especially alongside ORF-derived features from Orfipy.
when_to_use_negative:
- Input transcripts have not been redundancy-reduced (no CD-HIT at 80%) or include
  ncRNAs shorter than 200 nt — the article's pipeline eliminates these before feature
  computation
- A pre-computed feature matrix (e.g., existing orf_attention_features.csv) is already
  available and only the CatBoost classifier needs to be retrained
- The goal is purely ORF-attention or peptide BERT encoding rather than sequence-intrinsic
  k-mer/CTD/hexamer features
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3474
tools:
- name: LncCat
  role: Framework that fuses the k-mer/CTD/hexamer sequence-related features with
    codon-, GC-, peptide-, and ORF-attention features into the 599-dim vector
  repo: https://github.com/a525076133/LncCat
- name: Orfipy
  role: Extracts ORFs (Standard Code transl_table=1, top three longest per transcript)
    whose sequences are also encoded with k-mer/CTD/hexamer features
- name: CPAT
  role: Prior method whose Fickett TESTCODE score and hexamer score of ORF regions
    motivated the hexamer/Fickett features adopted by LncCat
  repo: http://cpat.readthedocs.io/en/latest
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.02.012
    title: 'LncCat: An ORF attention model to identify LncRNA based on ensemble learning
      strategy and fused sequence information'
  source_content_hash: sha256:ac7475df7a24671fe647bbc072268f48d0544c587968af1c4c9248c91037f655
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
  - CPAT
  - LncCat
  - Orfipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# kmer-ctd-hexamer-sequence-encoding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes sequence-related features — k-mer frequencies, CTD composition/transition/distribution descriptors, and hexamer usage bias with hexamer score — from transcript sequences and their ORFs, as one of the feature families LncCat fuses into a 599-dimensional vector for CatBoost-based lncRNA vs. protein-coding transcript classification.

## When to use

Use when raw transcript FASTA sequences (lncRNAs and PCTs) must be encoded into sequence-intrinsic numeric features (k-mer, CTD, hexamer scores) to feed a classifier such as CatBoost, especially alongside ORF-derived features from Orfipy.

## When NOT to use

- Input transcripts have not been redundancy-reduced (no CD-HIT at 80%) or include ncRNAs shorter than 200 nt — the article's pipeline eliminates these before feature computation
- A pre-computed feature matrix (e.g., existing orf_attention_features.csv) is already available and only the CatBoost classifier needs to be retrained
- The goal is purely ORF-attention or peptide BERT encoding rather than sequence-intrinsic k-mer/CTD/hexamer features

## Inputs

- CD-HIT-reduced (80% threshold) transcript FASTA files for human, mouse, zebrafish, wheat, and chicken datasets
- ORF DNA/peptide sequences extracted by Orfipy with transl_table=1 (top three longest ORFs per transcript)
- Class labels (lncRNA as positive, PCT as negative) per transcript

## Outputs

- orf_attention_features.csv — merged feature matrix with transcript identifiers, class labels, and a 599-dimensional feature vector per transcript
- Per-transcript sequence-related feature values (k-mer, CTD composition/transition/distribution, hexamer usage bias and score)

## How to apply

Start from CD-HIT-reduced transcript FASTA files (80% identity threshold) with ncRNAs shorter than 200 nt removed, from sources such as NCBI RefSeq, Ensembl, and GENCODE. Extract ORFs for every transcript with Orfipy using Standard Code (transl_table=1, start codons ATG/TTG/CTG, stop codons TAA/TAG/TGA) and retain the top three longest ORFs per transcript. Compute sequence-related features — length, k-mer, CTD composition/transition/distribution, hexamer usage bias and hexamer score — for both the full transcript and the selected ORFs, in parallel with codon-related (Fickett TESTCODE score, stop codon count/frequency), GC-related (GC content, GC1/GC2/GC3, GC frame), peptide-related, and ORF-attention (BERT-encoded peptide vectors, ORF length, ORF coverage) features. Concatenate all feature families into a single 599-dimensional vector per transcript and write the matrix with transcript identifiers and class labels (lncRNA positive, PCT negative) to orf_attention_features.csv. Train the CatBoost classifier on a 70/10/20 train/validation/test split and apply a 0.5 cutoff (score ≥ 0.5 → lncRNA). The rationale is that coding potential is reflected in hexamer usage and sequence composition patterns, which complement ORF-attention signals for detecting transcripts with sORFs.

## Related tools

- **LncCat** (Framework that fuses the k-mer/CTD/hexamer sequence-related features with codon-, GC-, peptide-, and ORF-attention features into the 599-dim vector) — https://github.com/a525076133/LncCat
- **Orfipy** (Extracts ORFs (Standard Code transl_table=1, top three longest per transcript) whose sequences are also encoded with k-mer/CTD/hexamer features)
- **CPAT** (Prior method whose Fickett TESTCODE score and hexamer score of ORF regions motivated the hexamer/Fickett features adopted by LncCat) — http://cpat.readthedocs.io/en/latest

## Examples

```
orfipy transcripts.fa --table 1 --dna orfs_dna.fa --pep peptides.fa
```

## Evaluation signals

- Output matrix has exactly one row per input transcript with a 599-dimensional feature layout consistent with Supplementary Table S1-1
- Each feature vector includes k-mer, CTD composition/transition/distribution, and hexamer usage bias/score components alongside ORF coverage (longest ORF length / transcript length)
- Feeding the encoded features to CatBoost reproduces the reported performance, e.g., MCC 0.9503 and F1 0.9742 on the human test dataset
- UMAP projection of the feature vectors shows lncRNAs and PCTs separating distinctly in ORF-attention feature space
- Predictions use the 0.5 cutoff (score ≥ 0.5 → lncRNA, < 0.5 → PCT)

## Limitations

- Human, mouse, and zebrafish models perform poorly on wheat (MCC ~0.67, 0.63, 0.66), indicating sequence-derived features are species-sensitive
- The article does not specify CatBoost or BERT versions, limiting exact reproducibility of the feature-to-classifier pipeline
- Existing methods with sequence/k-mer features perform poorly on sORF datasets, so these features alone are insufficient without ORF-attention components

## Evidence
