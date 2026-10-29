---
name: cross-tool-annotation-benchmarking
description: Use when you need to compare annotation pipelines head-to-head on million-cell
  datasets (>1 million cells) and quantify accuracy together with resource metrics
  (training time, peak memory, GPU requirement) to derive cost-efficiency conclusions.
when_to_use_negative:
- Your goal is only to annotate one dataset accurately — use a single pipeline rather
  than a full cross-tool benchmark with resource profiling.
- Datasets are small enough to fit in memory on a workstation; the backed-mode/batch-wise
  resource comparison is designed for datasets over 1 million cells.
- You need to compare biomarker/feature rankings across tools — feature importances
  cannot be extracted from several comparison tools (scBalance, scvi, devCellPy, CellPLM),
  limiting comparability.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scaLR
  role: DNN-based annotation pipeline under benchmark; its data_ingestion_pipeline,
    model_training_pipeline, and analysis.evaluation modules provide the accuracy
    and resource measurements
  repo: https://github.com/infocusp/scaLR
- name: scvi-tools
  role: Gold-standard comparison tool run via custom benchmark scripts; its accuracy,
    runtime, and memory are recorded on the same datasets and hardware
- name: CellTypist
  role: Gold-standard comparison tool run via custom benchmark scripts; its accuracy,
    runtime, and memory are recorded on the same datasets and hardware
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbaf243
    title: 'scaLR: a low-resource deep neural network-based platform for single cell
      analysis and biomarker discovery'
  source_content_hash: sha256:22923464fe681bda99238565f69944193932df192cb8f48872aefce121f56753
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
  - CellTypist
  - Python
  - scaLR
  - scvi-tools
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cross-tool-annotation-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark cell-type annotation tools (scaLR vs scvi-tools and CellTypist) on the same large scRNA-seq datasets, comparing classification accuracy, runtime, memory consumption, and derived analysis cost on equivalent hardware.

## When to use

Use when you need to compare annotation pipelines head-to-head on million-cell datasets (>1 million cells) and quantify accuracy together with resource metrics (training time, peak memory, GPU requirement) to derive cost-efficiency conclusions.

## When NOT to use

- Your goal is only to annotate one dataset accurately — use a single pipeline rather than a full cross-tool benchmark with resource profiling.
- Datasets are small enough to fit in memory on a workstation; the backed-mode/batch-wise resource comparison is designed for datasets over 1 million cells.
- You need to compare biomarker/feature rankings across tools — feature importances cannot be extracted from several comparison tools (scBalance, scvi, devCellPy, CellPLM), limiting comparability.

## Inputs

- Large scRNA-seq datasets in anndata .h5ad format (e.g. PBMCs-SLE >1.25M cells, PBMCs-HED >2M cells) with a cell_type target column in adata.obs
- Custom benchmark scripts from the scaLR comparison GitHub repository
- Configuration file (config.yaml) specifying device (cuda/cpu), model layers, epochs, batch sizes, and split ratios
- Held-out test sets for evaluation

## Outputs

- Per-tool accuracy, precision, recall, and f1-score from scalr.analysis.evaluation
- Per-tool runtime and peak memory consumption measurements
- Aggregated benchmark_comparison_table.csv with accuracy, runtime, memory, and derived cost metrics
- Cost-efficiency comparison figure across scaLR, scvi-tools, and CellTypist

## How to apply

Obtain each dataset (e.g. PBMCs-SLE from Cellxgene and PBMCs-HED from the Broad Institute Single Cell Portal) in .h5ad/anndata form and the custom benchmark scripts from the scaLR comparison GitHub repository. Preprocess all datasets identically with subset-wise preprocessing (SampleNorm/StandardScaler) and backed-mode loading so memory stays low, then train the scaLR DNN via scalr.model_training_pipeline on all features and on the top 3,500 features, reading training data from disk in batches while logging training time and peak memory (use the -m/--memoryprofiler flag). Run scvi-tools and CellTypist pipelines through the comparison repository's custom scripts on equivalent hardware, recording accuracy, runtime, and memory for each. Evaluate all models on held-out test sets with scalr.analysis.evaluation, computing precision, recall, f1-score, and accuracy. Aggregate accuracy, runtime, memory, and derived cost metrics into a comparison table (benchmark_comparison_table.csv) and a cost-efficiency figure; treat the reproduction as successful if scaLR reaches accuracy comparable to scvi-tools and CellTypist while using less training time and computational resources.

## Related tools

- **scaLR** (DNN-based annotation pipeline under benchmark; its data_ingestion_pipeline, model_training_pipeline, and analysis.evaluation modules provide the accuracy and resource measurements) — https://github.com/infocusp/scaLR
- **scvi-tools** (Gold-standard comparison tool run via custom benchmark scripts; its accuracy, runtime, and memory are recorded on the same datasets and hardware)
- **CellTypist** (Gold-standard comparison tool run via custom benchmark scripts; its accuracy, runtime, and memory are recorded on the same datasets and hardware)

## Examples

```
python pipeline.py --config /path/to/config.yaml -l -m
```

## Evaluation signals

- scaLR attains accuracy (and precision/recall/f1) comparable to scvi-tools and CellTypist on the held-out test sets
- scaLR requires less model training time and computational resources than the Python-based comparison pipelines, consistent with the published finding
- Metrics are computed with the same evaluation module (scalr.analysis.evaluation) on identical test splits for all tools
- All tools were run on equivalent hardware, so runtime and peak-memory comparisons are attributable to the software rather than the environment
- benchmark_comparison_table.csv contains rows for each tool with non-empty accuracy, runtime, and memory values

## Limitations

- Memory usage is inversely proportional to execution time: reducing memory consumption may increase processing time
- Specific DNN hyperparameters (layers, epochs) for the final runs were examined only via sensitivity analysis without reported values
- Comparison tool versions are not stated; feature importances cannot be extracted from several tools (scBalance, scvi, devCellPy, CellPLM), limiting comparability of biomarker rankings

## Evidence

- [readme] The trained model is evaluated using the test dataset by calculating metrics such as precision, recall, f1-score, and accuracy.: [evidence span withheld — anchor status: external_doc]
