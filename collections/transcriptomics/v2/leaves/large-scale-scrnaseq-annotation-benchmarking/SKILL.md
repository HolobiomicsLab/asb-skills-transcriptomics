---
name: large-scale-scrnaseq-annotation-benchmarking
description: Use when evaluating whether Python-based annotation pipelines (CellTypist,
  devCellPy, SingleCellNet, scBalance, ACTINN, CellPLM, scvi, scaLR) can process datasets
  with >0.6 million cells under a stated memory limit, or when reproducing cross-cohort
  benchmark tables.
when_to_use_negative:
- Input is a small dataset (<1 million cells) where memory is not a constraint — full-scale
  baseline tools will complete and the resource-cap benchmarking framing is unnecessary
- The goal is marker-gene-based annotation with prior marker knowledge rather than
  supervised reference-based benchmarking across tools
- You need feature importance/top biomarkers from every compared tool — scBalance,
  scvi, devCellPy, and CellPLM do not expose top features from their generated models,
  so that axis of comparison cannot be completed
tools:
- name: scaLR
  role: Low-resource DNN platform under test; run via the four-stage pipeline.py with
    batch partitioning and backed-mode loading
  repo: github.com/infocusp/scaLR
- name: scBalance
  role: Baseline annotation pipeline compared under the 128 GB memory limit via custom
    comparison scripts
- name: scvi
  role: Baseline annotation pipeline compared under the 128 GB memory limit via custom
    comparison scripts
- name: devCellPy
  role: Baseline annotation pipeline compared under the 128 GB memory limit via custom
    comparison scripts
- name: CellPLM
  role: Baseline annotation pipeline; the only comparison tool that completed on PBMCs-normal
    within 128 GB
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf243
    title: 'scaLR: a low-resource deep neural network-based platform for single cell
      analysis and biomarker discovery'
  source_content_hash: sha256:fea858578db128564a9450acc9e3499fa566a052fc87f1b635e2f5ff6b22742a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/infocusp/scaLR
  tools:
  - CellPLM
  - Python
  - cellPLM
  - devCellPy
  - scBalance
  - scaLR
  - scvi
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# large-scale-scrnaseq-annotation-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark cell-type annotation pipelines on multi-million-cell scRNA-seq datasets under a fixed resource cap (e.g. 128 GB RAM), recording run completion/crash, peak memory, runtime, and precision/recall/f1/accuracy. It establishes whether low-resource DNN platforms like scaLR match baseline tool accuracy while remaining executable on limited hardware.

## When to use

Use when evaluating whether Python-based annotation pipelines (CellTypist, devCellPy, SingleCellNet, scBalance, ACTINN, CellPLM, scvi, scaLR) can process datasets with >0.6 million cells under a stated memory limit, or when reproducing cross-cohort benchmark tables.

## When NOT to use

- Input is a small dataset (<1 million cells) where memory is not a constraint — full-scale baseline tools will complete and the resource-cap benchmarking framing is unnecessary
- The goal is marker-gene-based annotation with prior marker knowledge rather than supervised reference-based benchmarking across tools
- You need feature importance/top biomarkers from every compared tool — scBalance, scvi, devCellPy, and CellPLM do not expose top features from their generated models, so that axis of comparison cannot be completed

## Inputs

- Large-scale scRNA-seq datasets in anndata .h5ad format (e.g. PBMCs-normal >0.6M cells, PBMCs-SLE, PBMCs-C19-F, cross-cohort data from Cellxgene and the Broad Single Cell Portal)
- Custom run scripts for comparison pipelines (scBalance, scvi, devCellPy, CellPLM) from the scaLR comparison GitHub repository
- scaLR config.yaml with data, feature selection, final training, and analysis stages configured
- Resource cap specification (e.g. 128 GB RAM) and memory profiler

## Outputs

- benchmark_comparison.csv with per tool-dataset run completion/crash status, peak memory usage, and total runtime
- Annotation accuracy metrics (precision, recall, f1-score, accuracy) on the test set for completed runs
- Training time and memory comparison showing scaLR's resource cost versus baseline pipelines

## How to apply

Download the large-scale benchmark datasets (PBMCs-normal >0.6M cells, PBMCs-SLE, and cross-cohort studies sourced from Cellxgene and the Broad Single Cell Portal) as anndata .h5ad files. Execute each comparison pipeline (scBalance, scvi, devCellPy, CellPLM) via the custom scripts in the scaLR comparison GitHub repository under the 128 GB memory limit, and run scaLR's four pipelines (scalr.data_ingestion_pipeline, scalr.feature_extraction_pipeline, scalr.model_training_pipeline, scalr.eval_and_analysis_pipeline) using batch partitioning (sample_chunksize ~20000) and backed-mode loading. For each tool-dataset pair, record whether the run completes or crashes, together with peak memory usage (enable --memoryprofiler) and total runtime. For completed runs, evaluate annotation accuracy on the held-out test set using precision, recall, f1-score, and accuracy. Assemble results into benchmark_comparison.csv; validate that the table reproduces the paper's pattern — baseline tools crash on PBMCs-SLE, only CellPLM completes on PBMCs-normal within 128 GB, and scaLR processes the >11.5M-cell cross-cohort data with competitive accuracy at lower training time and memory.

## Related tools

- **scaLR** (Low-resource DNN platform under test; run via the four-stage pipeline.py with batch partitioning and backed-mode loading) — github.com/infocusp/scaLR
- **scBalance** (Baseline annotation pipeline compared under the 128 GB memory limit via custom comparison scripts)
- **scvi** (Baseline annotation pipeline compared under the 128 GB memory limit via custom comparison scripts)
- **devCellPy** (Baseline annotation pipeline compared under the 128 GB memory limit via custom comparison scripts)
- **CellPLM** (Baseline annotation pipeline; the only comparison tool that completed on PBMCs-normal within 128 GB)

## Examples

```
python pipeline.py --config config.yaml -l -m
```

## Evaluation signals

- benchmark_comparison.csv reproduces the paper's reported pattern: baseline tools crash on PBMCs-SLE, only CellPLM completes on PBMCs-normal within 128 GB, scaLR completes on >11.5M-cell cross-cohort data
- Completed runs report test-set precision, recall, f1-score, and accuracy, with scaLR matching baseline annotation accuracy
- scaLR requires less model training time and memory than the compared Python-based pipelines for equivalent accuracy
- Memory profiling logs (-m flag) capture peak memory usage per tool-dataset pair
- scaLR runs process data in batches (sample_chunksize ~20000, batch_size 15000-25000) in backed mode without loading the full matrix

## Limitations

- Memory usage is inversely proportional to execution time — reducing memory consumption may increase processing time
- Comparison tool versions are not stated, and specific DNN hyperparameters (layer/epoch values in final runs) are referenced only via sensitivity analysis
- scaLR's performance may vary based on the input data format, requiring users to ensure proper preprocessing (log1p-normalized adata.X in range 0-10) for optimal results
- Feature importance cannot be extracted from scBalance, scvi, devCellPy, or CellPLM, limiting comparability of biomarker ranking

## Evidence

- [readme] pipeline invocation: [evidence span withheld — anchor status: not_found]
