---
name: grn-tool-benchmark-comparison
description: Use when evaluating a new or modified GRN inference pipeline against
  established tools on a dataset with a known gold-standard network, to quantify accuracy
  and runtime trade-offs.
when_to_use_negative:
- The dataset has no gold-standard network, so confusion-matrix metrics cannot be
  computed and the comparison would be subjective
- The candidate pipeline is designed for single-cell RNA-Seq data and the comparison
  dataset is bulk time-series (tools like GRNBoost2 and Arboreto are scoped to single-cell
  in this context)
- You only need to run one tool rather than a controlled same-input, same-metric comparison
  across tools
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: Augusta
  role: Candidate pipeline under benchmark; run full (MI + TFBM + DB refinement) and
    MI-only variants
  repo: https://github.com/JanaMus/Augusta
- name: GENIE3
  role: Baseline GRN inference competitor run with default parameters
- name: TDAracne
  role: Baseline MI-based competitor; number-of-bins parameter set to 10 for percentile/rank
    normalization
- name: KBoost
  role: Baseline competitor run with default parameters
- name: MEME Suite
  role: Performs de novo TFBM discovery within the Augusta full-pipeline variant being
    benchmarked
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:1597f8274bafe79342995803e3ee31e1281c304266e75d72d9ca7fd3d8c9810d
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
  - TDAracne
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# grn-tool-benchmark-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a GRN inference pipeline (e.g., Augusta full pipeline vs. MI-only) head-to-head with established tools (GENIE3, TDAracne, KBoost) on a shared dataset with a gold-standard network, comparing sensitivity, specificity, accuracy, and execution time.

## When to use

Use when evaluating a new or modified GRN inference pipeline against established tools on a dataset with a known gold-standard network, to quantify accuracy and runtime trade-offs.

## When NOT to use

- The dataset has no gold-standard network, so confusion-matrix metrics cannot be computed and the comparison would be subjective
- The candidate pipeline is designed for single-cell RNA-Seq data and the comparison dataset is bulk time-series (tools like GRNBoost2 and Arboreto are scoped to single-cell in this context)
- You only need to run one tool rather than a controlled same-input, same-metric comparison across tools

## Inputs

- DREAM 4 In Silico Network Challenge time-series dataset (first experiment, 100 genes, 21 samples)
- DREAM 4 gold standard network (directed edges only)
- Annotated Escherichia coli BW25113 GenBank file (for Augusta refinement)
- Count table consumed by all benchmarked tools

## Outputs

- dream4_benchmark_metrics.csv with sensitivity, specificity, accuracy, and execution time per tool (Augusta full, Augusta no refinement, GENIE3, TDAracne, KBoost)
- Per-tool confusion matrices against the gold standard
- Benchmark comparison table mirroring the published DREAM 4 results

## How to apply

Run every tool on the identical input — here the DREAM 4 In Silico Network Challenge time-series dataset (100 genes, 21 samples for the first experiment) — with default parameters for competitors (GENIE3, TDAracne with number-of-bins set to 10 for percentile/rank normalization, KBoost) and the candidate pipeline both with and without its refinement stage (Augusta full: MI-based inference with bins D = min(n/5, 10), edge direction/sign from the most significant difference between adjacent time points, de novo TFBM discovery via MEME Suite on 1000 bp upstream promoters, and curated database search against OmniPath, Signor, SignaLink, TRRUST). Harmonize output formats before scoring: simplify Augusta networks by ignoring interaction types and converting negative edges to positive to match the gold standard's directed-edge-only format. Compute a confusion matrix of each predicted network against the gold standard to derive sensitivity, specificity, and accuracy, and record execution time per tool. Save all metrics (e.g., to dream4_benchmark_metrics.csv) and compare against reported reference values before drawing conclusions.

## Related tools

- **Augusta** (Candidate pipeline under benchmark; run full (MI + TFBM + DB refinement) and MI-only variants) — https://github.com/JanaMus/Augusta
- **GENIE3** (Baseline GRN inference competitor run with default parameters)
- **TDAracne** (Baseline MI-based competitor; number-of-bins parameter set to 10 for percentile/rank normalization)
- **KBoost** (Baseline competitor run with default parameters)
- **MEME Suite** (Performs de novo TFBM discovery within the Augusta full-pipeline variant being benchmarked)

## Examples

```
Augusta.RNASeq_to_GRN(count_table_input='MyCT_file.csv', promoter_length=1000, genbank_file_input='MyGB_file.gb', normalization_type='My_string', motifs_max_time=My_seconds)
```

## Evaluation signals

- Augusta full pipeline reproduces reported DREAM 4 metrics: sensitivity 0.06, specificity 0.91, accuracy 0.89 in 714.18 s
- Augusta no refinement (MI only) reproduces reported metrics: sensitivity 0.38, specificity 0.54, accuracy 0.54 in 83.07 s
- All tools are scored on the same 100-gene, 21-sample dataset against the same gold standard after format harmonization (negative edges converted to positive, interaction types ignored)
- Metrics table contains all five configurations (Augusta full, Augusta no refinement, GENIE3, TDAracne, KBoost) with all four columns populated
- Predicted networks match the gold standard's directed-edge-only format before confusion-matrix computation

## Limitations

- Augusta's refinement stage is computationally very demanding (714.18 s vs. 83.07 s for MI-only on this benchmark)
- Inferred networks are approximations, not perfectly accurate representations of biological regulatory systems
- Refinement steps require additional genome knowledge (GenBank annotation) and cannot be performed without it
- Results depend on format harmonization choices — ignoring interaction types and converting negative edges to positive affects the confusion matrix

## Evidence

- [results] DREAM4 benchmark Augusta vs competitors: [evidence span withheld — anchor status: not_found]
- [results] MI-only baseline metrics: [evidence span withheld — anchor status: not_found]
- [methods] Format harmonization and metrics from source card workflow: [evidence span withheld — anchor status: not_found]
- [methods] Baseline tool parameters: [evidence span withheld — anchor status: not_found]
