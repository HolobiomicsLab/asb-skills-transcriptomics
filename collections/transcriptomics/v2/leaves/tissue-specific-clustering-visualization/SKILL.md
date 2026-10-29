---
name: tissue-specific-clustering-visualization
description: Use when you have per-sample tncRNA detection data (e.g. Supplementary
  sheets 5.1–5.4) across plant tissues and want to test whether tncRNA composition
  clusters by tissue via 2D non-linear embedding.
when_to_use_negative:
- Tissues with fewer than 10 samples — the paper explicitly restricted matrix generation
  to tissues with at least 10 samples, so sparse tissues should be excluded first
- Input is already an expression/count matrix rather than per-sample tncRNA detection
  lists — the skill expects building a binary presence/absence matrix from unique
  sequences
- Questioning differential expression of tncRNAs under stress rather than tissue-specific
  organization under normal physiological conditions — that requires a DE analysis
  skill instead
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: tncRNA Toolkit
  role: Source pipeline that detected and classified the tncRNAs (tRF-5s, tRF-3s,
    tRF-1, tRNA halves) feeding the binary matrices used for t-SNE
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: pandas
  role: Loads Supplementary sheets 5.1–5.4, constructs the binary presence/absence
    matrices, and records the sample-to-cluster assignment table
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:273dcdda5dbe37b225975b9bf5acc8bb880116899a2e70f2d5ee9b785d3c84a2
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# tissue-specific-clustering-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds binary presence/absence matrices of unique tncRNA sequences (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH) per plant tissue sample and applies t-SNE to each class-specific matrix, revealing tissue-specific tncRNA clusters that indicate spatiotemporal expression under normal physiological conditions.

## When to use

Use when you have per-sample tncRNA detection data (e.g. Supplementary sheets 5.1–5.4) across plant tissues and want to test whether tncRNA composition clusters by tissue via 2D non-linear embedding.

## When NOT to use

- Tissues with fewer than 10 samples — the paper explicitly restricted matrix generation to tissues with at least 10 samples, so sparse tissues should be excluded first
- Input is already an expression/count matrix rather than per-sample tncRNA detection lists — the skill expects building a binary presence/absence matrix from unique sequences
- Questioning differential expression of tncRNAs under stress rather than tissue-specific organization under normal physiological conditions — that requires a DE analysis skill instead

## Inputs

- Tissue-level tncRNA presence data per sample (Supplementary sheets 5.1–5.4, one per tncRNA class)
- Sample-to-tissue metadata (to enforce the ≥10 samples per tissue criterion and color/label clusters)

## Outputs

- 2D t-SNE scatter plot per tncRNA class (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH)
- Sample-to-cluster assignment table (pandas-compatible)
- Binary presence/absence matrix of unique tncRNA sequences per tissue sample per class

## How to apply

Load the tissue-level tncRNA data (Supplementary sheets 5.1–5.4) with pandas and restrict the analysis to tissues having at least 10 samples, per the paper's criterion. For each tncRNA class (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH), construct a binary matrix where rows are tissue samples and columns are unique tncRNA sequences, encoding presence as 1 and absence as 0. Apply t-SNE to embed each class-specific binary matrix into 2D, then assign each sample to its nearest cluster and record the sample-to-cluster assignments in a pandas table. Generate a t-SNE scatter plot per class and save both the plots and the assignment table to the output directory. The rationale is that codon-dependent, precisely cleaved tncRNAs should show non-random, tissue-specific organization if they play roles in plant development.

## Related tools

- **tncRNA Toolkit** (Source pipeline that detected and classified the tncRNAs (tRF-5s, tRF-3s, tRF-1, tRNA halves) feeding the binary matrices used for t-SNE) — https://github.com/skbinfo/tncRNA-Toolkit
- **pandas** (Loads Supplementary sheets 5.1–5.4, constructs the binary presence/absence matrices, and records the sample-to-cluster assignment table)

## Examples

```
import pandas as pd; from sklearn.manifold import TSNE; X = pd.read_csv('sheet_5_1.tsv', sep='\t', index_col=0); emb = TSNE(n_components=2).fit_transform((X > 0).astype(int))
```

## Evaluation signals

- Each input tissue retained in the analysis has at least 10 samples
- The binary matrix contains only {0, 1} values with rows = tissue samples and columns = unique tncRNA sequences, one matrix per tncRNA class
- Resulting t-SNE scatter plots show distinct tissue-specific clusters, consistent with the paper's finding that tncRNAs cluster by tissue
- The sample-to-cluster assignment table covers all samples embedded in each t-SNE plot with matching sample counts
- All five tncRNA classes (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH) yield separate plots and tables saved to the output directory

## Limitations

- t-SNE results depend on the binary presence/absence encoding, which discards abundance information; the paper classified reads as tncRNAs only if they mapped exclusively to artificial chromosomes, so ambiguous reads were already removed upstream
- The analysis reflects normal physiological conditions; differential expression under abiotic/biotic stresses is a separate question the paper addressed separately
- The clustering is non-linear and stochastic (t-SNE), so cluster shapes/positions may vary between runs even when tissue separation is preserved

## Evidence
