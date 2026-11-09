---
name: mutual-information-network-inference
description: Use when you have a raw or normalized RNA-Seq time-series count table
  (e.g., DREAM 4 100 genes x 21 samples, B. subtilis 3997 genes x 7 samples) and need
  a first-pass directed, signed GRN before TFBM/database refinement.
when_to_use_negative:
- Data are single-cell RNA-Seq — the article notes alternative tools (Inferelator
  3.0, GRNBoost2, Arboreto) are typically designed for scRNA-Seq, while Augusta targets
  bulk time-series count tables
- You need a fully refined network with verified interactions — MI-only inference
  must be followed by de novo TFBM discovery (MEME Suite) and curated database search
  (OmniPath, Signor, SignaLink, TRRUST)
- Only a single time point or non-time-series expression data are available — edge
  orientation/sign relies on the most significant difference among adjacent pairs
  of time points
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2259
tools:
- name: Augusta
  role: Python package performing the MI-based initial GRN inference from the RNA-Seq
    count table and exporting the adjacency matrix
  repo: https://github.com/JanaMus/Augusta
- name: Python
  role: Implementation language of the Augusta package (version 3.7 or 3.8)
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  - task_005
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:a1f94dbc99f6ad4388f38bfd0795663f89e617acbfbd423f410572e842fd6d8a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/WanluLiuLab/CRISPRTE
  repo_url: https://github.com/JanaMus/Augusta
  tools:
  - Augusta
  - GENIE3
  - KBoost
  - MEME Suite
  - OmniPath
  - Python
  - SignaLink
  - Signor
  - TDAracne
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# mutual-information-network-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Augusta's core initial gene regulatory network (GRN) inference from RNA-Seq time-series count tables via pairwise mutual information (MI) with binning D = min(n/5, 10), edge direction/sign from the most significant difference (MSD) between adjacent time points, at O(mn²D²) complexity.

## When to use

Use when you have a raw or normalized RNA-Seq time-series count table (e.g., DREAM 4 100 genes x 21 samples, B. subtilis 3997 genes x 7 samples) and need a first-pass directed, signed GRN before TFBM/database refinement.

## When NOT to use

- Data are single-cell RNA-Seq — the article notes alternative tools (Inferelator 3.0, GRNBoost2, Arboreto) are typically designed for scRNA-Seq, while Augusta targets bulk time-series count tables
- You need a fully refined network with verified interactions — MI-only inference must be followed by de novo TFBM discovery (MEME Suite) and curated database search (OmniPath, Signor, SignaLink, TRRUST)
- Only a single time point or non-time-series expression data are available — edge orientation/sign relies on the most significant difference among adjacent pairs of time points

## Inputs

- RNA-Seq time-series count table (.csv) of genes x time points, raw or normalized (CPM/RPKM/TPM)
- Optionally an annotated genome in GenBank format if proceeding to refinement steps

## Outputs

- Initial GRN adjacency matrix in .csv format (initial_grn_adjacency_matrix.csv) with directed, signed (positive/negative) TF-TG edges

## How to apply

First normalize raw counts with one of Augusta's methods (CPM, RPKM, or TPM) if a raw count table is supplied. Discretize continuous expression values into D = min(n/5, 10) bins, then compute gene-wise entropy H(X), joint entropy H(X,Y), and pairwise mutual information using natural logarithm (nats) for every gene pair to propose candidate edges. Orient each TF-TG edge and assign its regulation sign (activation/repression) by identifying the most significant difference (MSD) among all adjacent pairs of time points (Eq. 5), evaluating the transcriptional time lag of each pair. Export the resulting initial GRN as an adjacency matrix CSV (e.g., initial_grn_adjacency_matrix.csv); on DREAM 4 this MI-only variant yields sensitivity 0.38, specificity 0.54, accuracy 0.54 in ~83 s, and runtime scales as O(mn²D²) with m time points and n genes.

## Related tools

- **Augusta** (Python package performing the MI-based initial GRN inference from the RNA-Seq count table and exporting the adjacency matrix) — https://github.com/JanaMus/Augusta
- **Python** (Implementation language of the Augusta package (version 3.7 or 3.8))

## Examples

```
Augusta.RNASeq_to_GRN(count_table_input='dream4_ct.csv', promoter_length=1000, genbank_file_input='ecoli_bw25113.gb', normalization_type='TPM', motifs_max_time=600)
```

## Evaluation signals

- On the DREAM 4 In Silico Network Challenge dataset (100 genes, 21 samples), the MI-only network should give sensitivity ~0.38, specificity ~0.54, accuracy ~0.54 in ~83.07 s when scored against the gold standard
- Runtime should scale approximately as O(m*n²*D²): benchmark on progressively larger gene counts and fit timing to verify the claimed complexity (timing_report.csv)
- Number of bins used equals D = min(n/5, 10), where n is the number of genes in the count table
- Every predicted edge carries a direction and a sign derived from the MSD among adjacent time points (transcriptional time lag evaluation)
- Output is a valid adjacency matrix CSV whose gene set matches the input count table's genes

## Limitations

- Autoregulation inference remains a significant challenge not only in the field of network inference
- Inferred networks should be considered approximations, not perfectly accurate representations of biological systems regulations
- Refinement of an initial GRN may be computationally very demanding, and refinement steps cannot be performed without additional knowledge of a genome (e.g., GenBank annotation)
- Processing multiple RNA-Seq experiments in a single computation and parallelization to reduce computational time are not yet supported

## Evidence

- [results] On the DREAM 4 dataset, Augusta without refinement (MI only) achieved sensitivity 0.38, specificity 0.54, accuracy 0.54 in 83.07 s: [evidence span withheld — anchor status: not_found]
- [readme] Augusta is a Python package (3.7 or 3.8) installed via pip and invoked as Augusta.RNASeq_to_GRN on the count table: [evidence span withheld — anchor status: external_doc]
