---
name: execution-time-profiling
description: Use when comparing computational scalability of network inference pipelines
  on genome-wide RNA-Seq data and you need per-tool and per-pipeline-step runtime
  metrics to populate a benchmark results table.
when_to_use_negative:
- When the goal is predictive performance alone (sensitivity/specificity/accuracy)
  and runtime is irrelevant to the comparison
- When tools are run on different hardware or with non-default parameters, making
  runtime comparisons unfair and non-reproducible
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: Augusta
  role: Tool being profiled; its individual pipeline steps (MI-based GRN inference,
    TFBM discovery, database search) are timed separately
  repo: https://github.com/JanaMus/Augusta
- name: GENIE3
  role: Baseline GRN inference tool whose default-parameter runtime is recorded for
    comparison on the same dataset and hardware
- name: TDAracne
  role: Baseline tool (number of bins set to 10) whose execution time is recorded
    in the benchmark
- name: KBoost
  role: Baseline tool run with default parameters whose execution time is recorded
    in the benchmark
- name: Python
  role: Implementation language; benchmark metrics and runtimes are written to CSV
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:94b5d4f8593da1c4a4c6c87ff8be3b859eb940abaa51228c781d1fb795a986d3
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# execution-time-profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Profiles wall-clock execution time of GRN inference tools (Augusta with/without refinement, GENIE3, TDAracne, KBoost) on the same dataset and hardware, including per-step timing for Augusta's MI inference, TFBM discovery, and database-search refinement, to build a fair runtime benchmark table.

## When to use

Use when comparing computational scalability of network inference pipelines on genome-wide RNA-Seq data and you need per-tool and per-pipeline-step runtime metrics to populate a benchmark results table.

## When NOT to use

- When the goal is predictive performance alone (sensitivity/specificity/accuracy) and runtime is irrelevant to the comparison
- When tools are run on different hardware or with non-default parameters, making runtime comparisons unfair and non-reproducible

## Inputs

- RPKM-normalized Bacillus subtilis 168 RNA-Seq time-series dataset (3997 genes, 7 samples)
- Predicted networks from each tool (Augusta with/without refinement, GENIE3, TDAracne, KBoost)
- Per-step timing logs from Augusta pipeline runs (MI inference, TFBM discovery via MEME Suite, database search)

## Outputs

- bsubtilis_benchmark_results.csv with execution time [s], sensitivity, specificity, and accuracy per tool
- Per-pipeline-step execution times for Augusta (initial GRN, refinement steps)

## How to apply

Run each tool (Augusta without refinement using mutual information with bins D = min(n/5, 10), Augusta with two-step TFBM/database refinement, GENIE3, TDAracne with 10 bins, and KBoost with defaults) on the same input — e.g. the RPKM-normalized B. subtilis 168 dataset of 3997 genes in 7 samples — on identical hardware. Record total wall-clock execution time for every tool, and for Augusta additionally record time per individual pipeline step (initial MI-based GRN inference, de novo TFBM discovery on regulons of at least 5 genes using 1000 bp promoter sequences, curated database search). Write all timing values alongside sensitivity, specificity, and accuracy metrics to a single results CSV so runtime can be compared against predictive performance. Correctness is judged by the recorded runtimes matching the paper's benchmark table for the same evaluation and by timing consistency across tools run under identical conditions.

## Related tools

- **Augusta** (Tool being profiled; its individual pipeline steps (MI-based GRN inference, TFBM discovery, database search) are timed separately) — https://github.com/JanaMus/Augusta
- **GENIE3** (Baseline GRN inference tool whose default-parameter runtime is recorded for comparison on the same dataset and hardware)
- **TDAracne** (Baseline tool (number of bins set to 10) whose execution time is recorded in the benchmark)
- **KBoost** (Baseline tool run with default parameters whose execution time is recorded in the benchmark)
- **Python** (Implementation language; benchmark metrics and runtimes are written to CSV)

## Examples

```
Augusta.RNASeq_to_GRN(count_table_input='bsubtilis_rpkm.csv', genbank_file_input='AL009126.3.gb', normalization_type='RPKM', motifs_max_time=My_seconds)
```

## Evaluation signals

- Recorded execution times for each tool match the paper's B. subtilis whole-genome benchmark results table
- Every tool was run on the identical input dataset (3997 genes, 7 samples) and hardware before timing
- Augusta per-step timings sum to (or are consistent with) Augusta's total reported runtime
- bsubtilis_benchmark_results.csv contains time, sensitivity, specificity, and accuracy columns for all compared tools

## Limitations

- GRN refinement (TFBM discovery) is computationally very demanding, dominating Augusta's runtime
- Parallelization to reduce computational time is not yet implemented in Augusta
- Runtimes are only comparable when all tools run on the same dataset and hardware

## Evidence

- [full_text] Record execution time for each tool and, for Augusta, per individual pipeline step.: [evidence span withheld — anchor status: not_found]
- [results] Time [s] 714.18 ... Sensitivity 0.06 ... Specificity 0.91 ... Accuracy 0.89: [evidence span withheld — anchor status: not_found]
