---
name: cluster-assignment-tabulation
description: Use when sample-wise tncRNA profiles (Supplementary sheets 5.1–5.4) must
  be reduced with t-SNE to 2D and each tissue sample assigned to its nearest cluster
  and logged in a tabular record.
when_to_use_negative:
- Tissues in the input have fewer than 10 samples — the paper restricts t-SNE matrix
  generation to tissues with at least 10 samples (Supplementary sheets 5.1–5.4).
- The question is differential expression or stress-response cataloging of tncRNAs,
  which relies on abundance/count data rather than binary presence/absence tissue
  clustering.
- A 2D embedding or cluster-label table already exists for the samples; re-embedding
  with t-SNE would reassign labels non-deterministically.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: tncRNA Toolkit
  role: Source of the tncRNA class calls (tRF-5s, tRF-3s, tRF-1, tRNA halves) whose
    per-tissue sample profiles feed the binary matrices that are clustered and tabulated
    here
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: pandas
  role: Loads Supplementary sheets 5.1–5.4, builds the class-specific binary presence/absence
    matrices, and writes the sample-to-cluster assignment table
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:d8d718180c5425ca6ddcf260cc35d077ed4b9ea86567feb6742122b0c070637f
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cluster-assignment-tabulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tabulates sample-to-cluster assignments after t-SNE embedding of tissue-wise tncRNA presence/absence profiles, capturing the tissue-specific clustering of plant tncRNAs (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH) reported in the paper.

## When to use

Use when sample-wise tncRNA profiles (Supplementary sheets 5.1–5.4) must be reduced with t-SNE to 2D and each tissue sample assigned to its nearest cluster and logged in a tabular record.

## When NOT to use

- Tissues in the input have fewer than 10 samples — the paper restricts t-SNE matrix generation to tissues with at least 10 samples (Supplementary sheets 5.1–5.4).
- The question is differential expression or stress-response cataloging of tncRNAs, which relies on abundance/count data rather than binary presence/absence tissue clustering.
- A 2D embedding or cluster-label table already exists for the samples; re-embedding with t-SNE would reassign labels non-deterministically.

## Inputs

- Tissue-level tncRNA tables (Supplementary sheets 5.1–5.4) mapping unique tncRNA sequences to tissue samples
- tncRNA class labels per sequence: tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH

## Outputs

- Sample-to-cluster assignment table (pandas DataFrame, one row per tissue sample per tncRNA class)
- 2D t-SNE scatter plots of sample-wise tncRNA profiles, one per tncRNA class, saved to the output directory

## How to apply

Load the tissue-level tncRNA data (Supplementary sheets 5.1–5.4) with pandas and retain only tissues with at least 10 samples, the paper's criterion for t-SNE matrix generation. For each tncRNA class (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH), construct a binary 1/0 presence–absence matrix of unique tncRNA sequence observations per tissue sample. Apply t-SNE (non-linear dimensionality reduction) to each class-specific binary matrix to embed the samples in 2D. Assign every embedded sample to its nearest cluster and record the sample-to-cluster assignments in a pandas DataFrame. Save the assignment table together with one t-SNE scatter plot per class to the output directory; visually distinct tissue-specific clusters in these plots validate the run against the paper's finding that tissue-specific tncRNA clusters highlight roles in plant development.

## Related tools

- **tncRNA Toolkit** (Source of the tncRNA class calls (tRF-5s, tRF-3s, tRF-1, tRNA halves) whose per-tissue sample profiles feed the binary matrices that are clustered and tabulated here) — https://github.com/skbinfo/tncRNA-Toolkit
- **pandas** (Loads Supplementary sheets 5.1–5.4, builds the class-specific binary presence/absence matrices, and writes the sample-to-cluster assignment table)

## Examples

```
import pandas as pd; sheets = pd.read_excel('Supplementary_5.1-5.4.xlsx', sheet_name=None)  # filter tissues with >=10 samples, then t-SNE per class
```

## Evaluation signals

- Each retained tissue has ≥10 samples, matching the paper's matrix-generation criterion on Supplementary sheets 5.1–5.4.
- Class-specific matrices are strictly binary (1/0) presence/absence of unique tncRNA sequences per tissue sample.
- The assignment table carries exactly one non-null cluster label per sample for each of the five classes (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH).
- t-SNE scatter plots per class show visually distinct tissue-specific clusters, consistent with the paper's finding.
- Both the plots and the cluster assignment table are present in the output directory after the run.

## Limitations

- Only tissues with at least 10 samples were utilized for matrix generation, so small tissues are excluded from the clustering.
- Nearest-cluster assignment on a stochastic t-SNE embedding is not a formal probabilistic clustering; labels can vary between runs unless the random seed is fixed.
- Binary presence/absence encoding discards tncRNA abundance, and coverage is limited to the paper's sRNA-seq-derived classes (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH).

## Evidence
