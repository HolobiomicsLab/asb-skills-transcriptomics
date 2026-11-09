---
name: cross-pipeline-benchmark-comparison
description: Use when you need to position a new single-cell analysis tool against
  existing Python ML/DNN pipelines by comparing accuracy, memory usage, and wall-clock
  execution time on a shared dataset and VM.
when_to_use_negative:
- When you only need to characterize one pipeline's standalone performance without
  a same-hardware, same-dataset baseline — the comparison is only meaningful under
  controlled identical conditions
- When the comparison tools cannot expose needed outputs for fair evaluation — the
  paper notes top features cannot be extracted from scBalance, scvi, devCellPy, and
  cellPLM, limiting biomarker-ranking comparability
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scaLR
  role: 'Target pipeline under benchmark: cell type/state annotation via feature extraction,
    model training, and evaluation pipelines on the full feature set'
  repo: https://github.com/infocusp/scaLR
- name: scVI-tools
  role: Comparison pipeline; failed to process all 22,858 features at once due to
    >128GB RAM demand
- name: Python
  role: Implementation language for all benchmarked pipelines and custom comparison
    scripts
- name: CellTypist
  role: Comparison pipeline variant run via custom scripts with identical resource
    logging
- name: ACTINN
  role: Comparison pipeline run via custom scripts with identical resource logging
- name: SingleCellNet
  role: Comparison pipeline run via custom scripts with identical resource logging
- name: devCellPy
  role: Comparison pipeline run via custom scripts with identical resource logging
- name: scBalance
  role: Comparison pipeline run via custom scripts with identical resource logging
- name: cellPLM
  role: Comparison pipeline run via custom scripts with identical resource logging
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf243
    title: 'scaLR: a low-resource deep neural network-based platform for single cell
      analysis and biomarker discovery'
  source_content_hash: sha256:ca353dbc05e1de103e326a67e9682db7cd220bd751b4a76170e131b3045141ad
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/infocusp/scaLR
  repo_url: https://github.com/infocusp/scaLR
  tools:
  - ACTINN
  - CellTypist
  - Python
  - SingleCellNet
  - cellPLM
  - devCellPy
  - scBalance
  - scVI-tools
  - scaLR
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# cross-pipeline-benchmark-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a target single-cell pipeline (e.g. scaLR) against competing frameworks (scVI-tools, SingleCellNet, CellTypist, ACTINN, devCellPy, scBalance, cellPLM) on the same dataset, hardware, and task, recording prediction accuracy, peak memory, and execution time.

## When to use

Use when you need to position a new single-cell analysis tool against existing Python ML/DNN pipelines by comparing accuracy, memory usage, and wall-clock execution time on a shared dataset and VM.

## When NOT to use

- When you only need to characterize one pipeline's standalone performance without a same-hardware, same-dataset baseline — the comparison is only meaningful under controlled identical conditions
- When the comparison tools cannot expose needed outputs for fair evaluation — the paper notes top features cannot be extracted from scBalance, scvi, devCellPy, and cellPLM, limiting biomarker-ranking comparability

## Inputs

- scRNA-seq benchmark dataset in .h5ad (anndata) format with a target column (e.g. PBMCs-BS, 22,858 features)
- Custom runner scripts for each comparison pipeline (from the scaLR comparison GitHub repository)
- Hardware specification to hold constant (VM with GPU, RAM, CPU count)

## Outputs

- Comparison table (e.g. pbmcs_bs_benchmark_comparison.csv) with accuracy, precision, recall, f1-score, peak memory usage, and execution time per pipeline
- Per-pipeline test-set classification predictions
- Recorded failures/resource-limit outcomes (e.g. scVI-tools unable to process all features within 128GB RAM)

## How to apply

Run all pipelines on identical hardware (the scaLR benchmark used a 12GB GPU / 128GB RAM / 8 CPU VM) and dataset (e.g. PBMCs-bacterial sepsis scRNA-seq with 22,858 features). For the target tool, execute its end-to-end pipeline (e.g. scalr.data_ingestion_pipeline with subset-wise normalization and backed-mode reading, then feature_extraction_pipeline, model_training_pipeline, and eval_and_analysis_pipeline) while logging peak memory and wall-clock time. Run each comparison pipeline with custom scripts (kept in the scaLR comparison GitHub repository), logging identical resource metrics and noting failures such as scVI-tools exceeding 128GB RAM on all 22,858 features. Evaluate every pipeline's test-set predictions with precision, recall, f1-score, and accuracy, then compile accuracy, memory, and execution time into a single comparison table (e.g. pbmcs_bs_benchmark_comparison.csv).

## Related tools

- **scaLR** (Target pipeline under benchmark: cell type/state annotation via feature extraction, model training, and evaluation pipelines on the full feature set) — https://github.com/infocusp/scaLR
- **scVI-tools** (Comparison pipeline; failed to process all 22,858 features at once due to >128GB RAM demand)
- **Python** (Implementation language for all benchmarked pipelines and custom comparison scripts)
- **CellTypist** (Comparison pipeline variant run via custom scripts with identical resource logging)
- **ACTINN** (Comparison pipeline run via custom scripts with identical resource logging)
- **SingleCellNet** (Comparison pipeline run via custom scripts with identical resource logging)
- **devCellPy** (Comparison pipeline run via custom scripts with identical resource logging)
- **scBalance** (Comparison pipeline run via custom scripts with identical resource logging)
- **cellPLM** (Comparison pipeline run via custom scripts with identical resource logging)

## Examples

```
python pipeline.py --config /path/to/config.yaml -l -m
```

## Evaluation signals

- All pipelines evaluated on the same test set with precision, recall, f1-score, and accuracy reported
- Memory and wall-clock time logged under identical hardware (12GB GPU / 128GB RAM / 8 CPUs) for every pipeline
- Target tool (scaLR) shows prediction accuracy comparable to compared pipelines while requiring less training time and fewer computational resources
- Hard-limit outcomes documented (e.g. scVI-tools cannot process all 22,858 features within 128GB RAM)
- All metrics compiled into a single CSV (pbmcs_bs_benchmark_comparison.csv) with one row per pipeline

## Limitations

- Memory usage is inversely proportional to execution time — reducing memory consumption may increase processing time
- Comparison tool versions are not stated, and custom scripts live only in the scaLR comparison GitHub repository
- Feature importance cannot be extracted from scBalance, scvi, devCellPy, and cellPLM, so biomarker ranking is not comparable across all pipelines
- scaLR's performance may vary based on input data format, requiring users to ensure proper preprocessing

## Evidence
