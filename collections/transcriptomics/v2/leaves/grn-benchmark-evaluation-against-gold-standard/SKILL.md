---
name: grn-benchmark-evaluation-against-gold-standard
description: Use when an inferred gene regulatory network (e.g., from Augusta on DREAM
  4 or time-series RNA-Seq) needs quantitative evaluation against a reference edge
  set, using sensitivity/specificity/accuracy metrics.
when_to_use_negative:
- The gold standard includes signed or directed edges and you want signed/direction-aware
  evaluation without the simplification step (dropping interaction types and negatives)
  used here
- You need autoregulation assessment — the article notes 'inferring autoregulations
  remains a significant challenge'
- The network is a refined Augusta GRN with signed interactions destined for Boolean
  model construction rather than benchmarking
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0204
tools:
- name: Augusta
  role: Infer the initial GRN via mutual information and MSD-based edge orientation
    for each normalization variant prior to gold-standard comparison
  repo: https://github.com/JanaMus/Augusta
- name: Python
  role: Implement the confusion-matrix computation and sensitivity/specificity/accuracy
    metric derivation and CSV export
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:9e605c656d9244a36abe863a850a00d13e7718d8495122148ba162d8b6dddd7b
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
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# grn-benchmark-evaluation-against-gold-standard

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a GRN inference pipeline (e.g., Augusta's MI-based initial GRN with MSD edge orientation) against an undirected gold-standard network by computing sensitivity, specificity, and accuracy from a confusion matrix of predicted versus true edges.

## When to use

Use when an inferred gene regulatory network (e.g., from Augusta on DREAM 4 or time-series RNA-Seq) needs quantitative evaluation against a reference edge set, using sensitivity/specificity/accuracy metrics.

## When NOT to use

- The gold standard includes signed or directed edges and you want signed/direction-aware evaluation without the simplification step (dropping interaction types and negatives) used here
- You need autoregulation assessment — the article notes 'inferring autoregulations remains a significant challenge'
- The network is a refined Augusta GRN with signed interactions destined for Boolean model construction rather than benchmarking

## Inputs

- Inferred GRN edge list (e.g., Augusta output CSV) per pipeline variant
- Gold-standard network (undirected typed, e.g., DREAM 4 In Silico Network Challenge, 100 genes)
- Count table (e.g., DREAM 4 time-series count table, 100 genes in 21 samples) and GenBank annotation file (e.g., Escherichia coli BW25113) if inference is re-run

## Outputs

- Confusion matrix of predicted versus true edges per variant
- Metrics CSV (e.g., normalization_benchmark_metrics.csv) with sensitivity, specificity, and accuracy rows per normalization method

## How to apply

Infer the GRN for each pipeline variant (e.g., one run per normalization setting: none, CPM, RPKM, TPM), keeping all other parameters at defaults. Augusta's initial GRN uses mutual information with continuous expression values binned into D = min(n/5, 10) bins, orienting edges and signs by the most significant difference (MSD) between adjacent time points. Before comparison, simplify each inferred network: drop interaction types and convert all negative edges to positive ones to match the undirected typed gold standard (e.g., DREAM 4 In Silico Network Challenge, 100 genes). Compute a confusion matrix of predicted versus true edges for each variant and derive sensitivity (true-positive proportion), specificity (true-negative proportion), and accuracy (overall correct-prediction proportion). Write all metrics per variant to a CSV (e.g., normalization_benchmark_metrics.csv) and compare against paper-scale values such as sensitivity 0.06, specificity 0.91, accuracy 0.89 with refinement.

## Related tools

- **Augusta** (Infer the initial GRN via mutual information and MSD-based edge orientation for each normalization variant prior to gold-standard comparison) — https://github.com/JanaMus/Augusta
- **Python** (Implement the confusion-matrix computation and sensitivity/specificity/accuracy metric derivation and CSV export)

## Examples

```
Augusta.RNASeq_to_GRN(count_table_input='dream4_ct.csv', promoter_length=My_number, genbank_file_input='ecoli_BW25113.gb', normalization_type='TPM', motifs_max_time=My_seconds)
```

## Evaluation signals

- The script runs to completion on all four normalization variants (none, CPM, RPKM, TPM)
- The metrics CSV contains sensitivity, specificity, and accuracy rows for each of the four normalization methods
- DREAM 4 metrics are of the same order as the paper's reported values (e.g., sensitivity 0.06, specificity 0.91, accuracy 0.89 with refinement; sensitivity 0.38, specificity 0.54, accuracy 0.54 without refinement)
- Inferred networks were simplified (interaction types dropped, negative edges converted to positive) before comparison with the undirected gold standard
- MI binning used D = min(n/5, 10) bins as in the Augusta pipeline

## Limitations

- GRN refinement is computationally very demanding (DREAM 4 refinement took 714.18 s vs. 83.07 s for MI-only)
- Inferred networks are approximations, not perfectly accurate representations of biological system regulation
- Augusta does not yet support processing multiple RNA-Seq experiments in a single computation or parallelization to reduce runtime

## Evidence

- [results] DREAM 4 metrics with refinement: [evidence span withheld — anchor status: not_found]
- [methods] count table normalization: [evidence span withheld — anchor status: not_found]
- [readme] Augusta invocation from README: [evidence span withheld — anchor status: external_doc]
