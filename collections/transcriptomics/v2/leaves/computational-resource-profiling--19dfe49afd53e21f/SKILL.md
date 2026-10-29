---
name: computational-resource-profiling
description: Use when comparing pipelines for cell type/state classification on the
  same dataset and you must report accuracy together with memory usage and execution
  time, especially where one tool may exhaust RAM (e.g. scVI-tools needing >128GB
  for 22,858 features).
when_to_use_negative:
- You need only qualitative agreement of annotations, not resource-efficiency benchmarking
  on identical hardware
- Datasets are small enough that memory/execution-time differences are not the deciding
  factor
- You need feature/biomarker ranking comparability — several comparison tools (scBalance,
  scVI, devCellPy, CellPLM) do not expose top features
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: scaLR
  role: Pipeline under test; run its data ingestion, feature extraction, training,
    and evaluation pipelines with the memoryprofiler flag to log peak memory and execution
    time
  repo: github.com/infocusp/scaLR
- name: Python
  role: Environment for executing scaLR and the custom comparison scripts for all
    competitor pipelines
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf243
    title: 'scaLR: a low-resource deep neural network-based platform for single cell
      analysis and biomarker discovery'
  source_content_hash: sha256:e248f954bef0f243cc6e1bc7e2e4dcea638541179cfc753c4054fe5c9b2d6f39
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/infocusp/scaLR
  tools:
  - Python
  - scaLR
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# computational-resource-profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a cell-annotation pipeline (e.g. scaLR) against competing Python ML/DNN pipelines on the same scRNA-seq dataset while logging prediction accuracy alongside peak memory usage and wall-clock execution time, to demonstrate low-resource operation (e.g. 12GB GPU / 128GB RAM VM).

## When to use

Use when comparing pipelines for cell type/state classification on the same dataset and you must report accuracy together with memory usage and execution time, especially where one tool may exhaust RAM (e.g. scVI-tools needing >128GB for 22,858 features).

## When NOT to use

- You need only qualitative agreement of annotations, not resource-efficiency benchmarking on identical hardware
- Datasets are small enough that memory/execution-time differences are not the deciding factor
- You need feature/biomarker ranking comparability — several comparison tools (scBalance, scVI, devCellPy, CellPLM) do not expose top features

## Inputs

- PBMCs-BS scRNA-seq dataset (anndata .h5ad, 22,858 features) with cell_type/disease target columns
- Custom comparison scripts from the scaLR comparison GitHub repository for each competitor pipeline
- Single VM with 12GB GPU, 128GB RAM, 8 CPUs shared by all benchmarked pipelines

## Outputs

- Comparison table (e.g. pbmcs_bs_benchmark_comparison.csv) with accuracy, precision, recall, f1-score, memory usage, and execution time per pipeline
- Per-pipeline resource logs (peak memory and wall-clock time), including failure records for pipelines that cannot load the full feature set
- Test-set prediction metrics for every pipeline

## How to apply

Run scaLR's end-to-end pipeline (scalr data_ingestion_pipeline with subset-wise normalization and backed-mode reading, feature_extraction_pipeline, model_training_pipeline, eval_and_analysis_pipeline) on the fixed hardware (12GB GPU / 128GB RAM / 8-CPU VM), enabling scaLR's memory-profiler logging so peak memory is captured per stage. Run each compared pipeline (scVI-tools, SingleCellNet, CellTypist variants, ACTINN, devCellPy, scBalance, cellPLM) via the custom scripts in the scaLR comparison GitHub repository on the identical VM, logging the same resource metrics and explicitly recording cases where a tool cannot process the full feature set (scVI-tools failed on all 22,858 features). Evaluate all predictions on the held-out test set with precision, recall, f1-score, and accuracy, then compile accuracy, memory usage, and execution time into a single comparison table (e.g. pbmcs_bs_benchmark_comparison.csv). Interpret memory/time trade-offs together: scaLR's authors note memory usage is inversely proportional to execution time.

## Related tools

- **scaLR** (Pipeline under test; run its data ingestion, feature extraction, training, and evaluation pipelines with the memoryprofiler flag to log peak memory and execution time) — github.com/infocusp/scaLR
- **Python** (Environment for executing scaLR and the custom comparison scripts for all competitor pipelines)

## Examples

```
python pipeline.py --config /path/to/config.yaml -l -m
```

## Evaluation signals

- scaLR accuracy on the PBMCs-BS test set is comparable to the compared pipelines while its model training time and computational resources are lower
- All pipelines were run on the same VM (12GB GPU / 128GB RAM / 8 CPUs) with identical metrics logged
- scVI-tools' inability to process all 22,858 features at once (>128GB RAM needed) is explicitly recorded in the benchmark output
- Per-stage peak memory and wall-clock time are present for every scaLR run (memoryprofiler flag enabled)
- Comparison table contains accuracy, memory usage, and execution time rows for every pipeline that completed

## Limitations

- Memory usage is inversely proportional to execution time — reducing memory consumption may increase processing time
- Performance may vary based on the input data format, requiring users to ensure proper data preprocessing (e.g. .h5ad anndata with log1p-normalized adata.X) for optimal results
- Comparison tool versions are not stated and benchmark result tables may be referenced but not present in extracted text

## Evidence

- [discussion] scaLR achieves superior accuracy, memory efficiency, and execution speed versus existing frameworks: [evidence span withheld — anchor status: not_found]
- [readme] Run the entire pipeline with memory profiling: [evidence span withheld — anchor status: not_found]
- [readme] Evaluation metrics on test set: [evidence span withheld — anchor status: external_doc]
