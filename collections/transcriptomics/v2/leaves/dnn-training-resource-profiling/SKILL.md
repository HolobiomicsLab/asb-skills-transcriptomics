---
name: dnn-training-resource-profiling
description: Use when benchmarking how much training time, runtime, and memory a cell-type
  annotation pipeline consumes on large (>1 million cell) scRNA-seq datasets, especially
  to compare low-resource (CPU-only) tools against GPU-dependent pipelines.
when_to_use_negative:
- Input datasets are small enough to fit in memory on a GPU-equipped machine, so resource-constrained
  batching adds overhead without benefit
- The goal is biomarker discovery or DGE analysis rather than measuring annotation
  pipeline resource consumption
- The annotation tool cannot expose feature importance or be profiled through the
  scaLR comparison scripts (e.g. scBalance, CellPLM), limiting comparability
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3316
tools:
- name: scaLR
  role: Primary platform whose DNN training pipeline is profiled for training time
    and peak memory on large datasets
  repo: https://github.com/infocusp/scaLR
- name: scvi-tools
  role: Gold-standard comparison tool benchmarked for accuracy, runtime, and memory
    on the same PBMCs datasets
- name: CellTypist
  role: Gold-standard comparison tool benchmarked for accuracy, runtime, and memory
    under equivalent hardware
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbaf243
    title: 'scaLR: a low-resource deep neural network-based platform for single cell
      analysis and biomarker discovery'
  source_content_hash: sha256:8073719d1b0487fa826b22f8e48cf8d8c6947a763253857e75183616781f0403
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
  - scalr.model_training_pipeline
  - scvi-tools
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# dnn-training-resource-profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Profiles training time and peak memory consumption of scaLR's low-resource DNN cell-type classification pipeline (and comparison tools like scvi-tools and CellTypist) on million-plus-cell scRNA-seq datasets to demonstrate cost-efficiency on standard CPU-only machines.

## When to use

Use when benchmarking how much training time, runtime, and memory a cell-type annotation pipeline consumes on large (>1 million cell) scRNA-seq datasets, especially to compare low-resource (CPU-only) tools against GPU-dependent pipelines.

## When NOT to use

- Input datasets are small enough to fit in memory on a GPU-equipped machine, so resource-constrained batching adds overhead without benefit
- The goal is biomarker discovery or DGE analysis rather than measuring annotation pipeline resource consumption
- The annotation tool cannot expose feature importance or be profiled through the scaLR comparison scripts (e.g. scBalance, CellPLM), limiting comparability

## Inputs

- scRNA-seq datasets in anndata .h5ad format (e.g. PBMCs-SLE with >1.25M cells, PBMCs-HED with >2M cells, sourced from Cellxgene and the Broad Institute Single Cell Portal)
- scaLR config.yaml specifying device ('cpu' or 'cuda'), batch sizes, epochs, and model layers
- Custom benchmark scripts from the scaLR comparison GitHub repository for scvi-tools and CellTypist

## Outputs

- Recorded training time and peak memory consumption per tool and per feature regime (all features vs. top 3,500 features)
- benchmark_comparison_table.csv aggregating accuracy, runtime, memory, and derived cost metrics
- A cost-efficiency comparison figure across scaLR, scvi-tools, and CellTypist

## How to apply

Run the scaLR data_ingestion_pipeline with subset-wise preprocessing (scalr.data.preprocess.sample_norm and scalr.data.preprocess.standard_scale) and backed-mode loading so data are read from disk in batches rather than held in memory. Train the DNN classifier via scalr.model_training_pipeline both on all features and on the top 3,500 features, recording training time and peak memory consumption at each stage. Launch the pipeline with `python pipeline.py --config /path/to/config.yaml -l -m`, where the `-m | --memoryprofiler` flag logs memory usage across pipeline stages. Run comparison tools (scvi-tools, CellTypist) through the scaLR comparison repository's custom scripts on equivalent hardware, recording their accuracy, runtime, and memory. Note the paper's trade-off: memory usage is inversely proportional to execution time, so reducing memory consumption may increase processing time.

## Related tools

- **scaLR** (Primary platform whose DNN training pipeline is profiled for training time and peak memory on large datasets) — https://github.com/infocusp/scaLR
- **scvi-tools** (Gold-standard comparison tool benchmarked for accuracy, runtime, and memory on the same PBMCs datasets)
- **CellTypist** (Gold-standard comparison tool benchmarked for accuracy, runtime, and memory under equivalent hardware)

## Examples

```
python pipeline.py --config /path/to/config.yaml -l -m
```

## Evaluation signals

- scaLR attains accuracy comparable to scvi-tools and CellTypist (evaluated via scalr.analysis.evaluation on precision, recall, f1-score, and accuracy)
- scaLR requires less model training time and computational resources than scvi-tools and CellTypist despite similar classification accuracy
- scaLR completes runs on a standard CPU-only machine (device: 'cpu') without specialized GPU resources
- Backed-mode, batch-wise training keeps peak memory bounded (memory usage is inversely proportional to execution time), which can be verified with the -m memoryprofiler log
- benchmark_comparison_table.csv contains rows for all three tools with accuracy, runtime, memory, and derived cost columns populated

## Limitations

- Memory usage is inversely proportional to execution time, meaning reducing memory consumption may increase processing time
- Performance may vary based on the input data format, requiring users to ensure proper data preprocessing (anndata .h5ad with log1p-normalized adata.X) for optimal results
- Specific DNN hyperparameters (epochs, layers) used in the final benchmark runs are only referenced via sensitivity analysis without reported values, and comparison tool versions are not stated
- Feature importance cannot be extracted from several comparison tools (scBalance, scvi, devCellPy, CellPLM), limiting comparability of biomarker ranking

## Evidence

- [results] scaLR was the most cost-efficient of the three tools: its low memory requirements and ability to run without a GPU let it operate on a standard machine: [evidence span withheld — anchor status: not_found]
- [readme] `python pipeline.py --config /path/to/config.yaml -l -m` with the -m flag to analyze memory usage: [evidence span withheld — anchor status: external_doc]
