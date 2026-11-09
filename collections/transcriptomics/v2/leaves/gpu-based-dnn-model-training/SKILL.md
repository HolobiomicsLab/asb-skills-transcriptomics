---
name: gpu-based-dnn-model-training
description: Use when a .h5ad anndata scRNA-seq dataset must be classified (e.g. cell_type
  or disease target) by a DNN on constrained hardware (e.g. 12GB GPU / 128GB RAM),
  where memory-heavy pipelines like scVI-tools fail on full feature sets (e.g. 22,858
  features).
when_to_use_negative:
- The goal is unsupervised representation learning or generative modeling (e.g. scVI-style
  latent embeddings) rather than supervised classification with labeled targets
- Data is not in anndata .h5ad format with a labeled target column — scaLR requires
  prior preprocessing into this format
- Feature importances must be extracted from a comparison model that does not expose
  them (scBalance, scVI-tools, devCellPy, cellPLM) — biomarker ranking comparability
  is limited
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: scaLR
  role: Provides the DNN training pipeline (SequentialModel, SimpleModelTrainer, SimpleDataLoader,
    EarlyStopping, ModelCheckpoint) run via pipeline.py on GPU
  repo: github.com/infocusp/scaLR
- name: Python
  role: Implementation language; scaLR is a Python-based platform tested in Python
    3.10.20
  repo: github.com/infocusp/scaLR
- name: scVI-tools
  role: Comparison pipeline; could not process all 22,858 features at once (>128GB
    RAM needed), motivating the low-resource DNN training approach
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf243
    title: 'scaLR: a low-resource deep neural network-based platform for single cell
      analysis and biomarker discovery'
  source_content_hash: sha256:cd588e07b4359be03d820840a1eefb3b5fd9c93403fad55d448128e2a92331b6
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
  - scVI-tools
  - scaLR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# gpu-based-dnn-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains a Deep Neural Network (SequentialModel via scaLR) on single-cell expression data in batch-wise, disk-backed reads on GPU ('cuda') or CPU, with early stopping and checkpointing, for cell type/state annotation under low-resource constraints.

## When to use

Use when a .h5ad anndata scRNA-seq dataset must be classified (e.g. cell_type or disease target) by a DNN on constrained hardware (e.g. 12GB GPU / 128GB RAM), where memory-heavy pipelines like scVI-tools fail on full feature sets (e.g. 22,858 features).

## When NOT to use

- The goal is unsupervised representation learning or generative modeling (e.g. scVI-style latent embeddings) rather than supervised classification with labeled targets
- Data is not in anndata .h5ad format with a labeled target column — scaLR requires prior preprocessing into this format
- Feature importances must be extracted from a comparison model that does not expose them (scBalance, scVI-tools, devCellPy, cellPLM) — biomarker ranking comparability is limited

## Inputs

- anndata .h5ad scRNA-seq file (adata.X with log1p-normalized values in range 0-10; adata.obs containing a 'target' column such as cell_type or disease; adata.var gene names)
- scaLR config.yaml specifying device, model layers, optimizer, loss, epochs, batch_size, and callbacks

## Outputs

- Trained DNN model checkpoint (ModelCheckpoint, resume_from_checkpoint supported)
- Per-class predictions on the test set with precision, recall, f1-score, and accuracy
- Resource usage logs: peak memory (memoryprofiler) and execution time for benchmark comparison (e.g. pbmcs_bs_benchmark_comparison.csv)

## How to apply

Set device to 'cuda' (GPU) or 'cpu' in the scaLR config.yaml and run the final_training stage: a SequentialModel is trained on the training set with batch-wise SimpleDataLoader reads directly from disk (e.g. batch_size 15000-25000 with padding 5000), so all samples are consumed without loading the full matrix. Choose the output layer to match the number of classes (e.g. layers [5000, 10] for 10 cell types, [5000, 2] for disease/normal), and train with CrossEntropyLoss and an optimizer such as SGD (lr 1.0e-3, weight_decay 0.1) or Adam (lr 1.0e-3) for up to 100 epochs with early stopping (patience 3, min_delta 1.0e-4) and ModelCheckpoint every 5 intervals. Validation is evaluated at each epoch, and TensorboardLogger records training. For the benchmark workflow, run on the PBMCs-BS dataset logging peak memory (via -m/--memoryprofiler) and wall-clock time, then compare against scVI-tools, SingleCellNet, CellTypist, ACTINN, devCellPy, scBalance, and cellPLM scripts from the comparison repository.

## Related tools

- **scaLR** (Provides the DNN training pipeline (SequentialModel, SimpleModelTrainer, SimpleDataLoader, EarlyStopping, ModelCheckpoint) run via pipeline.py on GPU) — github.com/infocusp/scaLR
- **Python** (Implementation language; scaLR is a Python-based platform tested in Python 3.10.20) — github.com/infocusp/scaLR
- **scVI-tools** (Comparison pipeline; could not process all 22,858 features at once (>128GB RAM needed), motivating the low-resource DNN training approach)

## Examples

```
python pipeline.py --config /path/to/config.yaml -l -m  # config with device: 'cuda', SequentialModel layers [5000, 10], Adam lr 1.0e-3, epochs 100, EarlyStopping patience 3
```

## Evaluation signals

- Model validation loss/metric decreases each epoch and EarlyStopping (patience 3, min_delta 1.0e-4) fires only after genuine plateau; TensorboardLogger shows a stable training curve
- Test-set evaluation yields precision, recall, f1-score, and accuracy comparable to comparison pipelines (scVI-tools, CellTypist, ACTINN, etc.) on the same test set
- Peak memory stays within the 128GB RAM / 12GB GPU VM budget where scVI-tools required >128GB for the full 22,858-feature set
- Output layer dimension matches the number of target classes (e.g. 10 for cell_type classes in the example config, 2 for disease/normal)
- Identical resource metrics (memory and wall-clock time) are logged for scaLR and all compared pipelines to fill the benchmark comparison table

## Limitations

- Memory usage is inversely proportional to execution time — reducing memory consumption may increase processing time
- Performance may vary based on the input data format, requiring users to ensure proper data preprocessing for optimal results
- Specific hyperparameters (epochs, DNN layers) were only assessed via sensitivity analysis without reported final values in the paper text
- The benchmark Tables 2–4 with exact accuracy/memory/time values were referenced but not present in the extracted text

## Evidence

- [readme] Device setup: [evidence span withheld — anchor status: external_doc]
- [readme] DNN training with validation and early stopping: [evidence span withheld — anchor status: external_doc]
