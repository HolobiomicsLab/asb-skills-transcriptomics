---
name: tsne-dimensionality-reduction
description: Use when sample-wise small-RNA profiles (e.g. tncRNA sequence presence
  per tissue sample) need non-linear 2D embedding to test whether samples cluster
  by tissue of origin.
when_to_use_negative:
- Input is already a low-dimensional embedding or a continuous expression matrix where
  linear methods suffice
- Tissues have fewer than 10 samples — the paper only utilized tissues with at least
  10 samples for t-SNE matrix generation
- Goal is tncRNA detection or classification rather than sample-level clustering/visualization
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3170
tools:
- name: tncRNA Toolkit
  role: Source workflow producing the tncRNA classes and data whose tissue-wise presence/absence
    profiles feed the t-SNE matrices
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: pandas
  role: Loads Supplementary sheets 5.1–5.4, builds the binary presence/absence matrices,
    and writes the sample-to-cluster assignment table
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:5aa1d14d49abb61a256fc4bd32e911d111f6bde590dafda11916d6aef191eb59
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/skbinfo/tncRNA-Toolkit
  repo_url: https://github.com/skbinfo/tncRNA-Toolkit
  tools:
  - pandas
  - tncRNA Toolkit
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# tsne-dimensionality-reduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies t-SNE non-linear dimensionality reduction to binary presence/absence matrices of unique tncRNA sequences (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH) per tissue sample, to reveal tissue-specific clustering of plant tncRNAs under normal physiological conditions.

## When to use

Use when sample-wise small-RNA profiles (e.g. tncRNA sequence presence per tissue sample) need non-linear 2D embedding to test whether samples cluster by tissue of origin.

## When NOT to use

- Input is already a low-dimensional embedding or a continuous expression matrix where linear methods suffice
- Tissues have fewer than 10 samples — the paper only utilized tissues with at least 10 samples for t-SNE matrix generation
- Goal is tncRNA detection or classification rather than sample-level clustering/visualization

## Inputs

- Tissue-level tncRNA tables (Supplementary sheets 5.1–5.4) readable with pandas
- Sample-to-tissue metadata mapping
- Unique tncRNA sequence lists per class (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH)

## Outputs

- 2D t-SNE scatter plots (one per tncRNA class: tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH)
- Sample-to-cluster assignment table saved via pandas
- Binary presence/absence matrices per tncRNA class

## How to apply

Load tissue-level tncRNA data (Supplementary sheets 5.1–5.4) with pandas, restricting to tissues with at least 10 samples per the paper's criterion. Build a binary 1/0 presence–absence matrix of unique tncRNA sequences per tissue sample, one matrix per tncRNA class (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH). Apply t-SNE to each class-specific matrix to embed samples in 2D, then assign each sample to its nearest cluster and record assignments in a pandas table. The rationale is that binary sequence-presence vectors capture spatiotemporal expression patterns, and t-SNE exposes tissue-specific organization that linear methods may not. Validate by confirming the scatter plots show distinct tissue-specific tncRNA clusters.

## Related tools

- **tncRNA Toolkit** (Source workflow producing the tncRNA classes and data whose tissue-wise presence/absence profiles feed the t-SNE matrices) — https://github.com/skbinfo/tncRNA-Toolkit
- **pandas** (Loads Supplementary sheets 5.1–5.4, builds the binary presence/absence matrices, and writes the sample-to-cluster assignment table)

## Examples

```
import pandas as pd; from sklearn.manifold import TSNE; X = pd.read_excel('sheet5.1.xlsx', index_col=0); emb = TSNE(n_components=2, random_state=42).fit_transform(X.values)
```

## Evaluation signals

- t-SNE scatter plots show distinct clusters whose membership corresponds to tissue of origin, consistent with the paper's finding of tissue-specific tncRNA clusters
- Only tissues with at least 10 samples are included in each class-specific matrix
- Each binary matrix contains exactly 1/0 entries marking presence/absence of unique tncRNA sequences per sample
- A cluster-assignment table exists for each of the five tncRNA classes (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH) and every sample is assigned to exactly one nearest cluster
- Plots and the assignment table are written to the output directory as artifacts

## Limitations

- Binary presence/absence encoding discards abundance information, so clusters reflect sequence occurrence patterns rather than expression levels
- t-SNE embeddings are stochastic; cluster assignments depend on nearest-neighbor assignment and may vary between runs
- Tissues with fewer than 10 samples are excluded, limiting coverage to well-sampled tissues

## Evidence
