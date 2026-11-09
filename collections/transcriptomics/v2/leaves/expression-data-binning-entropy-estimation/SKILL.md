---
name: expression-data-binning-entropy-estimation
description: Use when starting from an RNA-Seq time-series count table (e.g., C. beijerinckii
  5442 genes × 6 time points) and you need an initial genome-wide GRN via mutual information
  between gene pairs.
when_to_use_negative:
- The data are single-cell RNA-Seq — the article notes alternative tools (Inferelator
  3.0, GRNBoost2, Arboreto) are designed for scRNA-Seq, whereas Augusta targets bulk
  whole-genome time-series data
- A refined, directional network with activation/repression signs is already required
  as the immediate output — MI binning only yields the initial undirected estimation
  before MSD time-lag typing and TFBM refinement
- Only a Boolean model is needed from an existing GRN — use Augusta.GRN_to_BN instead
  of re-inferring from expression data
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: Augusta
  role: Python package performing count-table normalization, discretization of expression
    values, and pairwise mutual information calculation to infer the initial GRN
  repo: https://github.com/JanaMus/Augusta
- name: Python
  role: Implementation language of the Augusta package used for MI-based initial GRN
    inference
- name: MEME Suite
  role: Used only downstream, via Docker, for de novo TFBM refinement of the MI-inferred
    initial GRN
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:c097d08621fbffb2468498d939f862e73f95768a56a2553a9c0452ca9403863f
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
  - MEME Suite
  - Python
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# expression-data-binning-entropy-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Discretize continuous RNA-Seq expression values into bins and compute pairwise mutual information (MI) between all gene pairs to infer an initial gene regulatory network, as implemented in the Augusta package. This information-theory step converts high-throughput time-series expression data into undirected TF–TG associations prior to edge-direction assignment.

## When to use

Use when starting from an RNA-Seq time-series count table (e.g., C. beijerinckii 5442 genes × 6 time points) and you need an initial genome-wide GRN via mutual information between gene pairs.

## When NOT to use

- The data are single-cell RNA-Seq — the article notes alternative tools (Inferelator 3.0, GRNBoost2, Arboreto) are designed for scRNA-Seq, whereas Augusta targets bulk whole-genome time-series data
- A refined, directional network with activation/repression signs is already required as the immediate output — MI binning only yields the initial undirected estimation before MSD time-lag typing and TFBM refinement
- Only a Boolean model is needed from an existing GRN — use Augusta.GRN_to_BN instead of re-inferring from expression data

## Inputs

- RNA-Seq time-series count table (CSV) of raw or normalized counts, e.g. Clostridium beijerinckii NRRL B-598 dataset with 5442 genes across six time points
- Normalization method choice among CPM, RPKM, or TPM (applied to raw counts)
- Number of time points n per gene (controls the bin count D = min(n/5, 10))

## Outputs

- Initial GRN as an undirected gene–gene adjacency structure based on pairwise mutual information scores
- Binned/discretized expression matrix used for MI estimation

## How to apply

First normalize raw counts with one of Augusta's provided methods (CPM, RPKM, or TPM) to remove systematic experimental bias. Then bin the continuous expression values into D = min(n/5, 10) bins — for the six-time-point C. beijerinckii dataset this yields D = min(6/5, 10) — and compute pairwise mutual information between all gene pairs to infer the initial GRN. This MI-based step produces undirected gene–gene associations; edge direction and regulation sign (activation/repression) are added later via the most-significant-difference (MSD) time-lag evaluation, and refinement via de novo TFBM discovery (MEME Suite, default 1000 bp promoters for regulons with ≥5 genes) follows. For C. beijerinckii this MI step yields an initial GRN of 4684 TFs and 4529 TGs, which the refinement reduces to 2864 TFs and 1880 TGs while improving performance.

## Related tools

- **Augusta** (Python package performing count-table normalization, discretization of expression values, and pairwise mutual information calculation to infer the initial GRN) — https://github.com/JanaMus/Augusta
- **Python** (Implementation language of the Augusta package used for MI-based initial GRN inference)
- **MEME Suite** (Used only downstream, via Docker, for de novo TFBM refinement of the MI-inferred initial GRN)

## Examples

```
import Augusta; Augusta.RNASeq_to_GRN(count_table_input='MyCT_file.csv', promoter_length=1000, genbank_file_input='MyGB_file.gb', normalization_type='TPM', motifs_max_time=My_seconds)
```

## Evaluation signals

- Initial C. beijerinckii GRN counts should match the reported 4684 TFs and 4529 TGs before refinement
- Bin count must follow D = min(n/5, 10); verify the discretization used n = 6 time points for the C. beijerinckii data
- Raw input counts must be normalized (CPM/RPKM/TPM) before binning; skipping normalization is flagged as 'a crucial initial step'
- The initial (MI-only) network is the baseline: refinement must reduce network size (2864 TFs / 1880 TGs) and outperform the initial GRN
- On the DREAM 4 benchmark, the MI-only (no-refinement) variant achieved sensitivity 0.38, specificity 0.54, accuracy 0.54 — a sanity range for MI-only inference quality

## Limitations

- MI-only initial inference has low specificity on benchmarks (0.54 on DREAM 4) compared with the refined network (0.91), so the initial GRN should not be treated as final
- Binning with few time points (e.g., six in C. beijerinckii) limits the number of bins (D = min(n/5, 10)) and hence MI estimation resolution
- Inferring autoregulations remains a significant challenge in network inference
- Processing multiple RNA-Seq experiments from different environmental conditions in a single computation is not yet supported

## Evidence

- [other] Bin the continuous expression values into D = min(n/5, 10) bins and compute pairwise mutual information: [evidence span withheld — anchor status: not_found]
- [results] Augusta no refinement MI-only benchmark metrics: [evidence span withheld — anchor status: not_found]
