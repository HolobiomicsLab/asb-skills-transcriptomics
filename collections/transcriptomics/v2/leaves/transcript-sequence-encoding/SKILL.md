---
name: transcript-sequence-encoding
description: Use when lncRNA vs protein-coding transcript (PCT) FASTA sequences need
  to be converted into numeric feature vectors, or when verifying via dimensionality-reduction
  visualization that ORF-attention features separate the two classes.
when_to_use_negative:
- Input is already a numeric feature matrix or embedding — re-extracting sequence
  features would be redundant
- Transcripts are shorter than 200 nt or the study requires distinguishing RNAs without
  ORFs, since ncRNAs shorter than 200 nt are eliminated in this pipeline
- The goal is classifying full-length proteins or genomic DNA rather than transcript-level
  lncRNA/PCT discrimination
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
tools:
- name: LncCat
  role: Framework whose feature-encoding scheme (five fused feature types) this skill
    reproduces
  repo: https://github.com/a525076133/LncCat
- name: Orfipy
  role: ORF extraction from transcript sequences using Standard Code (transl_table=1)
- name: BERT
  role: Pre-trained model producing contextual vector representations of the longest-ORF
    peptide sequences (ORF-attention features)
- name: UMAP
  role: 2D dimensionality reduction to visualize the distribution of lncRNAs vs PCTs
    in feature space
- name: CD-HIT
  role: Redundancy reduction of the transcript datasets at an 80% similarity threshold
    before feature extraction
- name: CatBoost
  role: Downstream classifier the encoded feature vectors feed into for lncRNA prediction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.02.012
    title: 'LncCat: An ORF attention model to identify LncRNA based on ensemble learning
      strategy and fused sequence information'
  source_content_hash: sha256:11e8a3a6ab2ae1419924db5c1dfbdd2e64077f7cffcc646c284d2fd83fc899d9
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
  - CatBoost
  - LncCat
  - Orfipy
  - UMAP
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# transcript-sequence-encoding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encode lncRNA/PCT transcript sequences into fused feature vectors (codon-related, GC-related, sequence-related, peptide-related, and ORF-attention features) as used by LncCat, then embed them in 2D with UMAP to verify that ORF-attention features discriminate lncRNAs from protein-coding transcripts.

## When to use

Use when lncRNA vs protein-coding transcript (PCT) FASTA sequences need to be converted into numeric feature vectors, or when verifying via dimensionality-reduction visualization that ORF-attention features separate the two classes.

## When NOT to use

- Input is already a numeric feature matrix or embedding — re-extracting sequence features would be redundant
- Transcripts are shorter than 200 nt or the study requires distinguishing RNAs without ORFs, since ncRNAs shorter than 200 nt are eliminated in this pipeline
- The goal is classifying full-length proteins or genomic DNA rather than transcript-level lncRNA/PCT discrimination

## Inputs

- Deduplicated lncRNA and PCT transcript FASTA sequences (from NCBI RefSeq, Ensembl, or GENCODE), ≥200 nt
- ORF annotations extracted by Orfipy (Standard Code, transl_table=1, start codons ATG/TTG/CTG, stop codons TAA/TAG/TGA)
- Pre-trained BERT model weights for peptide sequence encoding

## Outputs

- Fused ORF-attention feature vectors per transcript (codon-, GC-, sequence-, peptide-related features plus BERT-encoded longest-ORF peptide)
- Contrast base feature vectors lacking ORF-attention features
- 2D UMAP embeddings of both feature sets
- Per-species scatter figure comparing without vs with ORF-attention class separation (umap_orf_attention_distribution.png)

## How to apply

Collect transcript sequences for the species of interest and deduplicate with CD-HIT at an 80% similarity threshold, eliminating ncRNAs shorter than 200 nt. Extract ORFs with Orfipy using Standard Code (transl_table=1) with start codons ATG/TTG/CTG and stop codons TAA/TAG/TGA, then compute codon-related, GC-related, sequence-related, and peptide-related features for the top three longest ORFs. Encode the longest-ORF peptide sequence with a pre-trained BERT model to obtain contextual vector representations, and combine these into ORF-attention feature vectors; also assemble a contrast base feature set lacking ORF-attention features. Embed both feature sets into two dimensions with UMAP and render per-species 2D scatter plots colored by lncRNA vs PCT class. The rationale is that the comparison of visualizations reflects that lncRNAs and PCTs are significantly different in ORF-attention feature space, which underpins LncCat's CatBoost-based prediction performance.

## Related tools

- **LncCat** (Framework whose feature-encoding scheme (five fused feature types) this skill reproduces) — https://github.com/a525076133/LncCat
- **Orfipy** (ORF extraction from transcript sequences using Standard Code (transl_table=1))
- **BERT** (Pre-trained model producing contextual vector representations of the longest-ORF peptide sequences (ORF-attention features))
- **UMAP** (2D dimensionality reduction to visualize the distribution of lncRNAs vs PCTs in feature space)
- **CD-HIT** (Redundancy reduction of the transcript datasets at an 80% similarity threshold before feature extraction)
- **CatBoost** (Downstream classifier the encoded feature vectors feed into for lncRNA prediction)

## Examples

```
import umap; embedding = umap.UMAP(n_components=2).fit_transform(feature_matrix)
```

## Evaluation signals

- With-ORF-attention UMAP panels show lncRNAs and PCTs adequately divided into two clusters, while without-ORF-attention panels show poor separation (Fig. 2-(1)s vs Fig. 2-(2)s behavior)
- Every transcript passed CD-HIT 80% deduplication and the 200 nt length filter before encoding
- ORF extraction used transl_table=1 with exactly the start codons ATG/TTG/CTG and stop codons TAA/TAG/TGA
- Feature vectors contain all five LncCat feature types; the contrast set deliberately omits only ORF-attention features
- Downstream CatBoost models reach MCCs in the reported ranges (e.g. 0.9503 human, 0.9219 mouse, 0.8591 zebrafish, 0.8672 wheat, 0.9047 chicken)

## Limitations

- No public code repository for the LncCat model itself (only the web server at http://cczubio.top/lnccat/), so exact encoding details may need reimplementation
- No version of CatBoost or BERT is specified for reproducibility, and CatBoost hyperparameter tuning settings are not reported
- Species may lack annotations, making it impossible to build models for all species

## Evidence
