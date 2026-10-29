---
name: tncrna-presence-absence-matrix-construction
description: Use when you have tissue-level tncRNA abundance/presence data (e.g.,
  Supplementary sheets 5.1-5.4) for plant samples and need sample-by-sequence binary
  feature matrices for tissue-specific clustering or dimensionality reduction.
when_to_use_negative:
- Input is already a feature/counts matrix with no per-sample tissue annotation to
  apply the >=10-sample tissue filter
- Tissue of interest has fewer than 10 samples, which the paper's matrix-generation
  criterion excludes
- Data are raw sRNA-seq FASTQ reads rather than classified tncRNA detections from
  the tncRNA Toolkit — run the detection pipeline first
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: pandas
  role: Loads Supplementary sheets 5.1-5.4, builds the binary presence/absence matrices,
    and writes the cluster assignment table
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: tncRNA Toolkit
  role: Upstream pipeline that classifies quality-filtered, adapter-trimmed sRNA-seq
    reads into tncRNA subtypes (tRF-5s, tRF-3s, tRF-1, tRNA halves) supplying the
    per-class sequences for the matrices
  repo: https://github.com/skbinfo/tncRNA-Toolkit
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:949d6955a035432623f983f7182524879212daa9954c9f1a8c12fafd737b9442
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# tncrna-presence-absence-matrix-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs class-specific binary presence (1)/absence (0) matrices of unique tncRNA sequences (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH) per plant tissue sample, restricted to tissues with at least 10 samples, to enable tissue-specific clustering of tncRNA profiles (e.g., via t-SNE).

## When to use

Use when you have tissue-level tncRNA abundance/presence data (e.g., Supplementary sheets 5.1-5.4) for plant samples and need sample-by-sequence binary feature matrices for tissue-specific clustering or dimensionality reduction.

## When NOT to use

- Input is already a feature/counts matrix with no per-sample tissue annotation to apply the >=10-sample tissue filter
- Tissue of interest has fewer than 10 samples, which the paper's matrix-generation criterion excludes
- Data are raw sRNA-seq FASTQ reads rather than classified tncRNA detections from the tncRNA Toolkit — run the detection pipeline first

## Inputs

- Tissue-level tncRNA tables from the tncRNA Toolkit output (Supplementary sheets 5.1-5.4), listing unique tncRNA sequences per tissue sample
- Sample-to-tissue metadata sufficient to enforce the >=10 samples-per-tissue filter
- tncRNA class labels (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH) per detected sequence

## Outputs

- Per-class binary presence/absence matrix (samples x unique tncRNA sequences) as a pandas DataFrame
- Sample-to-cluster assignment table (pandas) from downstream clustering
- 2D t-SNE scatter plot per tncRNA class saved to the output directory

## How to apply

Load the tissue-level tncRNA data (Supplementary sheets 5.1-5.4) with pandas, first restricting the analysis to tissues that have at least 10 samples per the paper's criterion ('The tissues with at least 10 samples ... were utilized for matrix generation'). For each tncRNA class separately (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH, as produced by the tncRNA Toolkit pipeline), build a binary matrix where rows are tissue samples and columns are unique tncRNA sequences, assigning 1 when the sequence is detected in the sample and 0 otherwise. The rationale is that spatiotemporal tncRNA expression can be captured as a discrete presence/absence signature, which t-SNE can then embed in 2D to reveal tissue-specific clusters. Record sample-to-cluster assignments in a pandas table, and save the per-class scatter plots and assignment table to the output directory. Classes originate from the tncRNA Toolkit, which classifies tRNA-derived reads mapped exclusively to artificial tRNA-genome chromosomes into these subtypes.

## Related tools

- **pandas** (Loads Supplementary sheets 5.1-5.4, builds the binary presence/absence matrices, and writes the cluster assignment table) — https://github.com/skbinfo/tncRNA-Toolkit
- **tncRNA Toolkit** (Upstream pipeline that classifies quality-filtered, adapter-trimmed sRNA-seq reads into tncRNA subtypes (tRF-5s, tRF-3s, tRF-1, tRNA halves) supplying the per-class sequences for the matrices) — https://github.com/skbinfo/tncRNA-Toolkit

## Examples

```
import pandas as pd; df = pd.read_excel('Supplementary_sheet_5.1.xlsx'); M = pd.crosstab(df['sample'], df['tncRNA_seq']).gt(0).astype(int)
```

## Evaluation signals

- Every tissue retained in the matrix has at least 10 samples
- Matrix cells are strictly binary (0/1) for each of the five class-specific matrices (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH)
- Rows of each matrix correspond one-to-one to tissue samples; columns to unique tncRNA sequences of that class
- Downstream t-SNE scatter plots show distinct tissue-specific clusters, consistent with the paper's finding that 'distinct tissue-specific tncRNA clusters highlight their role in plant development'

## Limitations

- Only tissues with at least 10 samples are eligible; rarer tissues are excluded from matrix generation
- The binary encoding discards abundance information, so within-tissue quantitative differences between samples are not captured
- tncRNA classifications depend on upstream read mapping (bowtie1, uniquely mapping reads to artificial tRNA chromosomes); misclassification would propagate into the matrices

## Evidence
