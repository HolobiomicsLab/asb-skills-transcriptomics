---
name: wallclock-and-memory-profiling-of-pipelines
description: Use when comparing multiple cell type annotation pipelines for a benchmark
  table and you must report, per pipeline, prediction accuracy together with wall-clock
  execution time and maximum memory usage under fixed hardware (e.g. 12GB GPU / 128GB
  RAM / 8 CPU VM).
when_to_use_negative:
- When you only need prediction quality (accuracy/precision/recall/f1) and resource
  consumption is out of scope — plain test-set evaluation via scalr.analysis.evaluation
  suffices
- 'When input data format is unverified: scaLR performance may vary based on the input
  data format, requiring proper preprocessing (log1p-normalized adata.X in range 0–10)
  before timing comparisons are meaningful'
- When the goal is biomarker ranking comparability — feature importance cannot be
  extracted from scBalance, scvi, devCellPy, and CellPLM, so those pipelines cannot
  participate in feature-based comparisons
tools:
- name: scaLR
  role: Pipeline under test; runs data ingestion, batch-wise disk-backed training,
    and evaluation while its wall-clock time and max memory are recorded
  repo: https://github.com/infocusp/scaLR
- name: Python
  role: Language in which scaLR and the compared pipelines are implemented and in
    which the profiling/comparison scripts run
  repo: https://github.com/infocusp/scaLR
- name: CellAssign
  role: Compared annotation pipeline run via custom comparison scripts on the same
    top-3,500 features for the resource benchmark
- name: scvi
  role: Compared pipeline whose feature importances cannot be extracted, but whose
    accuracy, time, and memory are recorded in the comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf243
    title: 'scaLR: a low-resource deep neural network-based platform for single cell
      analysis and biomarker discovery'
  source_content_hash: sha256:56235b19eec4b781cc39ff839820ad688faa21c949680c0640ec3ad20dee87a6
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
  - CellAssign
  - Python
  - scaLR
  - scvi
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# wallclock-and-memory-profiling-of-pipelines

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarking skill that measures wall-clock training time and peak (maximum) memory consumption of scRNA-seq cell-classification pipelines run on identical inputs, so resource profiles can be compared side-by-side with prediction accuracy as in scaLR's Table 3 comparison on the PBMCs-BS dataset.

## When to use

Use when comparing multiple cell type annotation pipelines for a benchmark table and you must report, per pipeline, prediction accuracy together with wall-clock execution time and maximum memory usage under fixed hardware (e.g. 12GB GPU / 128GB RAM / 8 CPU VM).

## When NOT to use

- When you only need prediction quality (accuracy/precision/recall/f1) and resource consumption is out of scope — plain test-set evaluation via scalr.analysis.evaluation suffices
- When input data format is unverified: scaLR performance may vary based on the input data format, requiring proper preprocessing (log1p-normalized adata.X in range 0–10) before timing comparisons are meaningful
- When the goal is biomarker ranking comparability — feature importance cannot be extracted from scBalance, scvi, devCellPy, and CellPLM, so those pipelines cannot participate in feature-based comparisons

## Inputs

- PBMCs-BS scRNA-seq count matrix in anndata .h5ad format restricted to the scaLR top-3,500 ranked feature list
- Installed comparison pipelines (scaLR, scvi, devCellPy, CellPLM, scBalance, CellAssign, etc.) with custom comparison scripts from the scaLR comparison GitHub repository
- Fixed hardware environment: 12GB GPU / 128GB RAM / 8 CPU VM

## Outputs

- pipeline_comparison_metrics.csv with per-pipeline accuracy, wall-clock training time, and maximum memory usage
- A comparison table equivalent to the paper's Table 3 (resource profile of scaLR vs. other pipelines)

## How to apply

Run each pipeline (scaLR via scalr.data_ingestion_pipeline, scalr.model_training_pipeline, scalr.eval_and_analysis_pipeline; competitors such as scvi, devCellPy, CellPLM, scBalance, CellAssign via the custom scripts in the scaLR comparison GitHub repository) on the same input — e.g. the scaLR-extracted top-3,500 features of the PBMCs-BS .h5ad dataset. Execute scaLR with training data read directly from disk in batches (SimpleDataLoader with batch_size like 15000–25000) to bound memory, and invoke pipeline.py with the -m | --memoryprofiler flag to instrument memory usage and -l for logging. After training, compute test-set accuracy through scalr.analysis.evaluation using precision, recall, f1-score, and accuracy scores, and record wall-clock training time plus peak memory observed during each run. Aggregate per-pipeline accuracy, wall-clock time, and peak-memory values into a single table (pipeline_comparison_metrics.csv) mirroring the paper's Table 3 pattern. Remember the acknowledged trade-off: scaLR's memory usage is inversely proportional to execution time, so a low-memory setting may increase runtime.

## Related tools

- **scaLR** (Pipeline under test; runs data ingestion, batch-wise disk-backed training, and evaluation while its wall-clock time and max memory are recorded) — https://github.com/infocusp/scaLR
- **Python** (Language in which scaLR and the compared pipelines are implemented and in which the profiling/comparison scripts run) — https://github.com/infocusp/scaLR
- **CellAssign** (Compared annotation pipeline run via custom comparison scripts on the same top-3,500 features for the resource benchmark)
- **scvi** (Compared pipeline whose feature importances cannot be extracted, but whose accuracy, time, and memory are recorded in the comparison)

## Examples

```
python pipeline.py --config /path/to/config.yaml -l -m
```

## Evaluation signals

- The reproduced comparison table reproduces the reported pattern: scaLR achieves prediction accuracy comparable to existing Python-based pipelines while requiring less model training time and computational resources on the top-3,500-feature setting
- scaLR's wall-clock time and maximum memory usage for cell type and state classification are better than all other compared pipelines (Table 3 pattern)
- Each pipeline was run on the identical input (top-3,500 scaLR features of PBMCs-BS) within the same 12GB GPU / 128GB RAM / 8 CPU VM, so timings and memory are comparable
- Test-set metrics for every pipeline are computed via the same evaluation module (precision, recall, f1-score, accuracy scores), not pipeline-specific reporting
- A single aggregated CSV (pipeline_comparison_metrics.csv) exists with all three metric types populated for every pipeline

## Limitations

- Memory usage is inversely proportional to execution time: reducing memory consumption may increase processing time, so single-number resource rankings can obscure this trade-off
- Comparison tool versions are not stated, and benchmark result tables (Tables 2–4) were referenced but not present in the extracted text
- scaLR's performance may vary based on the input data format, requiring users to ensure proper data preprocessing for optimal results before fair timing comparisons

## Evidence

- [discussion] scaLR's wall-clock time and maximum memory usage for cell type and state classification are better than all other compared pipelines: [evidence span withheld — anchor status: not_found]
- [readme] memoryprofiler CLI flag: [evidence span withheld — anchor status: external_doc]
