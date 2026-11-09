---
name: cross-cohort-dataset-scaling
description: Use when an scRNA-seq dataset exceeds available RAM (e.g. >0.6M cells
  on a 128GB machine, or ~11.5M-cell cross-cohort data) and you must run or benchmark
  annotation pipelines like scaLR, scBalance, scvi, devCellPy, or CellPLM under that
  limit.
when_to_use_negative:
- Dataset fits comfortably in RAM (e.g. small PBMCs-BS-style studies well under the
  128GB ceiling) — a standard annotation tool without batch partitioning is simpler
  and likely faster
- Input data is not in .h5ad/anndata format or adata.X lacks log1p-normalized values
  in the 0-10 range — scaLR performance varies with input format and requires prior
  preprocessing
- The goal is biomarker/feature ranking from comparison tools (scBalance, scvi, devCellPy,
  CellPLM), since top features cannot be extracted from those models
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: scaLR
  role: 'Primary platform: runs data ingestion, feature extraction, model training,
    and evaluation pipelines with batch partitioning and backed-mode loading to annotate
    >11.5M-cell cross-cohort data under 128GB'
  repo: github.com/infocusp/scaLR
- name: scBalance
  role: Baseline annotation tool run via custom scripts in the scaLR comparison repository
    to test memory feasibility and accuracy on the same datasets
  repo: github.com/infocusp/scaLR
- name: scvi
  role: Baseline annotation tool run via custom scripts from the scaLR comparison
    repository under the 128GB memory limit
  repo: github.com/infocusp/scaLR
- name: devCellPy
  role: Baseline annotation tool run via custom scripts from the scaLR comparison
    repository; crashes on PBMCs-normal within 128GB
  repo: github.com/infocusp/scaLR
- name: cellPLM
  role: Baseline annotation tool and the only comparison tool besides scaLR able to
    complete on PBMCs-normal (>0.6M cells) within 128GB
  repo: github.com/infocusp/scaLR
- name: Python
  role: Runtime environment for all pipelines; scaLR tested on Python 3.10.20
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf243
    title: 'scaLR: a low-resource deep neural network-based platform for single cell
      analysis and biomarker discovery'
  source_content_hash: sha256:728469a3b0609c75dc4efdd0ad6ecfda9fc01f3e66647c29bf90a45967825f1a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/infocusp/scaLR
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
yes_rate: 0.6428571428571429
n_claims: 14
---

# cross-cohort-dataset-scaling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarking and executing cell type annotation pipelines on multi-million-cell, cross-cohort scRNA-seq datasets (sourced from Cellxgene and the Broad Single Cell Portal) under a fixed memory ceiling, using scaLR's batch partitioning and backed-mode loading to stay within ~128GB RAM.

## When to use

Use when an scRNA-seq dataset exceeds available RAM (e.g. >0.6M cells on a 128GB machine, or ~11.5M-cell cross-cohort data) and you must run or benchmark annotation pipelines like scaLR, scBalance, scvi, devCellPy, or CellPLM under that limit.

## When NOT to use

- Dataset fits comfortably in RAM (e.g. small PBMCs-BS-style studies well under the 128GB ceiling) — a standard annotation tool without batch partitioning is simpler and likely faster
- Input data is not in .h5ad/anndata format or adata.X lacks log1p-normalized values in the 0-10 range — scaLR performance varies with input format and requires prior preprocessing
- The goal is biomarker/feature ranking from comparison tools (scBalance, scvi, devCellPy, CellPLM), since top features cannot be extracted from those models

## Inputs

- Large-scale scRNA-seq datasets in anndata .h5ad format (e.g. PBMCs-normal >0.6M cells, PBMCs-SLE, cross-cohort ~11.5M-cell collections) from Cellxgene and the Broad Single Cell Portal
- Custom comparison scripts from the scaLR comparison GitHub repository
- scaLR config.yml specifying data paths, target (e.g. cell_type), splitters, model layers, and epochs

## Outputs

