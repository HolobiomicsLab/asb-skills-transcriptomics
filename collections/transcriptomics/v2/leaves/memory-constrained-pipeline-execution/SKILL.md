---
name: memory-constrained-pipeline-execution
description: Use when an scRNA-seq dataset is too large to fit in RAM (e.g. >0.6 million
  cells, >1 million cells) and annotation, feature selection, or DGE analysis must
  run under a fixed memory cap such as 128 GB.
when_to_use_negative:
- The dataset fits comfortably in RAM — batched/backed-mode processing adds overhead
  and the paper notes memory usage is inversely proportional to execution time
- Input is not in .h5ad anndata format — the pipeline requires user preprocessing
  to convert data first, since scaLR's performance varies with input format
- The task requires feature importances from tools like scBalance, scvi, devCellPy,
  or CellPLM, which cannot expose top features from their generated models
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: scaLR
  role: Low-resource platform whose four pipelines (data_ingestion, feature_extraction,
    model_training, eval_and_analysis) partition large datasets into manageable batches
    and read data in backed mode
  repo: https://github.com/infocusp/scaLR
- name: Python
  role: Execution environment (tested in Python 3.10.20) for running the pipeline
    with memory profiling
  repo: https://github.com/infocusp/scaLR
- name: cellPLM
  role: Comparison annotation tool; the only baseline that completed on the >0.6M-cell
    PBMCs-normal dataset within 128 GB
- name: scBalance
  role: Comparison annotation pipeline run via custom scripts from the scaLR comparison
    repository; crashed under the 128 GB memory limit on large datasets
  repo: https://github.com/infocusp/scaLR
- name: devCellPy
  role: Comparison annotation pipeline that required more than 128 GB of memory on
    PBMCs-normal and failed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf243
    title: 'scaLR: a low-resource deep neural network-based platform for single cell
      analysis and biomarker discovery'
  source_content_hash: sha256:338e189d21ee26712a13459f90df3428927cd134bbce092372be567ba19b3e2d
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
  - Python
  - cellPLM
  - devCellPy
  - scBalance
  - scaLR
  - scvi
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# memory-constrained-pipeline-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Executes large-scale scRNA-seq analysis pipelines (millions of cells) on limited-memory machines (e.g. 128 GB RAM) by partitioning data into batches, reading .h5ad in backed mode, and applying subset-wise normalization, as done in the scaLR platform.

## When to use

Use when an scRNA-seq dataset is too large to fit in RAM (e.g. >0.6 million cells, >1 million cells) and annotation, feature selection, or DGE analysis must run under a fixed memory cap such as 128 GB.

## When NOT to use

- The dataset fits comfortably in RAM — batched/backed-mode processing adds overhead and the paper notes memory usage is inversely proportional to execution time
- Input is not in .h5ad anndata format — the pipeline requires user preprocessing to convert data first, since scaLR's performance varies with input format
- The task requires feature importances from tools like scBalance, scvi, devCellPy, or CellPLM, which cannot expose top features from their generated models

## Inputs

- Large-scale scRNA-seq dataset in anndata .h5ad format (adata.X log1p-normalized counts, adata.obs with target/cell_type column, adata.var gene names)
- Pipeline configuration file (config.yaml) specifying chunk sizes, splitter, model layers, and epochs
- Hardware resource limit (e.g. 128 GB RAM)

## Outputs

- Completed pipeline results (trained model checkpoint, precision/recall/f1-score/accuracy on test set) produced within the memory cap
- Peak memory usage and total runtime per tool-dataset pair (e.g. benchmark_comparison.csv)
- Completion/crash status per run under the resource limit

## How to apply

Configure the pipeline (config.yaml) to read the anndata .h5ad input in backed mode and process it in batches (e.g. sample_chunksize: 20000, dataloader batch_size 15000–25000). Apply subset-wise preprocessing such as SampleNorm instead of global normalization, and split data with GroupSplitter stratified by donor_id so folds stay within memory limits. During feature extraction, train on feature subsets batch-wise so all features and samples are used without full in-memory loading, then train the final DNN on top-K features (e.g. k=5000) reading training data directly from disk. Run with the -m/--memoryprofiler flag to track peak memory usage, and record whether each run completes or crashes plus total runtime when benchmarking against tools that require more than the available memory.

## Related tools

- **scaLR** (Low-resource platform whose four pipelines (data_ingestion, feature_extraction, model_training, eval_and_analysis) partition large datasets into manageable batches and read data in backed mode) — https://github.com/infocusp/scaLR
- **Python** (Execution environment (tested in Python 3.10.20) for running the pipeline with memory profiling) — https://github.com/infocusp/scaLR
- **cellPLM** (Comparison annotation tool; the only baseline that completed on the >0.6M-cell PBMCs-normal dataset within 128 GB)
- **scBalance** (Comparison annotation pipeline run via custom scripts from the scaLR comparison repository; crashed under the 128 GB memory limit on large datasets) — https://github.com/infocusp/scaLR
- **devCellPy** (Comparison annotation pipeline that required more than 128 GB of memory on PBMCs-normal and failed)

## Examples

```
python pipeline.py --config /path/to/config.yaml -l -m
```

## Evaluation signals

- Run completes under the 128 GB limit where baseline tools (CellTypist, devCellPy, SingleCellNet, scBalance, ACTINN) crash on the same dataset
- Peak memory usage recorded via the -m/--memoryprofiler flag stays below the resource cap for every stage
- Annotation metrics (precision, recall, f1-score, accuracy) on the test set are competitive with in-memory baselines while using less training time and memory
- Benchmark table reproduces the paper's pattern: only cellPLM completes on PBMCs-normal (>0.6M cells) within 128 GB; scaLR processes >11.5M-cell cross-cohort data
- Batch-wise training reads data directly from disk rather than loading the full matrix, verified by flat (non-growing) memory profile across epochs

## Limitations

- Memory usage is inversely proportional to execution time — reducing memory consumption increases processing time
- Performance varies with input data format; users must ensure proper .h5ad preprocessing (log1p normalization, range 0–10) for optimal results
- Comparison tool versions and scaLR's own software version number are not stated in the paper, complicating exact reproduction
- Feature importance cannot be extracted from several comparison tools (scBalance, scvi, devCellPy, cellPLM), limiting comparability of biomarker ranking

## Evidence

- [results] batch partitioning and backed-mode loading: [evidence span withheld — anchor status: not_found]
- [results] only CellPLM completes within 128 GB: [evidence span withheld — anchor status: not_found]
- [readme] memoryprofiler flag: [evidence span withheld — anchor status: external_doc]
