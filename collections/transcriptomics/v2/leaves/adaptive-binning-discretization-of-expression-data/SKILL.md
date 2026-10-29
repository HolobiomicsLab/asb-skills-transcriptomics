---
name: adaptive-binning-discretization-of-expression-data
description: Use when a raw or normalized time-series RNA-Seq count table (e.g. DREAM
  4 In Silico, ~21 samples, 100 genes) must be binned into a limited number of states
  to make pairwise mutual-information GRN inference tractable.
when_to_use_negative:
- 'The goal is a refined GRN or Boolean Network: discretization/MI alone only produces
  the initial (MI-only) network, which the paper shows has low sensitivity (0.38)
  compared to the refined model (accuracy 0.89)'
- The input is single-cell RNA-Seq data; the paper notes alternative tools (Inferelator
  3.0, GRNBoost2, Arboreto) are typically designed for scRNA-Seq, while Augusta targets
  bulk time-series count tables with few time points
- 'Only one or two time points are available: the edge-orientation step relies on
  the most significant difference (MSD) among adjacent pairs of time points and transcriptional
  time lag, which needs a multi-point time series'
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Augusta
  role: Python package whose initial-GRN module performs the count-table normalization,
    adaptive binning, and pairwise mutual-information computation on the discretized
    data
  repo: https://github.com/JanaMus/Augusta
- name: Python
  role: Implementation language of Augusta; used to drive the GRN inference and the
    runtime benchmarking of the binned MI computation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:28d9318019173ac8418b16c7be386d6c5e33763ec5c739cbb5c3cbf5fcb43f92
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
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# adaptive-binning-discretization-of-expression-data

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Discretize a normalized RNA-Seq time-series count table into D bins so that pairwise mutual information (MI) can be computed for initial GRN inference in Augusta, keeping the runtime within the paper's O(mn^2 D^2) bound where m is time points and n is genes.

## When to use

Use when a raw or normalized time-series RNA-Seq count table (e.g. DREAM 4 In Silico, ~21 samples, 100 genes) must be binned into a limited number of states to make pairwise mutual-information GRN inference tractable.

## When NOT to use

- The goal is a refined GRN or Boolean Network: discretization/MI alone only produces the initial (MI-only) network, which the paper shows has low sensitivity (0.38) compared to the refined model (accuracy 0.89)
- The input is single-cell RNA-Seq data; the paper notes alternative tools (Inferelator 3.0, GRNBoost2, Arboreto) are typically designed for scRNA-Seq, while Augusta targets bulk time-series count tables with few time points
- Only one or two time points are available: the edge-orientation step relies on the most significant difference (MSD) among adjacent pairs of time points and transcriptional time lag, which needs a multi-point time series

## Inputs

- Normalized RNA-Seq time-series count table in CSV format (e.g. DREAM 4 In Silico Network Challenge first experiment, 100 genes, 21 samples)
- Normalization type string selecting CPM, RPKM, or TPM
- Number of genes n (used to set the number of bins D = min(n/5, 10))

## Outputs

- Discretized (binned) expression matrix with D states per gene, feeding pairwise MI computation
- Initial GRN adjacency matrix exported as initial_grn_adjacency_matrix.csv
- Timing benchmark file (timing_report.csv) fitting runtime to O(mn^2 D^2)

## How to apply

First normalize the raw count table (Augusta supports CPM, RPKM, or TPM) since normalization of raw counts is the crucial initial step. Then, before computing gene-wise entropy H(X), joint entropy H(X,Y), and pairwise MI in nats for every gene pair, bin the continuous expression values with the number of bins set to D = min(n/5, 10), where n is the number of genes in the count table. The small, gene-count-adaptive D is what keeps the core algorithm within its stated O(mn^2 D^2) complexity, where m is the number of time points (typically units to tens). Discretization feeds directly into the MI-based candidate edges, whose orientation and sign are then assigned using the transcriptional time lag identified via the most significant difference (MSD) among adjacent pairs of time points. Validate by benchmarking runtime on progressively larger gene counts and fitting it to the claimed O(m*n^2*D^2) scaling, and sanity-check the resulting no-refinement (MI-only) network against the paper's DREAM 4 metrics (sensitivity 0.38, specificity 0.54, accuracy 0.54, ~83.07 s).

## Related tools

- **Augusta** (Python package whose initial-GRN module performs the count-table normalization, adaptive binning, and pairwise mutual-information computation on the discretized data) — https://github.com/JanaMus/Augusta
- **Python** (Implementation language of Augusta; used to drive the GRN inference and the runtime benchmarking of the binned MI computation)

## Examples

```
Augusta.RNASeq_to_GRN(count_table_input='MyCT_file.csv', promoter_length=My_number, genbank_file_input='MyGB_file.gb', normalization_type='My_string', motifs_max_time=My_seconds)
```

## Evaluation signals

- Runtime on progressively scaled gene counts fits the claimed O(m*n^2*D^2) complexity, with m in the order of units to tens of time points
- MI-only (no refinement) DREAM 4 result matches the paper's order of magnitude (~83.07 s) with nonzero edge recovery (sensitivity 0.38, specificity 0.54, accuracy 0.54)
- Number of bins respects D = min(n/5, 10) for the given gene count n
- Entropies and MI are computed with the natural logarithm (nats) for every gene pair
- Output adjacency matrix (initial_grn_adjacency_matrix.csv) contains directed, sign-assigned TF-TG edges derived via the MSD time-lag criterion

## Limitations

- GRN refinement (the step after initial MI-based inference) may be computationally very demanding and requires additional knowledge of a genome (e.g. GenBank file for TFBM prediction)
- Processing multiple RNA-Seq experiments in a single computation is not yet supported; the discretized MI inference assumes a single time-series count table
- Inferred networks are approximations, not perfectly accurate representations of biological regulation, and the MI-only stage has notably low sensitivity on DREAM 4 (0.38)

## Evidence

- [methods] initial GRN inference via mutual information calculation: "an initial Gene Regulatory Network (initial GRN) is inferred using a mutual information (MI) calculation"
- [methods] MSD time-lag edge orientation: "the evaluation is performed by identifying the most significant difference (MSD) among all adjacent pairs of time points in the measured time series"
- [methods] Normalization precedes discretization: [evidence span withheld — anchor status: not_found]
- [results] MI-only DREAM 4 benchmark metrics: [evidence span withheld — anchor status: not_found]
- [readme] Augusta GRN inference entry point: [evidence span withheld — anchor status: external_doc]
