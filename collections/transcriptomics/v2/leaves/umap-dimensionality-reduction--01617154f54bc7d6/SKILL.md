---
name: umap-dimensionality-reduction
description: Use when you have high-dimensional per-transcript feature vectors (e.g.,
  codon-, GC-, sequence-, peptide-, or BERT-encoded ORF-attention features) and need
  a 2D scatter visualization to check whether lncRNAs and PCTs form separable clusters.
when_to_use_negative:
- Input is already a low-dimensional (≤3D) representation where a scatter plot suffices
  without embedding
- The goal is quantitative classification performance (MCC/F1) rather than qualitative
  cluster-separation visualization — use the CatBoost classifier directly
- Sequences have not yet been filtered and deduplicated (CD-HIT 80%, ≥200 nt), so
  the embedding would be dominated by redundant or fragmentary transcripts
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: UMAP
  role: Embeds the ORF-attention and base feature vectors into two dimensions for
    visual inspection of lncRNA/PCT cluster separation
- name: LncCat
  role: Source of the five-type feature encoding (including ORF-attention features)
    that is visualized; code at https://github.com/a525076133/LncCat
  repo: https://github.com/a525076133/LncCat
- name: Orfipy
  role: Extracts ORFs (transl_table=1; start ATG/TTG/CTG; stop TAA/TAG/TGA) used to
    compute the features fed to UMAP
- name: BERT
  role: Pre-trained model producing contextual vector representations of longest-ORF
    peptide sequences, forming the ORF-attention feature block
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.02.012
    title: 'LncCat: An ORF attention model to identify LncRNA based on ensemble learning
      strategy and fused sequence information'
  source_content_hash: sha256:6c03ba0c307771afa1eb49629c27a98fff5c0cbacf5f48323280ad44fbbb2e1e
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
  - UMAP
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# umap-dimensionality-reduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies Uniform Manifold Approximation and Projection (UMAP) to embed high-dimensional transcript feature vectors (e.g., ORF-attention features from LncCat) into two dimensions so that lncRNA vs. PCT class separation can be visually assessed.

## When to use

Use when you have high-dimensional per-transcript feature vectors (e.g., codon-, GC-, sequence-, peptide-, or BERT-encoded ORF-attention features) and need a 2D scatter visualization to check whether lncRNAs and PCTs form separable clusters.

## When NOT to use

- Input is already a low-dimensional (≤3D) representation where a scatter plot suffices without embedding
- The goal is quantitative classification performance (MCC/F1) rather than qualitative cluster-separation visualization — use the CatBoost classifier directly
- Sequences have not yet been filtered and deduplicated (CD-HIT 80%, ≥200 nt), so the embedding would be dominated by redundant or fragmentary transcripts

## Inputs

- Feature matrix of encoded transcript sequences (with ORF-attention features: codon-related, GC-related, sequence-related, peptide-related, plus BERT-encoded longest-ORF peptide vectors)
- Contrast feature matrix lacking ORF-attention features (base feature set)
- Per-transcript class labels (lncRNA vs. PCT) from the five species datasets (human, mouse, zebrafish, wheat, chicken; NCBI RefSeq / Ensembl / GENCODE, CD-HIT 80% deduplicated, ≥200 nt)

## Outputs

- Two-dimensional UMAP embedding of each feature matrix
- Per-species 2D scatter plots colored by lncRNA vs. PCT class for the with- and without-ORF-attention conditions
- Comparison figure (e.g., umap_orf_attention_distribution.png)

## How to apply

First assemble the feature matrix: in the LncCat workflow this means computing codon-related, GC-related, sequence-related, and peptide-related features for the top three longest ORFs extracted with Orfipy (Standard Code, transl_table=1, start codons ATG/TTG/CTG, stop codons TAA/TAG/TGA) and concatenating the pre-trained BERT embedding of the longest-ORF peptide sequence into ORF-attention feature vectors; build a contrast matrix lacking the ORF-attention block. Fit UMAP with n_components=2 on each feature set independently and render per-species 2D scatter plots colored by class (lncRNA vs. PCT). The rationale is that if ORF-attention features carry coding-potential signal, the with-attention embedding should show two distinct clusters while the without-attention embedding should show poor separation, providing visual evidence for the finding that ORF-attention features positively affect lncRNA prediction.

## Related tools

- **UMAP** (Embeds the ORF-attention and base feature vectors into two dimensions for visual inspection of lncRNA/PCT cluster separation)
- **LncCat** (Source of the five-type feature encoding (including ORF-attention features) that is visualized; code at https://github.com/a525076133/LncCat) — https://github.com/a525076133/LncCat
- **Orfipy** (Extracts ORFs (transl_table=1; start ATG/TTG/CTG; stop TAA/TAG/TGA) used to compute the features fed to UMAP)
- **BERT** (Pre-trained model producing contextual vector representations of longest-ORF peptide sequences, forming the ORF-attention feature block)

## Examples

```
import umap; embedding = umap.UMAP(n_components=2).fit_transform(feature_matrix)
```

## Evaluation signals

- With-ORF-attention UMAP panels show lncRNAs and PCTs adequately divided into two clusters
- Without-ORF-attention UMAP panels show poor lncRNA/PCT separation, providing the expected contrast (Fig. 2-(1)s vs Fig. 2-(2)s pattern)
- Embedding has exactly 2 dimensions (n_components=2) and one point per transcript, colored by class
- A separate embedding is produced per species (human, mouse, zebrafish, wheat, chicken) and per condition (with/without ORF-attention)

## Limitations

- UMAP visualization is qualitative; cluster separation seen in the 2D projection does not itself yield MCC/F1 metrics and depends on hyperparameters not reported in the paper
- No version of the BERT model or UMAP parameters is specified for reproducibility, so exact embeddings may not be bit-identical across runs
- The LncCat repository offers only a web server (http://cczubio.top/lnccat/) with email-returned results, limiting programmatic access to the feature pipeline being visualized

## Evidence
