---
name: bert-peptide-sequence-embedding
description: Use when transcript-derived peptide sequences (e.g., translated longest
  ORFs from Orfipy with transl_table=1) must be converted into fixed-size contextual
  vectors to augment ORF-attention features for lncRNA vs PCT classification.
when_to_use_negative:
- You only need raw nucleotide-level features (GC content, k-mer, Fickett TESTCODE)
  without ORF-attention enhancement — the base feature set suffices for the contrast
  condition
- Transcripts lack usable ORFs or annotations (the paper notes species may lack annotations,
  making model building impossible for all species)
- The goal is de novo ORF prediction or peptide functional annotation — this skill
  only encodes already-extracted ORF peptides, not sORF-translated peptide function
  (listed as future work)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BERT
  role: Pre-trained language model that produces contextual vector representations
    of translated longest-ORF peptide sequences, forming the ORF-attention feature
    block
- name: Orfipy
  role: Extracts ORFs with Standard Code (transl_table=1, start ATG/TTG/CTG, stop
    TAA/TAG/TGA) whose longest member is translated to the peptide input
- name: LncCat
  role: Downstream pipeline that concatenates the BERT-based ORF-attention features
    with four other feature types into the 599-d vector for CatBoost classification
  repo: https://github.com/a525076133/LncCat
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.02.012
    title: 'LncCat: An ORF attention model to identify LncRNA based on ensemble learning
      strategy and fused sequence information'
  source_content_hash: sha256:3526a25bed2a8e1d9830b3775848ea7e5d2aeff07b40db3982bcc4269ad918dd
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# bert-peptide-sequence-embedding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes the translated peptide of each transcript's longest ORF with a pre-trained BERT model to obtain contextual vector representations that become the ORF-attention features in LncCat's fused 599-dimensional encoding, enhancing ORF information beyond hand-crafted codon/GC/sequence/peptide features.

## When to use

Use when transcript-derived peptide sequences (e.g., translated longest ORFs from Orfipy with transl_table=1) must be converted into fixed-size contextual vectors to augment ORF-attention features for lncRNA vs PCT classification.

## When NOT to use

- You only need raw nucleotide-level features (GC content, k-mer, Fickett TESTCODE) without ORF-attention enhancement — the base feature set suffices for the contrast condition
- Transcripts lack usable ORFs or annotations (the paper notes species may lack annotations, making model building impossible for all species)
- The goal is de novo ORF prediction or peptide functional annotation — this skill only encodes already-extracted ORF peptides, not sORF-translated peptide function (listed as future work)

## Inputs

- Transcript FASTA sequences (five species: human, mouse, zebrafish, wheat, chicken) deduplicated with CD-HIT at 80% and filtered to >=200 nt
- ORF coordinates from Orfipy (Standard Code, transl_table=1), top three longest ORFs per transcript
- Translated longest-ORF peptide sequences (one-third the transcript length)

## Outputs

- Contextual BERT vector representation per ORF-derived peptide sequence
- ORF-attention feature block contributing to a 599-dimensional fused feature vector per transcript
- orf_attention_features.csv with transcript identifiers and class labels (lncRNA positive, PCT negative)

## How to apply

First extract ORFs from CD-HIT-reduced (80% identity) transcript FASTA sequences using Orfipy with the Standard Code (transl_table=1, start codons ATG/TTG/CTG, stop codons TAA/TAG/TGA) and retain the top three longest ORFs per transcript. Translate the longest ORF to its putative peptide (sequence length reduced to one-third) and feed each peptide through a pre-trained BERT model, which obtains vector representations of the sequence via contextual information. Concatenate the resulting embeddings with the ORF-dominance, ORF length, and ORF coverage (longest-ORF-length/transcript-length) features, and combine with codon-related, GC-related, sequence-related, and peptide-related features into a single 599-dimensional vector per transcript. The rationale is that contextual peptide embeddings capture ORF information that static features (Fickett TESTCODE, hexamer score, CTD) miss, which is critical for small-ORF transcripts. Retain transcript identifiers and class labels (lncRNA positive, PCT negative) when writing the merged matrix (orf_attention_features.csv) for downstream CatBoost training.

## Related tools

- **BERT** (Pre-trained language model that produces contextual vector representations of translated longest-ORF peptide sequences, forming the ORF-attention feature block)
- **Orfipy** (Extracts ORFs with Standard Code (transl_table=1, start ATG/TTG/CTG, stop TAA/TAG/TGA) whose longest member is translated to the peptide input)
- **LncCat** (Downstream pipeline that concatenates the BERT-based ORF-attention features with four other feature types into the 599-d vector for CatBoost classification) — https://github.com/a525076133/LncCat

## Evaluation signals

- The merged feature matrix must contain one row per input transcript with the 599-dimensional layout matching Supplementary Table S1-1
- UMAP of with-ORF-attention features must separate lncRNAs and PCTs into two clusters, whereas the without-ORF-attention base features show poor separation (Fig. 2-(1)s vs Fig. 2-(2)s)
- CatBoost fed with the embedded features should reproduce the reported MCC (e.g., 0.9503 human, 0.9219 mouse) on the 70/10/20 train/validation/test split
- On sORF test datasets (human 639/641, mouse 993/846, zebrafish 497/387 lncRNA/PCT), MCC should improve by at least ~6.88%, ~22.98%, ~39.43% relative to competitors per the README
- Each encoded peptide should be exactly one-third the length of its source longest ORF transcript region (nucleotide-to-amino-acid translation check)

## Limitations

- No version of CatBoost or BERT is specified, limiting reproducibility of the embedding
- Species may lack annotations, and it is impossible to build models for all species
- sORF-translated peptide sequence collection and function annotation are not yet available and are deferred to future work

## Evidence

- [methods] Pre-trained BERT contextual peptide embeddings: "The pre-trained BERT model can get the vector representation of the sequence by contextual information"
- [methods] Five fused feature types including ORF-attention: "five types of features are calculated from the sequences, including codon-related, GC-related, sequence-related, peptide-related features, and ORF-attention features"
- [abstract] ORF-attention features help prediction: "ORF-attention features show positive effects on predicting lncRNA"
- [readme] Repository README confirms five-type integration with CatBoost: "LncCat integrates five types of information to encode transcript sequences and utilizes CatBoost to construct a prediction model."
