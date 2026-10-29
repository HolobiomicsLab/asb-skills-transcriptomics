---
name: rnaseq-time-series-network-benchmarking
description: Use when comparing gene regulatory network inference methods on a time-series
  RNA-Seq dataset with a known gold-standard GRN, requiring sensitivity, specificity,
  accuracy, and runtime metrics per tool.
when_to_use_negative:
- The goal is single-cell RNA-Seq network inference — tools like Inferelator 3.0,
  GRNBoost2, and Arboreto are designed for scRNA-Seq, and this benchmark targets bulk
  whole-genome time-series data
- No gold standard or annotated genome is available — refinement steps require additional
  knowledge of a genome (promoter sequences, GenBank annotation) and validation requires
  known positive/negative interactions
- The task is Boolean Network inference/model simulation rather than comparative GRN
  inference benchmarking
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: Augusta
  role: Primary GRN inference tool under benchmark; run without refinement (MI-only)
    and with full TFBM + database refinement
  repo: https://github.com/JanaMus/Augusta
- name: GENIE3
  role: Baseline GRN inference tool run with default parameters for comparison
- name: TDAracne
  role: Baseline information-theoretic inference tool run with number of bins set
    to 10
- name: KBoost
  role: Baseline GRN inference tool run with default parameters for comparison
- name: MEME Suite
  role: De novo transcription factor binding motif discovery on 1000 bp promoters
    of regulons with at least 5 genes during Augusta refinement
- name: OmniPath
  role: Manually curated interaction database searched in Augusta's refinement step
- name: Signor
  role: Manually curated interaction database searched in Augusta's refinement step
- name: SignaLink
  role: Manually curated interaction database searched in Augusta's refinement step
- name: TRRUST
  role: Manually curated interaction database searched in Augusta's refinement step
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:3ae0ebaa1f85217722ff33c32ab0cd78e631d85ef6c6504a31ddeff35620329f
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
  - TRRUST
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# rnaseq-time-series-network-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark workflow for evaluating GRN inference tools (Augusta with/without TFBM/database refinement, GENIE3, TDAracne, KBoost) on a whole-genome RNA-Seq time-series dataset, scoring predictions against a gold standard with a multiclass (positive/negative/absent) confusion matrix plus runtime measurement.

## When to use

Use when comparing gene regulatory network inference methods on a time-series RNA-Seq dataset with a known gold-standard GRN, requiring sensitivity, specificity, accuracy, and runtime metrics per tool.

## When NOT to use

- The goal is single-cell RNA-Seq network inference — tools like Inferelator 3.0, GRNBoost2, and Arboreto are designed for scRNA-Seq, and this benchmark targets bulk whole-genome time-series data
- No gold standard or annotated genome is available — refinement steps require additional knowledge of a genome (promoter sequences, GenBank annotation) and validation requires known positive/negative interactions
- The task is Boolean Network inference/model simulation rather than comparative GRN inference benchmarking

## Inputs

- RPKM-normalized RNA-Seq time-series count table (e.g. 3997 genes, 7 samples, CSV)
- Annotated genome sequence in GenBank file format (e.g. AL009126.3)
- Gold standard GRN with positive and negative interactions
- Upstream promoter sequences (1000 bp) for TFBM refinement

## Outputs

- Predicted GRNs from each tool (Augusta, GENIE3, TDAracne, KBoost)
- Benchmark metrics table (e.g. bsubtilis_benchmark_results.csv) with execution time, sensitivity, specificity, and accuracy per tool

## How to apply

Start from an RPKM-normalized time-series count table (e.g. Bacillus subtilis 168: 3997 genes x 7 samples), the annotated genome (GenBank ID: AL009126.3), and a gold standard GRN containing positive and negative interactions. Run Augusta without refinement, inferring the initial GRN by mutual information with the number of bins set to D = min(n/5, 10) and edges oriented/signed via the most-significant-difference (MSD) transcriptional time lag; then run Augusta with full two-step refinement: de novo TFBM discovery on regulons of at least 5 genes using 1000 bp upstream promoter sequences (MEME Suite) followed by curated database search (OmniPath, Signor, SignaLink, TRRUST). Run baselines GENIE3, TDAracne (bins=10), and KBoost with default parameters on the same dataset and hardware. Evaluate every predicted network with a 3x3 confusion matrix (multiclass over positive, negative, and absent edges), computing sensitivity and specificity per edge class and averaging them plus overall accuracy, and record execution time per tool and per Augusta pipeline step.

## Related tools

- **Augusta** (Primary GRN inference tool under benchmark; run without refinement (MI-only) and with full TFBM + database refinement) — https://github.com/JanaMus/Augusta
- **GENIE3** (Baseline GRN inference tool run with default parameters for comparison)
- **TDAracne** (Baseline information-theoretic inference tool run with number of bins set to 10)
- **KBoost** (Baseline GRN inference tool run with default parameters for comparison)
- **MEME Suite** (De novo transcription factor binding motif discovery on 1000 bp promoters of regulons with at least 5 genes during Augusta refinement)
- **OmniPath** (Manually curated interaction database searched in Augusta's refinement step)
- **Signor** (Manually curated interaction database searched in Augusta's refinement step)
- **SignaLink** (Manually curated interaction database searched in Augusta's refinement step)
- **TRRUST** (Manually curated interaction database searched in Augusta's refinement step)

## Examples

```
Augusta.RNASeq_to_GRN(count_table_input='bsubtilis_rpkm.csv', promoter_length=1000, genbank_file_input='AL009126.3.gb', normalization_type='RPKM', motifs_max_time=My_seconds)
```

## Evaluation signals

- Reproduced metrics and runtimes in bsubtilis_benchmark_results.csv must match the values reported in the paper's benchmark results table for the B. subtilis whole-genome evaluation
- Augusta without refinement is expected to achieve the best sensitivity and considerably higher accuracy than GENIE3 and KBoost; with refinement, accuracy should increase further
- Confusion matrix must be 3x3, covering positive, negative, and absent edge classes, with sensitivity/specificity computed per class then averaged
- Mutual information binning in Augusta must use D = min(n/5, 10) bins; TDAracne must use 10 bins
- Execution time must be recorded per tool and, for Augusta, per individual pipeline step

## Limitations

- Refinement steps cannot be performed without additional knowledge of a genome (annotated GenBank sequence, promoter regions)
- GRN refinement via de novo TFBM discovery may be computationally very demanding
- Inferred networks are approximations, not perfectly accurate representations of biological systems regulation
- Processing multiple RNA-Seq experiments in a single computation and parallelization to reduce computational time are not yet supported
- Known DREAM 4 metrics show Augusta without refinement has low specificity (0.54) despite high sensitivity (0.38), so tool choice depends on which metric matters most

## Evidence

- [readme] README usage (readme): [evidence span withheld — anchor status: external_doc]
