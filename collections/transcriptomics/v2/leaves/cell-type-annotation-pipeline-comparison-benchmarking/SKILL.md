---
name: cell-type-annotation-pipeline-comparison-benchmarking
description: Use when you must quantify resource/accuracy trade-offs of cell type
  annotation pipelines on identical feature subsets of an scRNA-seq dataset (e.g.,
  reproducing a Table-3-style comparison of accuracy, memory, and execution time).
when_to_use_negative:
- When the goal is biomarker discovery or DGE analysis (e.g., pseudo bulk / LMEM volcano
  plots) rather than comparing pipeline accuracy, time, and memory
- When you need feature importance / top features from the comparison tools themselves
  — the article notes features cannot be extracted, or there is no explanation of
  how, from scBalance, scvi, devCellPy, and CellPLM models
- When input data is not preprocessed into the expected anndata .h5ad format (log1p-normalized
  adata.X, cell metadata in adata.obs with a target column) — data-format dependence
  requires user preprocessing first
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scaLR
  role: Primary pipeline under benchmark; provides feature extraction, batch-wise
    disk-read training, and the evaluation module used to score all pipelines
  repo: https://github.com/infocusp/scaLR
- name: Python
  role: Execution environment for scaLR and the custom comparison scripts for all
    compared pipelines
- name: scvi
  role: Compared annotation pipeline run on the same top-3,500 features via the scaLR
    comparison repository scripts
- name: devCellPy
  role: Compared annotation pipeline run on the same top-3,500 features
- name: CellPLM
  role: Compared annotation pipeline run on the same top-3,500 features
- name: scBalance
  role: Compared annotation pipeline run on the same top-3,500 features
- name: CellAssign
  role: Compared clustering-then-annotation pipeline run on the same top-3,500 features
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf243
    title: 'scaLR: a low-resource deep neural network-based platform for single cell
      analysis and biomarker discovery'
  source_content_hash: sha256:9e8df2ca4a7d57ffd5cd3c7b326d7fc014a2f9ecb23ea7ff8b8aad9fbb8b9ed1
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
  - CellPLM
  - Python
  - devCellPy
  - scBalance
  - scaLR
  - scvi
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cell-type-annotation-pipeline-comparison-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark scaLR against alternative single-cell annotation pipelines (scvi, devCellPy, CellPLM, scBalance, CellAssign, etc.) by running each on the same top-3,500 scaLR-extracted features of a scRNA-seq dataset and comparing test accuracy, wall-clock time, and peak memory in a single table.

## When to use

Use when you must quantify resource/accuracy trade-offs of cell type annotation pipelines on identical feature subsets of an scRNA-seq dataset (e.g., reproducing a Table-3-style comparison of accuracy, memory, and execution time).

## When NOT to use

- When the goal is biomarker discovery or DGE analysis (e.g., pseudo bulk / LMEM volcano plots) rather than comparing pipeline accuracy, time, and memory
- When you need feature importance / top features from the comparison tools themselves — the article notes features cannot be extracted, or there is no explanation of how, from scBalance, scvi, devCellPy, and CellPLM models
- When input data is not preprocessed into the expected anndata .h5ad format (log1p-normalized adata.X, cell metadata in adata.obs with a target column) — data-format dependence requires user preprocessing first

## Inputs

- Single-cell RNA-seq count matrix in anndata .h5ad format (e.g., PBMCs-BS dataset) with cell metadata and target column
- Top-3,500 ranked feature list produced by the upstream scaLR feature-extraction step (SGD-based scoring)
- Custom comparison scripts from the scaLR comparison GitHub repository for each alternative pipeline
- Compute environment with 12GB GPU, 128GB RAM, 8 CPUs

## Outputs

- pipeline_comparison_metrics.csv with per-pipeline test-set accuracy, wall-clock execution time, and maximum memory usage
- Reproduced benchmark table (Table 3 equivalent) showing the scaLR-vs-alternative comparison pattern

## How to apply

First run the upstream scaLR feature-extraction stage (scalr.feature_extraction_pipeline) on the anndata .h5ad count matrix (e.g., PBMCs-BS) to obtain the top-3,500 ranked feature list, using a LinearScorer and AbsMean selector in the feature_selection config. Then execute the scaLR pipeline modules (scalr.data_ingestion_pipeline, scalr.model_training_pipeline, scalr.eval_and_analysis_pipeline) restricted to those features, reading training data directly from disk in batches to bound memory usage. Run each compared pipeline (scvi, devCellPy, CellPLM, scBalance, CellAssign, etc.) with the custom scripts from the scaLR comparison GitHub repository on the same top-3,500 features, under a fixed hardware envelope (12GB GPU / 128GB RAM / 8 CPU VM) so resource metrics are comparable. For every pipeline, compute test-set accuracy via the scalr.analysis.evaluation module and record wall-clock training time and maximum memory usage during execution. Aggregate per-pipeline accuracy, wall-clock time, and peak memory into one comparison table (pipeline_comparison_metrics.csv) and check it reproduces the reported pattern: scaLR achieves accuracy comparable to existing Python-based pipelines while using less training time and computational resources.

## Related tools

- **scaLR** (Primary pipeline under benchmark; provides feature extraction, batch-wise disk-read training, and the evaluation module used to score all pipelines) — https://github.com/infocusp/scaLR
- **Python** (Execution environment for scaLR and the custom comparison scripts for all compared pipelines)
- **scvi** (Compared annotation pipeline run on the same top-3,500 features via the scaLR comparison repository scripts)
- **devCellPy** (Compared annotation pipeline run on the same top-3,500 features)
- **CellPLM** (Compared annotation pipeline run on the same top-3,500 features)
- **scBalance** (Compared annotation pipeline run on the same top-3,500 features)
- **CellAssign** (Compared clustering-then-annotation pipeline run on the same top-3,500 features)

## Examples

```
python pipeline.py --config /path/to/config.yaml -l -m
```

## Evaluation signals

- The aggregated comparison table reproduces the reported pattern: scaLR's wall-clock time and maximum memory usage for cell type and state classification are better than all compared pipelines on the top-3,500 features
- scaLR test-set accuracy is comparable to existing Python-based pipelines (precision, recall, f1-score, and accuracy from scalr.analysis.evaluation)
- All compared pipelines were run on the identical top-3,500 scaLR-extracted feature subset of the same dataset, so metrics are directly comparable
- Resource metrics were measured under the same VM envelope (12GB GPU / 128GB RAM / 8 CPU) and reflect the backed-mode, batch-wise disk-read training behavior of scaLR
- pipeline_comparison_metrics.csv exists and contains accuracy, wall-clock time, and peak memory for every pipeline in the comparison set

## Limitations

- scaLR's memory usage is inversely proportional to execution time: reducing memory consumption may increase processing time, so time/memory trade-offs depend on batching choices
- Benchmark result tables (Tables 2–4) are referenced but not present in the extracted article text; comparison tool versions are not stated
- Feature importance cannot be extracted from scBalance, scvi, devCellPy, and CellPLM, limiting comparability of biomarker ranking beyond the annotation benchmark itself

## Evidence

- [results] scaLR's wall-clock time and maximum memory usage for cell type and state classification are better than all other compared pipelines (Table 3): [evidence span withheld — anchor status: not_found]
- [discussion] custom scripts to run all the above pipelines that are present in the scaLR comparison GitHub repository: "We have designed custom scripts to run all the above pipelines that are present in the scaLR comparison GitHub repository"
- [readme] The trained model is evaluated using the test dataset by calculating metrics such as precision, recall, f1-score, and accuracy.: [evidence span withheld — anchor status: external_doc]