- benchmark_comparison.csv recording run completion/crash, peak memory usage, and runtime per tool-dataset pair
- Cell-type annotation metrics (precision, recall, f1-score, accuracy) on the test set for tools that complete
- scaLR experiment outputs (feature ranks, trained model checkpoints, evaluation plots) under scalr_experiments/<exp_name>/exp_run_N

## How to apply

Download the cross-cohort scRNA-seq benchmark datasets (e.g. PBMCs-SLE, PBMCs-normal with >0.6M cells) from Cellxgene and the Broad Single Cell Portal as .h5ad files. Run the four scaLR pipelines — scalr.data_ingestion_pipeline, scalr.feature_extraction_pipeline, scalr.model_training_pipeline, scalr.eval_and_analysis_pipeline — using batch partitioning (sample_chunksize, e.g. 20000) and backed-mode reads, with subset-wise preprocessing and training data read from disk in batches. Execute comparison tools (scBalance, scvi, devCellPy, cellPLM) via the custom scripts in the scaLR comparison GitHub repository under the same 128GB limit, recording completion/crash status, peak memory, and runtime for each tool-dataset pair. For completed runs, evaluate annotation accuracy on the test set with precision, recall, f1-score, and accuracy, then assemble results into benchmark_comparison.csv. Rationale: scaLR partitions datasets into manageable batches so annotation of millions of cells succeeds on resource-limited systems, whereas baseline tools crashed on PBMCs-normal and PBMCs-SLE at 128GB; expect memory-vs-runtime trade-offs since memory usage is inversely proportional to execution time.

## Related tools

- **scaLR** (Primary platform: runs data ingestion, feature extraction, model training, and evaluation pipelines with batch partitioning and backed-mode loading to annotate >11.5M-cell cross-cohort data under 128GB) — github.com/infocusp/scaLR
- **scBalance** (Baseline annotation tool run via custom scripts in the scaLR comparison repository to test memory feasibility and accuracy on the same datasets) — github.com/infocusp/scaLR
- **scvi** (Baseline annotation tool run via custom scripts from the scaLR comparison repository under the 128GB memory limit) — github.com/infocusp/scaLR
- **devCellPy** (Baseline annotation tool run via custom scripts from the scaLR comparison repository; crashes on PBMCs-normal within 128GB) — github.com/infocusp/scaLR
- **cellPLM** (Baseline annotation tool and the only comparison tool besides scaLR able to complete on PBMCs-normal (>0.6M cells) within 128GB) — github.com/infocusp/scaLR
- **Python** (Runtime environment for all pipelines; scaLR tested on Python 3.10.20)

## Examples

```
python pipeline.py --config config/config_celltype.yaml -l -m
```

## Evaluation signals

- benchmark_comparison.csv reproduces the paper's pattern: all baseline tools except cellPLM fail (OOM) on PBMCs-normal and PBMCs-SLE at 128GB, while scaLR completes
- scaLR processes the >11.5M-cell cross-cohort data to completion within the 128GB limit
- Completed runs' annotation accuracy (precision, recall, f1-score, accuracy) matches baseline-level accuracy while using less training time and memory
- Memory profiler output (-m | --memoryprofiler flag) shows peak usage staying under 128GB across pipeline stages
- Training runs read data batch-wise from disk with sample_chunksize partitioning rather than loading the full matrix, confirming backed-mode behavior

## Limitations

- Memory usage is inversely proportional to execution time — reducing memory consumption increases processing time
- Performance varies with input data format; users must ensure proper preprocessing (log1p normalization with range 0-10 in adata.X) for optimal results
- Comparison tool versions are not stated, and feature importance cannot be extracted from scBalance, scvi, devCellPy, or cellPLM, limiting biomarker comparability
- Sensitivity to hyperparameters (epochs, DNN layers) was tested but final-run values are not reported in the extracted text; Tables 2-4 were not present in the extracted text

## Evidence

- [other] On the PBMCs-normal dataset (>0.6 million cells, ~0.4 million training cells), only CellPLM could run, while all other tested tools required more than 128GB of memory: [evidence span withheld — anchor status: not_found]
- [readme] python pipeline.py --config /path/to/config.yaml -l -m: [evidence span withheld — anchor status: external_doc]
