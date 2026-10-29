---
name: transcriptional-time-lag-edge-orientation
description: Use when you have a time-series RNA-Seq count table (e.g., DREAM 4 100-gene/21-sample,
  C. beijerinckii 6 time points) and need to assign direction (TF→TG) and sign (activation/repression)
  to mutual-information-derived edges.
when_to_use_negative:
- Data are not a measured time series (no adjacent time points exist), so the most-significant-difference
  time-lag evaluation cannot be performed
- You only need an undirected, unsigned association network — MI inference alone (e.g.,
  Augusta without refinement, which scored sensitivity 0.38, specificity 0.54 on DREAM
  4) suffices without edge orientation
- Processing multiple RNA-Seq experiments measured under various environmental conditions,
  which Augusta does not yet support in a single computation
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: Augusta
  role: Python package whose MI-based GRN inference applies the MSD time-lag evaluation
    to orient and sign TF-TG edges and re-types newly introduced edges during refinement
  repo: https://github.com/JanaMus/Augusta
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:292f7f11bb967559fdb51dfc953b92eab46a3d934f548b9202701489af51ca46
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
  - Python
  - TDAracne
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# transcriptional-time-lag-edge-orientation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Orients and signs edges of an MI-inferred gene regulatory network by identifying the most significant difference (MSD) among adjacent pairs of time points in a time-series RNA-Seq count table, which evaluates the transcriptional time lag between each TF-TG pair.

## When to use

Use when you have a time-series RNA-Seq count table (e.g., DREAM 4 100-gene/21-sample, C. beijerinckii 6 time points) and need to assign direction (TF→TG) and sign (activation/repression) to mutual-information-derived edges.

## When NOT to use

- Data are not a measured time series (no adjacent time points exist), so the most-significant-difference time-lag evaluation cannot be performed
- You only need an undirected, unsigned association network — MI inference alone (e.g., Augusta without refinement, which scored sensitivity 0.38, specificity 0.54 on DREAM 4) suffices without edge orientation
- Processing multiple RNA-Seq experiments measured under various environmental conditions, which Augusta does not yet support in a single computation

## Inputs

- Time-series RNA-Seq count table (e.g., CSV; DREAM 4 In Silico Network Challenge time-series dataset, 100 genes in 21 samples; or C. beijerinckii NRRL B-598, 5442 genes over six time points)
- Initial GRN (TF-TG pairs) inferred via mutual information with D = min(n/5, 10) bins
- Optionally, refined GRN edges from de novo TFBM discovery or curated database search that require sign/direction typing

## Outputs

- Directed, signed adjacency matrix of the GRN (CSV) where each edge has a TF→TG direction and an activation/repression sign derived from the most significant difference between adjacent time points

## How to apply

After binning continuous expression values into D = min(n/5, 10) bins and computing pairwise mutual information to infer the initial GRN, evaluate each TF-TG pair by identifying the most significant difference (MSD) among all adjacent pairs of time points in the measured time series. The time point pair with the MSD captures the transcriptional time lag between the regulator and its target; use it to assign the edge direction and the regulation sign (activation or repression). The same time-lag evaluation is re-applied during GRN refinement to type newly introduced edges, e.g., after de novo TFBM discovery with the MEME Suite on 1000 bp upstream promoters (regulons of five or more genes) and after adding curated database interactions (OmniPath, Signor, SignaLink, TRRUST), where database edge types are preferred on mismatch.

## Related tools

- **Augusta** (Python package whose MI-based GRN inference applies the MSD time-lag evaluation to orient and sign TF-TG edges and re-types newly introduced edges during refinement) — https://github.com/JanaMus/Augusta

## Examples

```
Augusta.RNASeq_to_GRN(count_table_input='MyCT_file.csv', promoter_length=1000, genbank_file_input='MyGB_file.gb', normalization_type='My_string', motifs_max_time=My_seconds)
```

## Evaluation signals

- Every edge in the exported adjacency matrix has an assigned direction and sign (no unsigned or undirected edges remain)
- Simplifying the signed network (dropping interaction types, converting negative edges to positive) and scoring against the DREAM 4 gold standard yields sensitivity/specificity/accuracy of the same order as reported (full pipeline: sensitivity 0.06, specificity 0.91, accuracy 0.89; MI only: 0.38/0.54/0.54)
- Reproducing the C. beijerinckii run yields the reported network sizes (initial: 4684 TFs / 4529 TGs; refined: 2864 TFs / 1880 TGs)
- Binning parameter D equals min(n/5, 10) for the input sample count n (e.g., D = 10 for 21 DREAM 4 samples)

## Limitations

- Inferred networks are approximations and not perfectly accurate representations of biological system regulation; autoregulation inference remains a significant challenge
- Refinement steps require additional knowledge of the genome (annotated GenBank file, promoter sequences) and are computationally very demanding, and parallelization to reduce runtime is not yet implemented
- Multiple RNA-Seq experiments in a single computation are not yet supported, limiting time-lag assessment to a single time-series experiment

## Evidence

- [other] MSD assigns direction and sign: [evidence span withheld — anchor status: not_found]
- [other] Binning parameter: [evidence span withheld — anchor status: not_found]
- [other] Re-application during refinement: [evidence span withheld — anchor status: not_found]
- [results] DREAM 4 benchmark metrics: [evidence span withheld — anchor status: not_found]
