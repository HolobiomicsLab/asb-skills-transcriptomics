---
name: memory-runtime-cost-comparison-across-pipelines
description: Use when choosing an annotation pipeline for million-cell scRNA-seq datasets
  under hardware or budget constraints, and you need measured accuracy, memory, runtime,
  and cost trade-offs across tools.
when_to_use_negative:
- Input is small (<1M cells) and runs comfortably on available hardware — the cost/memory
  dimension of the comparison becomes non-discriminative
- The task is biomarker/feature extraction rather than annotation cost benchmarking
  — comparison tools like scvi may not expose extractable top features
- Datasets cannot be converted to anndata .h5ad with the expected adata.X / adata.obs
  / adata.var layout required by scaLR
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scaLR
  role: Low-resource DNN annotation pipeline being benchmarked; runs data ingestion,
    training, and evaluation with low memory
  repo: github.com/infocusp/scaLR
- name: scvi-tools
  role: Baseline annotation pipeline requiring specialized computing resources (GPU)
- name: CellTypist
  role: Baseline annotation pipeline requiring specialized computing resources
- name: Python
  role: Implementation language for scaLR and the custom benchmark scripts
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbaf243
    title: 'scaLR: a low-resource deep neural network-based platform for single cell
      analysis and biomarker discovery'
  source_content_hash: sha256:3c6b99c2d1f743bc2fbcd607129e9e15137354b40427e38c92fb1df7cefb5232
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/infocusp/scaLR
  tools:
  - CellTypist
  - Python
  - scaLR
  - scvi-tools
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# memory-runtime-cost-comparison-across-pipelines

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill for comparing cell-type annotation pipelines (scaLR vs scvi-tools vs CellTypist) on very large scRNA-seq datasets (>1.25M cells) by jointly recording classification accuracy, peak memory consumption, runtime, and derived cost metrics. It grounds tool-choice decisions in measured resource efficiency rather than accuracy alone.

## When to use

Use when choosing an annotation pipeline for million-cell scRNA-seq datasets under hardware or budget constraints, and you need measured accuracy, memory, runtime, and cost trade-offs across tools.

## When NOT to use

- Input is small (<1M cells) and runs comfortably on available hardware — the cost/memory dimension of the comparison becomes non-discriminative
- The task is biomarker/feature extraction rather than annotation cost benchmarking — comparison tools like scvi may not expose extractable top features
- Datasets cannot be converted to anndata .h5ad with the expected adata.X / adata.obs / adata.var layout required by scaLR

## Inputs

- PBMCs-SLE (>1.25M cells) and PBMCs-HED (>2M cells) scRNA-seq datasets in anndata .h5ad format with a 'target'/cell_type column in adata.obs
- Custom benchmark scripts from the scaLR comparison GitHub repository
- Equivalent hardware environment for all three pipelines (CPU-only allowed for scaLR)

## Outputs

- benchmark_comparison_table.csv with accuracy, runtime, peak memory, and derived cost metrics per tool
- Cost-efficiency comparison figure
- Per-tool classification metrics (precision, recall, f1-score, accuracy) on held-out test sets

## How to apply

Run each candidate tool (scaLR, scvi-tools, CellTypist) on the same PBMCs-SLE / PBMCs-HED datasets sourced from Cellxgene and the Broad Institute Single Cell Portal, using the custom benchmark scripts from the scaLR comparison GitHub repository. Preprocess with scaLR's data-ingestion pipeline (scalr.data_ingestion_pipeline) using subset-wise normalization (scalr.data.preprocess.sample_norm, scalr.data.preprocess.standard_scale) and backed-mode loading to keep memory low; train the DNN classifier via scalr.model_training_pipeline on all features and on the top 3,500 features, reading training data from disk in batches while recording training time and peak memory. Run scvi-tools and CellTypist through the comparison repository's custom scripts under equivalent hardware with the same logging. Evaluate all models on held-out test sets with scalr.analysis.evaluation (precision, recall, f1-score, accuracy), then aggregate accuracy, runtime, memory, and derived cost metrics into benchmark_comparison_table.csv and a cost-efficiency figure. The skill succeeds when the low-resource tool attains comparable accuracy while using less training time and computational resources (e.g., CPU-only operation) than the specialized-resource baselines.

## Related tools

- **scaLR** (Low-resource DNN annotation pipeline being benchmarked; runs data ingestion, training, and evaluation with low memory) — github.com/infocusp/scaLR
- **scvi-tools** (Baseline annotation pipeline requiring specialized computing resources (GPU))
- **CellTypist** (Baseline annotation pipeline requiring specialized computing resources)
- **Python** (Implementation language for scaLR and the custom benchmark scripts)

## Examples

```
python pipeline.py --config /path/to/config.yaml -l -m
```

## Evaluation signals

- scaLR attains accuracy comparable to scvi-tools and CellTypist (similar precision/recall/f1/accuracy) on held-out test sets
- scaLR requires less model training time and computational resources than the baselines, matching the reported finding
- scaLR runs on a standard machine without a GPU while baselines require specialized resources
- Peak memory and runtime are recorded for every tool and populate benchmark_comparison_table.csv without missing entries
- Evaluation is performed on held-out test sets via scalr.analysis.evaluation, not on training data

## Limitations

- Memory usage is inversely proportional to execution time — reducing memory consumption may increase processing time
- scaLR performance may vary based on input data format, requiring users to ensure proper data preprocessing
- Comparison tool versions are not stated; benchmark scripts are in the scaLR comparison GitHub repository
- Benchmark result tables (Tables 2–5 content) are not present in the extracted text, so exact numeric baselines must be taken from the repository scripts or original tables

## Evidence

- [other] its low memory requirements and ability to run without a GPU let it operate on a standard machine, whereas scVI-tools and CellTypist required specialized computing resources, despite similar classification accuracy: [evidence span withheld — anchor status: not_found]
- [readme] README: python pipeline.py --config /path/to/config.yaml -l -m: [evidence span withheld — anchor status: external_doc]
