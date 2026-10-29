---
name: cell-type-annotation-accuracy-evaluation
description: Use when a trained cell-type annotation model (e.g. a DNN on .h5ad scRNA-seq
  data) must be scored against ground-truth labels on a held-out test set, or when
  comparing annotation tools for accuracy and run feasibility on large datasets (>0.6M
  cells).
when_to_use_negative:
- The dataset has no ground-truth cell-type labels in adata.obs (no 'target' column),
  so no held-out test accuracy can be computed.
- The goal is biomarker/feature discovery rather than annotation accuracy — feature
  importance extraction is not available for comparison tools (scBalance, scvi, devCellPy,
  cellPLM), limiting downstream comparability.
- The input has not been preprocessed to the expected format (log1p normalization,
  range 0-10); scaLR's performance may vary based on the input data format.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3173
tools:
- name: scaLR
  role: 'Provides the evaluation stage of the pipeline: computes precision, recall,
    f1-score, accuracy, and ROC-AUC curves on the held-out test set from the trained
    DNN checkpoint.'
  repo: https://github.com/infocusp/scaLR
- name: cellPLM
  role: Baseline Python-based annotation pipeline compared on accuracy and resource
    usage; the only baseline able to complete on the >0.6M-cell PBMCs-normal dataset
    within 128 GB.
- name: scBalance
  role: Baseline annotation pipeline run via custom scripts from the scaLR comparison
    repository; crashes on PBMCs-SLE and exceeds 128 GB on PBMCs-normal.
- name: devCellPy
  role: Baseline annotation pipeline benchmarked for accuracy and memory usage under
    the 128 GB limit.
- name: scvi
  role: Baseline annotation pipeline included in the benchmark comparison runs.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf243
    title: 'scaLR: a low-resource deep neural network-based platform for single cell
      analysis and biomarker discovery'
  source_content_hash: sha256:04e7a7b612dfc6c695563e63939d26264c6849a8f7ad9d1ac4d536d43210445c
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
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# cell-type-annotation-accuracy-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates trained cell-type annotation models on a held-out test set of scRNA-seq data using precision, recall, f1-score, and accuracy, and benchmarks competing annotation pipelines (e.g. CellTypist, devCellPy, scBalance, cellPLM) under a fixed resource budget such as 128 GB RAM.

## When to use

Use when a trained cell-type annotation model (e.g. a DNN on .h5ad scRNA-seq data) must be scored against ground-truth labels on a held-out test set, or when comparing annotation tools for accuracy and run feasibility on large datasets (>0.6M cells).

## When NOT to use

- The dataset has no ground-truth cell-type labels in adata.obs (no 'target' column), so no held-out test accuracy can be computed.
- The goal is biomarker/feature discovery rather than annotation accuracy — feature importance extraction is not available for comparison tools (scBalance, scvi, devCellPy, cellPLM), limiting downstream comparability.
- The input has not been preprocessed to the expected format (log1p normalization, range 0-10); scaLR's performance may vary based on the input data format.

## Inputs

- anndata .h5ad scRNA-seq dataset with a 'target' cell-type column in adata.obs and log1p-normalized expression in adata.X
- trained model checkpoint from the scaLR final training stage
- held-out test split of cell samples with ground-truth cell-type labels
- optional: baseline tool outputs (scBalance, scvi, devCellPy, cellPLM) run under a 128 GB memory limit

## Outputs

- per-class precision, recall, f1-score, and accuracy on the test set
- per-class ROC-AUC curves
- benchmark comparison table (benchmark_comparison.csv) recording run completion, peak memory, runtime, and accuracy per tool-dataset pair

## How to apply

Hold out a labeled test split of the anndata (.h5ad) dataset using the pipeline's splitter (e.g. GroupSplitter with split_ratio [7, 1, 2.5], stratified on donor_id), then run the trained model checkpoint over the test set in batches (SimpleDataLoader, batch_size 15000). Compute precision, recall, f1-score, and accuracy per class on the test predictions, and optionally per-class ROC curves via the RocAucCurve analysis stage. When benchmarking against external pipelines (scBalance, scvi, devCellPy, cellPLM), run each tool-dataset pair under the same memory limit (128 GB), record run completion, peak memory, and runtime, and evaluate only completed runs on the same test metrics. Assemble tool-dataset outcomes into a benchmark comparison table and verify it reproduces the expected pattern (e.g. only cellPLM completes on the PBMCs-normal dataset within 128 GB).

## Related tools

- **scaLR** (Provides the evaluation stage of the pipeline: computes precision, recall, f1-score, accuracy, and ROC-AUC curves on the held-out test set from the trained DNN checkpoint.) — https://github.com/infocusp/scaLR
- **cellPLM** (Baseline Python-based annotation pipeline compared on accuracy and resource usage; the only baseline able to complete on the >0.6M-cell PBMCs-normal dataset within 128 GB.)
- **scBalance** (Baseline annotation pipeline run via custom scripts from the scaLR comparison repository; crashes on PBMCs-SLE and exceeds 128 GB on PBMCs-normal.)
- **devCellPy** (Baseline annotation pipeline benchmarked for accuracy and memory usage under the 128 GB limit.)
- **scvi** (Baseline annotation pipeline included in the benchmark comparison runs.)

## Examples

```
python pipeline.py --config scaLR/tutorials/pipeline/config_celltype.yaml -l -m
```

## Evaluation signals

- Metrics are computed exclusively on the held-out test split, never on training or validation batches.
- Per-class precision, recall, and f1-score are reported alongside overall accuracy (all four metrics expected).
- The benchmark table reproduces the paper's completion/failure pattern: baseline tools crash on PBMCs-SLE and only cellPLM completes on PBMCs-normal within 128 GB.
- Per-class ROC-AUC curves are generated as part of the analysis stage downstream outputs.
- scaLR's accuracy is competitive with baselines while requiring less training time and memory.

## Limitations

- Memory usage is inversely proportional to execution time — reducing memory consumption may increase processing time.
- Feature importance cannot be extracted from several comparison tools (scBalance, scvi, devCellPy, cellPLM), limiting comparability of biomarker ranking.
- Specific model hyperparameters (layers, epochs) used in final runs are only referenced via sensitivity analysis without reported values.
- Versions of the comparison tools are not stated in the paper or the comparison repository scripts.

## Evidence

- [discussion] evaluation and downstream analysis (precision, recall, f1-score, accuracy; SHAP for multi-layer models): "The trained model is then evaluated on the test set using metrics such as precision, recall, f1-score, and accuracy scores"
- [readme] readme: evaluation stage: [evidence span withheld — anchor status: external_doc]
- [abstract] abstract: comparable accuracy at lower cost: "scaLR offers comparable prediction accuracy and requires less model training time and computational resources than existing Python-based pipelines."
- [readme] readme: ROC-AUC analysis output: [evidence span withheld — anchor status: not_found]
- [readme] readme: pipeline invocation: [evidence span withheld — anchor status: not_found]
