---
name: resource-constrained-gpu-execution
description: Use when training or evaluating scRNA-seq annotation models on datasets
  with millions of cells where the VM has limited resources (e.g., 12GB GPU, 128GB
  RAM, 8 CPUs) and full in-memory processing would fail.
when_to_use_negative:
- The dataset and model already fit comfortably in GPU memory — batching and backed-mode
  loading add I/O overhead and increase processing time without benefit
- Input is not .h5ad anndata with counts normalized to the expected 0–10 log1p range
  — scaLR's performance may vary and require user preprocessing first
- The goal is feature-importance extraction from comparison tools (scBalance, scvi,
  devCellPy, CellPLM) that do not expose feature scores, since resource-bounded execution
  of those pipelines does not yield comparable biomarker rankings
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: scaLR
  role: Platform whose batch-wise dataloader, backed-mode data ingestion, and --memoryprofiler
    flag implement low-resource training and evaluation
  repo: github.com/infocusp/scaLR
- name: Python
  role: Runtime environment (tested with Python 3.10.20) in which pipeline.py, the
    config-driven stages, and profiling are executed
  repo: github.com/infocusp/scaLR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf243
    title: 'scaLR: a low-resource deep neural network-based platform for single cell
      analysis and biomarker discovery'
  source_content_hash: sha256:c8e5687d66b62e731d1bacf7e2b503f1b7ec841bfec0d9667a4958cc21347821
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
  - scaLR
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# resource-constrained-gpu-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run the scaLR DNN pipeline for scRNA-seq cell-type annotation on limited hardware (e.g., 12GB GPU / 128GB RAM / 8 CPU VM) by combining backed-mode .h5ad loading, subset-wise normalization, batch-wise disk reads, and memory profiling.

## When to use

Use when training or evaluating scRNA-seq annotation models on datasets with millions of cells where the VM has limited resources (e.g., 12GB GPU, 128GB RAM, 8 CPUs) and full in-memory processing would fail.

## When NOT to use

- The dataset and model already fit comfortably in GPU memory — batching and backed-mode loading add I/O overhead and increase processing time without benefit
- Input is not .h5ad anndata with counts normalized to the expected 0–10 log1p range — scaLR's performance may vary and require user preprocessing first
- The goal is feature-importance extraction from comparison tools (scBalance, scvi, devCellPy, CellPLM) that do not expose feature scores, since resource-bounded execution of those pipelines does not yield comparable biomarker rankings

## Inputs

- Annotated scRNA-seq dataset in anndata .h5ad format (adata.X with log1p-normalized counts in 0–10 range; target column such as cell_type in adata.obs)
- Pipeline config.yaml with device (cuda/cpu), dataloader batch_size, sample_chunksize, and feature-selection model layers set
- Optional top-K feature list (e.g., top 3,500 features from scaLR SGD-based feature extraction) to restrict training width

## Outputs

- Trained SequentialModel checkpoint produced under scalr_experiments/<exp_run> with early stopping applied
- Per-stage memory profile and wall-clock execution time captured via the --memoryprofiler flag
- Test-set evaluation metrics (precision, recall, f1-score, accuracy) from the scalr.analysis.evaluation module

## How to apply

Set device to 'cuda' (or 'cpu') in config.yaml, keep the input anndata .h5ad in backed mode, and enable subset-wise preprocessing (SampleNorm) so normalization is applied per chunk rather than on the full matrix. Load training data directly from disk in batches — configure the SimpleDataLoader batch_size (e.g., 15000–25000) and sample_chunksize (e.g., 20000) — so peak memory stays bounded while all samples and features are consumed. Configure the final training stage with EarlyStopping and ModelCheckpoint so epoch count adapts to the 100-epoch budget without overshooting GPU memory. Launch the pipeline with the --memoryprofiler flag (-m) to log per-stage memory usage and wall-clock time, and record these metrics alongside test-set accuracy from scalr.analysis.evaluation. Rationale: scaLR strategically partitions large datasets into manageable batches precisely so cell-type classification on >1M-cell datasets (PBMCs-BS, PBMCs-SLE) is feasible where existing Python pipelines exhaust GPU memory.

## Related tools

- **scaLR** (Platform whose batch-wise dataloader, backed-mode data ingestion, and --memoryprofiler flag implement low-resource training and evaluation) — github.com/infocusp/scaLR
- **Python** (Runtime environment (tested with Python 3.10.20) in which pipeline.py, the config-driven stages, and profiling are executed) — github.com/infocusp/scaLR

## Examples

```
python pipeline.py --config /path/to/config.yaml -l -m
```

## Evaluation signals

- Peak GPU/RAM usage logged with --memoryprofiler stays within the 12GB GPU / 128GB RAM budget of the VM throughout training
- Dataloader configuration confirms batch-wise disk reads (batch_size ~15000–25000, sample_chunksize 20000) rather than full-matrix loading
- Test-set accuracy via scalr.analysis.evaluation is comparable to existing Python-based pipelines while wall-clock training time and memory usage are lower
- EarlyStopping callback fired (or final-model checkpoint saved at interval 5) indicating the training loop terminated within resource bounds
- Input .h5ad verified as anndata with adata.X in the 0–10 log1p range and a target column present in adata.obs before launch

## Limitations

- Memory usage is inversely proportional to execution time — reducing memory consumption may increase processing time
- scaLR only accepts .h5ad anndata input; performance may vary based on the input data format, requiring users to ensure proper preprocessing
- Memory-management improvements and broader data-format compatibility are proposed future work, not yet implemented; the software version used is not stated

## Evidence

- [readme] Update device: 'cuda' for GPU enabled runtype, else device: 'cpu' for CPU enabled runtype.: [evidence span withheld — anchor status: external_doc]
- [readme] `python pipeline.py --config /path/to/config.yaml -l -m` to run the scaLR.: [evidence span withheld — anchor status: external_doc]
