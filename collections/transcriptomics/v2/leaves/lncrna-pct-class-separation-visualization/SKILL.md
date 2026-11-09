---
name: lncrna-pct-class-separation-visualization
description: Use when you need to visually demonstrate that a fused feature representation
  (e.g. ORF-attention features from BERT-encoded ORF peptides) discriminates lncRNAs
  from PCTs in feature space, via per-species UMAP scatter plots colored by class.
when_to_use_negative:
- Input is already a 2D embedding or a rendered figure — this skill requires raw per-transcript
  feature vectors as input
- The goal is quantitative classification performance (MCC/F1) rather than qualitative
  class-separation visualization — use the LncCat CatBoost prediction workflow instead
- Transcripts are shorter than 200 nt or the dataset is not redundancy-reduced, since
  the paper's filtering (200 nt minimum, CD-HIT 80%) defines the data regime being
  visualized
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3474
tools:
- name: Orfipy
  role: Extracts ORFs from transcripts under Standard Code (transl_table=1) with start
    codons ATG/TTG/CTG and stop codons TAA/TAG/TGA, providing the peptide sequences
    and ORF structure used to build the features being visualized
- name: BERT
  role: Pre-trained model used to encode the longest-ORF peptide sequences into contextual
    vector representations that, combined with base features, form the ORF-attention
    feature vectors
- name: UMAP
  role: Uniform Manifold Approximation and Projection used to reduce both feature
    sets to two dimensions for the lncRNA vs PCT distribution visualization
- name: LncCat
  role: The published pipeline whose five-type feature encoding (codon-related, GC-related,
    sequence-related, peptide-related, ORF-attention) supplies the feature vectors
    embedded by UMAP
  repo: https://github.com/a525076133/LncCat
- name: CD-HIT
  role: Reduces dataset redundancy at an 80% similarity threshold before feature extraction
    so the visualization reflects the paper's non-redundant benchmark datasets
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.02.012
    title: 'LncCat: An ORF attention model to identify LncRNA based on ensemble learning
      strategy and fused sequence information'
  source_content_hash: sha256:712e5c3bde7e94ed7a16c82f6906ed015632ebc0568ae16ab6f2da2e0d35b618
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
  - UMAP
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# lncrna-pct-class-separation-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce the LncCat paper's UMAP comparison showing that ORF-attention feature vectors separate lncRNAs from protein-coding transcripts (PCTs) into distinct 2D clusters, whereas base features lacking ORF-attention do not, across five species datasets.

## When to use

Use when you need to visually demonstrate that a fused feature representation (e.g. ORF-attention features from BERT-encoded ORF peptides) discriminates lncRNAs from PCTs in feature space, via per-species UMAP scatter plots colored by class.

## When NOT to use

- Input is already a 2D embedding or a rendered figure — this skill requires raw per-transcript feature vectors as input
- The goal is quantitative classification performance (MCC/F1) rather than qualitative class-separation visualization — use the LncCat CatBoost prediction workflow instead
- Transcripts are shorter than 200 nt or the dataset is not redundancy-reduced, since the paper's filtering (200 nt minimum, CD-HIT 80%) defines the data regime being visualized

## Inputs

- Five species lncRNA/PCT transcript FASTA datasets (human, mouse, zebrafish, wheat, chicken) from NCBI RefSeq, Ensembl, and GENCODE, deduplicated with CD-HIT at 80% and filtered to >=200 nt
- Feature matrices per transcript: ORF-attention feature vectors (base features + BERT-encoded longest-ORF peptide embeddings) and base feature set (codon-related, GC-related, sequence-related, peptide-related only)
- Class labels (lncRNA vs PCT) per transcript

## Outputs

- Per-species 2D UMAP embeddings for both the with-ORF-attention and without-ORF-attention conditions
- Composed comparison figure (umap_orf_attention_distribution.png) with per-species scatter panels colored by lncRNA vs PCT class

## How to apply

Load the five-species lncRNA/PCT transcript datasets (NCBI RefSeq, Ensembl, GENCODE-derived), deduplicated with CD-HIT at an 80% threshold and filtered to transcripts of at least 200 nt. Extract ORFs with Orfipy under Standard Code (transl_table=1) with start codons ATG/TTG/CTG and stop codons TAA/TAG/TGA, then compute codon-related, GC-related, sequence-related, and peptide-related features for the top three longest ORFs and encode the longest-ORF peptide with a pre-trained BERT model to build ORF-attention vectors; also assemble the base feature set lacking ORF-attention features for the contrast condition. Embed both feature sets into two dimensions with UMAP and render per-species 2D scatters colored by lncRNA vs PCT class for both conditions, composing the comparison figure (e.g. umap_orf_attention_distribution.png). The rationale is that visualization directly evidences the paper's finding that ORF-attention features carry class-discriminative signal beyond base features.

## Related tools

- **Orfipy** (Extracts ORFs from transcripts under Standard Code (transl_table=1) with start codons ATG/TTG/CTG and stop codons TAA/TAG/TGA, providing the peptide sequences and ORF structure used to build the features being visualized)
- **BERT** (Pre-trained model used to encode the longest-ORF peptide sequences into contextual vector representations that, combined with base features, form the ORF-attention feature vectors)
- **UMAP** (Uniform Manifold Approximation and Projection used to reduce both feature sets to two dimensions for the lncRNA vs PCT distribution visualization)
- **LncCat** (The published pipeline whose five-type feature encoding (codon-related, GC-related, sequence-related, peptide-related, ORF-attention) supplies the feature vectors embedded by UMAP) — https://github.com/a525076133/LncCat
- **CD-HIT** (Reduces dataset redundancy at an 80% similarity threshold before feature extraction so the visualization reflects the paper's non-redundant benchmark datasets)

## Examples

```
python make_umap_orf_attention_distribution.py --features orf_attention_features.csv --base_features base_features.csv --labels classes.csv --out umap_orf_attention_distribution.png
```

## Evaluation signals

- With-ORF-attention panels must show lncRNAs and PCTs adequately divided into two clusters, matching the Fig. 2-(2) distributions reported in the paper
- Without-ORF-attention panels must show poor lncRNA/PCT separation, matching the Fig. 2-(1) contrast condition
- The separation pattern should be reproduced across all five species (human, mouse, zebrafish, wheat, chicken) panels
- Output figure must be a composed per-species, two-condition comparison (umap_orf_attention_distribution.png) with points colored by lncRNA vs PCT class
- Embeddings must be 2-dimensional UMAP projections of the exact feature sets described (base features vs base + ORF-attention features)

## Limitations

- The LncCat repository provides code and datasets, but the pre-trained BERT model version and CatBoost hyperparameters are not specified, so the exact ORF-attention vectors may not be bit-for-bit reproducible
- Separation quality is a qualitative visual claim (two-cluster adequacy), not a quantitative metric; cluster boundaries may be subject to interpretation
- Species may lack annotations, making it impossible to build feature sets for all species as noted in the paper's discussion

## Evidence
