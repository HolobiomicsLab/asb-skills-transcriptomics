---
name: entropy-estimation-from-gene-expression
description: Use when starting from a normalized gene-expression count table (e.g.
  DREAM 4 time-series, units-to-tens of time points) and you need to rank gene-gene
  relationships by mutual information to seed GRN inference.
when_to_use_negative:
- 'You need the refined, high-confidence GRN: MI-only inference yields low sensitivity
  (0.38) and requires downstream refinement via TFBM prediction and curated database
  interactions'
- The organism's genome/GenBank file is unavailable and you require refinement steps,
  which cannot be performed without additional genome knowledge
- You must distinguish transcription factor from target gene labels from single-cell
  data without a time series — the MSD-based time-lag orientation depends on adjacent
  time points
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2258
tools:
- name: Augusta
  role: Open-source Python package implementing the entropy/MI-based initial GRN inference
    from expression data
  repo: https://github.com/JanaMus/Augusta
- name: Python
  role: Language in which Augusta is implemented and through which the RNASeq_to_GRN
    entry point is invoked
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:1e50554d9963feda5daa643343deabee7576e61280a5b6541e40b654eecd6817
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# entropy-estimation-from-gene-expression

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute gene-wise entropy, joint entropy, and pairwise mutual information over a discretized RNA-Seq count table to propose candidate regulatory edges for an initial GRN, as implemented in Augusta's first inference stage.

## When to use

Use when starting from a normalized gene-expression count table (e.g. DREAM 4 time-series, units-to-tens of time points) and you need to rank gene-gene relationships by mutual information to seed GRN inference.

## When NOT to use

- You need the refined, high-confidence GRN: MI-only inference yields low sensitivity (0.38) and requires downstream refinement via TFBM prediction and curated database interactions
- The organism's genome/GenBank file is unavailable and you require refinement steps, which cannot be performed without additional genome knowledge
- You must distinguish transcription factor from target gene labels from single-cell data without a time series — the MSD-based time-lag orientation depends on adjacent time points

## Inputs

- Normalized or raw RNA-Seq count table in .csv format (genes x time points), e.g. DREAM 4 In Silico Network Challenge time-series with 100 genes and 21 samples
- Choice of normalization method: CPM, RPKM, or TPM (if starting from raw counts)

## Outputs

- Pairwise mutual information values (in nats) for all gene pairs
- Initial GRN as a directed, signed adjacency matrix (initial_grn_adjacency_matrix.csv)
- Timing report (timing_report.csv) from complexity benchmarking

## How to apply

Load the count table (e.g. DREAM 4 In Silico Network Challenge, 100 genes, 21 samples) and first normalize raw counts (CPM, RPKM, or TPM) to remove systematic bias. Discretize continuous expression values by binning with D = min(n/5, 10), where n is the number of genes. For every gene pair, compute gene-wise entropy H(X), joint entropy H(X,Y), and pairwise mutual information using the natural logarithm (nats) to propose candidate edges; the algorithm's complexity is O(mn^2 D^2) where m is the number of time points. Edges are then oriented TF-TG with a positive/negative sign using the transcriptional time lag identified via the most significant difference (MSD) among adjacent pairs of time points (Eq. 5). Export the result as an adjacency matrix (initial_grn_adjacency_matrix.csv) and benchmark runtime on progressively scaled inputs to confirm the stated complexity.

## Related tools

- **Augusta** (Open-source Python package implementing the entropy/MI-based initial GRN inference from expression data) — https://github.com/JanaMus/Augusta
- **Python** (Language in which Augusta is implemented and through which the RNASeq_to_GRN entry point is invoked)

## Examples

```
Augusta.RNASeq_to_GRN(count_table_input='MyCT_file.csv', promoter_length=My_number, genbank_file_input='MyGB_file.gb', normalization_type='My_string', motifs_max_time=My_seconds)
```

## Evaluation signals

- Order-of-magnitude runtime matches the reported MI-only DREAM 4 benchmark (~83.07 s) when scaling inputs
- Resulting adjacency matrix is nonzero-edged and its edge recovery is consistent with the paper's MI-only metrics: sensitivity 0.38, specificity 0.54, accuracy 0.54
- Number of discretization bins equals min(n/5, 10) for the given gene count n
- Empirical runtime on progressively scaled inputs fits the claimed O(m*n^2*D^2) complexity
- Mutual information values are non-negative and computed in nats (natural logarithm), consistent with I(X,Y) = H(X) + H(Y) - H(X,Y)

## Limitations

- MI-only initial inference has modest accuracy (sensitivity 0.38, specificity 0.54 on DREAM 4); refinement via TFBM prediction and database interactions is computationally very demanding
- m (time points) is expected in the order of units to tens at maximum; autoregulations remain hard to infer
- Parallelization is not yet implemented, limiting runtime reduction on large gene counts

## Evidence

- [results] DREAM 4 MI-only benchmark metrics: [evidence span withheld — anchor status: not_found]
