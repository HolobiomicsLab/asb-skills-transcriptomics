---
name: gc-content-frame-feature-engineering
description: Use when encoding transcript FASTA sequences (lncRNA vs PCT classification)
  and GC-composition descriptors are needed as classifier inputs, e.g. feeding CatBoost-style
  models like LncCat.
when_to_use_negative:
- Input is already a numeric feature matrix — re-deriving GC descriptors from encoded
  vectors is redundant
- Sequences are raw reads or protein sequences rather than nucleotide transcripts,
  since GC1/GC2/GC3 positional features require codon-framed nucleotide sequence
- Task is lncRNA function annotation rather than coding-potential classification —
  GC features alone do not provide functional interpretation
edam_operation: http://edamontology.org/operation_0236
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0091
tools:
- name: LncCat
  role: Consuming model whose 599-dimensional encoding integrates the GC-related features
    with codon-, sequence-, peptide-, and ORF-attention features before CatBoost classification
  repo: https://github.com/a525076133/LncCat
- name: Orfipy
  role: Extracts ORFs (Standard Code, transl_table=1) from each transcript so GC frame
    scores can be computed on ORF sequences as well as full transcripts
- name: CatBoost
  role: Trains the classifier on the feature matrix containing the GC-related block
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.02.012
    title: 'LncCat: An ORF attention model to identify LncRNA based on ensemble learning
      strategy and fused sequence information'
  source_content_hash: sha256:ea860d72c8e4bb546d5ebd2510560aa99864edff9f4d151657fa4345d0fae985
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
  edam_operation: http://edamontology.org/operation_0236
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# GC-content-frame feature engineering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes GC-related features — overall GC content plus positional GC1/GC2/GC3 and GC frame scores — for full transcript sequences and their extracted ORFs, as the GC-related block of LncCat's 599-dimensional feature vector used to distinguish lncRNAs from protein-coding transcripts (PCTs).

## When to use

Use when encoding transcript FASTA sequences (lncRNA vs PCT classification) and GC-composition descriptors are needed as classifier inputs, e.g. feeding CatBoost-style models like LncCat.

## When NOT to use

- Input is already a numeric feature matrix — re-deriving GC descriptors from encoded vectors is redundant
- Sequences are raw reads or protein sequences rather than nucleotide transcripts, since GC1/GC2/GC3 positional features require codon-framed nucleotide sequence
- Task is lncRNA function annotation rather than coding-potential classification — GC features alone do not provide functional interpretation

## Inputs

- CD-HIT-reduced transcript FASTA sequences (human, mouse, zebrafish, wheat, chicken) from NCBI RefSeq, Ensembl, GENCODE
- ORF coordinates from Orfipy (Standard Code, transl_table=1) per transcript

## Outputs

- GC-related feature block (GC content, GC1/GC2/GC3, GC frame scores) per transcript and per ORF
- Rows contributing to the merged 599-dimensional feature matrix (orf_attention_features.csv) with transcript IDs and lncRNA/PCT class labels

## How to apply

Start from the CD-HIT-reduced (80% identity threshold) transcript FASTA datasets collected from NCBI RefSeq, Ensembl, and GENCODE, with ncRNAs shorter than 200 nt removed. Extract ORFs from every transcript with Orfipy using Standard Code (transl_table=1, start codons ATG/TTG/CTG, stop codons TAA/TAG/TGA) so that GC frame features can be computed on both the whole transcript and its ORFs. Calculate overall GC content and position-specific scores GC1, GC2, GC3 (GC at first, second, and third codon positions) together with GC frame scores, following the GC-related feature block defined alongside codon-related, sequence-related, peptide-related, and ORF-attention features. Concatenate these GC descriptors with the other four feature types into the single 599-dimensional vector per transcript before training the CatBoost classifier. Rationale: frame- and position-resolved GC statistics reflect coding-potential signals that differ between lncRNAs and PCTs, including transcripts with small ORFs (sORFs).

## Related tools

- **LncCat** (Consuming model whose 599-dimensional encoding integrates the GC-related features with codon-, sequence-, peptide-, and ORF-attention features before CatBoost classification) — https://github.com/a525076133/LncCat
- **Orfipy** (Extracts ORFs (Standard Code, transl_table=1) from each transcript so GC frame scores can be computed on ORF sequences as well as full transcripts)
- **CatBoost** (Trains the classifier on the feature matrix containing the GC-related block)

## Examples

```
orfipy transcripts.fa --out orfs.txt --pep orfs.pep --table 1  # then compute GC content and GC1/GC2/GC3 frame scores per transcript and ORF
```

## Evaluation signals

- Each input transcript yields exactly one row in the merged feature matrix (one row per input transcript)
- Feature dimensionality of the concatenated matrix is 599, consistent with Supplementary Table S1-1
- GC1/GC2/GC3 values each lie in [0, 1] and overall GC content equals the fraction of G/C bases in the transcript
- Downstream CatBoost performance matches reported metrics, e.g. MCC 0.9503 and F1 0.9742 on the human test dataset
- UMAP visualization of the encoded features shows lncRNAs and PCTs forming distinguishable clusters

## Limitations

- CatBoost and BERT versions are unspecified in the paper, limiting reproducibility of the full feature pipeline
- Cross-species transfer is limited: human, mouse, and zebrafish models perform poorly on wheat (MCC 0.67, 0.63, 0.66), so GC descriptors learned on one species may not generalize
- Some species lack annotations, making it impossible to build models for all species

## Evidence
