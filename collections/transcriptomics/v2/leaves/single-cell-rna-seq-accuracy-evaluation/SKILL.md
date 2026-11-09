---
name: single-cell-rna-seq-accuracy-evaluation
description: Use when a cell-type or cell-state classifier trained on scRNA-seq data
  (.h5ad anndata, e.g. PBMCs-BS top-3,500 scaLR features) must be scored on a held-out
  test set, or when benchmarking prediction accuracy across annotation pipelines.
when_to_use_negative:
- The goal is biomarker ranking rather than classification accuracy — several comparison
  tools (scBalance, scvi, devCellPy, CellPLM) cannot extract top features, limiting
  comparability of feature importance
- Data has not been preprocessed into the expected anndata format (log1p-normalized
  adata.X, cell metadata in obs with target column) — scaLR's performance may vary
  based on input data format
- Evaluating differential gene expression or DGE biomarker analysis (Pseudo bulk /
  LMEM), which is a separate downstream module
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scaLR
  role: Trains the DNN on top-K features batch-wise from disk and evaluates on the
    test set via the scalr.analysis.evaluation module; provides custom comparison
    scripts for benchmarking other pipelines
  repo: https://github.com/infocusp/scaLR
- name: Python
  role: Implementation language of the scaLR platform and the comparison pipeline
    scripts
  repo: https://github.com/infocusp/scaLR
- name: CellAssign
  role: Compared annotation pipeline run on the same top-3,500 features for accuracy/time/memory
    benchmarking
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf243
    title: 'scaLR: a low-resource deep neural network-based platform for single cell
      analysis and biomarker discovery'
  source_content_hash: sha256:7d5832f2a42420cecf41600c7f15f250c3077f66fef808b41284fef6240d746d
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# single-cell-rna-seq-accuracy-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates a trained single-cell classifier (e.g. scaLR DNN on top-3,500 features) on a held-out test set using precision, recall, f1-score, and accuracy, and aggregates per-pipeline accuracy alongside wall-clock time and peak memory into a comparison table (e.g. pipeline_comparison_metrics.csv reproducing Table 3 of scaLR).

## When to use

Use when a cell-type or cell-state classifier trained on scRNA-seq data (.h5ad anndata, e.g. PBMCs-BS top-3,500 scaLR features) must be scored on a held-out test set, or when benchmarking prediction accuracy across annotation pipelines.

## When NOT to use

- The goal is biomarker ranking rather than classification accuracy — several comparison tools (scBalance, scvi, devCellPy, CellPLM) cannot extract top features, limiting comparability of feature importance
- Data has not been preprocessed into the expected anndata format (log1p-normalized adata.X, cell metadata in obs with target column) — scaLR's performance may vary based on input data format
- Evaluating differential gene expression or DGE biomarker analysis (Pseudo bulk / LMEM), which is a separate downstream module

## Inputs

- Trained model checkpoint for each pipeline (e.g. scaLR SequentialModel on top-3,500 features)
- Held-out test split of the anndata .h5ad dataset (e.g. PBMCs-BS) with obs.target labels (e.g. cell_type)
- Feature subset (top-3,500 ranked feature list from scaLR feature extraction)
- Custom comparison scripts from the scaLR comparison GitHub repository

## Outputs

- Per-pipeline test-set accuracy (precision, recall, f1-score, accuracy)
- Wall-clock training time per pipeline
- Maximum (peak) memory usage per pipeline
- Aggregated comparison table written as pipeline_comparison_metrics.csv

## How to apply

Train the model (scaLR SequentialModel trained batch-wise on top-K features read directly from disk), then evaluate it on the test split via the scalr analysis/evaluation module, computing precision, recall, f1-score, and accuracy per class. Run each compared pipeline (scvi, devCellPy, CellPLM, scBalance, CellAssign, etc.) using the custom comparison scripts from the scaLR comparison GitHub repository on the identical top-3,500-feature subset within the fixed VM (12GB GPU / 128GB RAM / 8 CPUs). For every pipeline, record test-set accuracy plus wall-clock training time and maximum memory usage during execution. Aggregate per-pipeline accuracy, time, and peak-memory values into a single CSV (pipeline_comparison_metrics.csv) and check that scaLR attains accuracy comparable to existing Python-based pipelines while requiring less training time and computational resources.

## Related tools

- **scaLR** (Trains the DNN on top-K features batch-wise from disk and evaluates on the test set via the scalr.analysis.evaluation module; provides custom comparison scripts for benchmarking other pipelines) — https://github.com/infocusp/scaLR
- **Python** (Implementation language of the scaLR platform and the comparison pipeline scripts) — https://github.com/infocusp/scaLR
- **CellAssign** (Compared annotation pipeline run on the same top-3,500 features for accuracy/time/memory benchmarking)

## Examples

```
python pipeline.py --config /path/to/config.yaml -l -m
```

## Evaluation signals

- Per-pipeline metrics computed on the identical top-3,500-feature subset and identical train/test splits, so accuracy values are directly comparable
- Reproduced Table 3 pattern: scaLR's prediction accuracy is comparable to existing Python-based pipelines while wall-clock time and maximum memory usage are lower than all compared pipelines
- Accuracy is computed on the held-out test set only (precision, recall, f1-score, accuracy scores from the evaluation module)
- Memory and timing recorded under a fixed VM configuration (12GB GPU / 128GB RAM / 8 CPUs) so hardware does not confound the comparison
- Output file pipeline_comparison_metrics.csv contains one row per pipeline with accuracy, wall-clock time, and peak memory columns

## Limitations

- scaLR's memory usage is inversely proportional to execution time, so reducing memory consumption may increase processing time
- Feature importance cannot be extracted from several comparison tools (scBalance, scvi, devCellPy, CellPLM), limiting biomarker-ranking comparability
- Comparison tool versions are not stated in the article; benchmark tables (Tables 2–4) were referenced but not present in the extracted text

## Evidence

- [readme] README: evaluation metrics on test dataset: [evidence span withheld — anchor status: external_doc]
