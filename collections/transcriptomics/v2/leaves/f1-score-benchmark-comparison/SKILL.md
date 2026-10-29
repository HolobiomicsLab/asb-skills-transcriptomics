---
name: f1-score-benchmark-comparison
description: Use when comparing a generalized model trained on merged multi-dataset
  training data against disease-specific counterparts on each cancer type's validation
  set, using macro-averaged F1 deltas.
when_to_use_negative:
- Evaluating a single disease-specific model in isolation with no generalized counterpart
  to compare against — there is no delta to compute
- Inputs are already aggregated F1 score tables rather than per-cell predictions —
  you would only compute the subtraction, not the full training/prediction pipeline
- Classifying normal (non-cancer) tissue data — the benchmarking workflow removes
  normal epithelial cells and validates on diseased-held-out sets
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: Pollock
  role: Trains generalized modules (train mode) and produces cell type predictions
    on validation sets (predict mode) used for F1 computation
  repo: https://github.com/ding-lab/pollock
- name: PyTorch
  role: Backend framework for Pollock's VAE classifier installed via conda
- name: scanpy
  role: Reads .h5ad anndata validation objects passed to pollock train/predict from_scanpy
- name: Conda
  role: Environment management for installing Pollock and its dependencies
- name: Seurat
  role: R object format (.rds) for disease-specific training data and validation sets
    processed via pollock from_seurat
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbac028
    title: 'Pollock: fishing for cell states'
  source_content_hash: sha256:102f966cc669cb28ddf7b72b257333f144648def0d997010add691477a40439a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ding-lab/pollock
  tools:
  - Conda
  - Pollock
  - PyTorch
  - Seurat
  - scanpy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# f1-score-benchmark-comparison

## Summary

Benchmarking workflow for quantifying how well a generalized pan-cancer Pollock model performs relative to disease-specific modules, by computing macro-averaged F1 scores per cancer type on held-out validation data and subtracting disease-specific F1 to obtain per-cancer-type deltas.

## When to use

Use when comparing a generalized model trained on merged multi-dataset training data against disease-specific counterparts on each cancer type's validation set, using macro-averaged F1 deltas.

## When NOT to use

- Evaluating a single disease-specific model in isolation with no generalized counterpart to compare against — there is no delta to compute
- Inputs are already aggregated F1 score tables rather than per-cell predictions — you would only compute the subtraction, not the full training/prediction pipeline
- Classifying normal (non-cancer) tissue data — the benchmarking workflow removes normal epithelial cells and validates on diseased-held-out sets

## Inputs

- benchmarking_datasets.tar.gz (Zenodo record 5895221) — per-cancer-type disease-specific training datasets for scRNA-seq, snRNA-seq, snATAC-seq
- pretrained_models.tar.gz (Zenodo record 5895221) — disease-specific and generalized Pollock modules
- Seurat .rds or scanpy .h5ad objects with cell_type labels in metadata/.obs (raw counts)
- Diseased-held-out validation sets per cancer type

## Outputs

- generalized_vs_disease_specific_f1.csv — per-cancer-type macro-averaged F1 difference summary
- Prediction tables from pollock predict mode (output.rds / output.h5ad or tab-separated txt files)
- Trained generalized modules per data modality

## How to apply

Download benchmarking_datasets.tar.gz and pretrained_models.tar.gz from Zenodo record 5895221, then for each modality (scRNA-seq, snRNA-seq, snATAC-seq as gene activity) merge all disease-specific training datasets into one training set per modality, removing normal epithelial cells from the combined training data. Train each generalized module with the Pollock CLI in train mode (pollock train from_seurat/from_scanpy) using default hyperparameters: 20 epochs, latent-dim 64, encoder-out-dim 128, middle-dim 512, n-per-cell-type 500, kl-scaler 1e-3. Run Pollock in predict mode on each cancer type's validation set, saving predictions to output.rds/output.h5ad or a tab-separated txt file. Load the diseased-held-out prediction tables, compute macro-averaged F1 per cancer type for the generalized modules, and subtract the disease-specific modules' F1 scores to yield per-cancer-type differences, writing the summary to generalized_vs_disease_specific_f1.csv. Validation is confirmed when reproduced F1 deltas fall within the reported spread — 8 of 12 cancer-type comparisons within 0.30 F1 of their disease-specific counterpart (best: PDAC scRNA-seq 0.03, BRCA snRNA-seq/snATAC-seq 0.06/0.02; worst: CESC scRNA-seq 0.31, GBM snRNA-seq/snATAC-seq 0.52/0.43).

## Related tools

- **Pollock** (Trains generalized modules (train mode) and produces cell type predictions on validation sets (predict mode) used for F1 computation) — https://github.com/ding-lab/pollock
- **PyTorch** (Backend framework for Pollock's VAE classifier installed via conda)
- **scanpy** (Reads .h5ad anndata validation objects passed to pollock train/predict from_scanpy)
- **Conda** (Environment management for installing Pollock and its dependencies)
- **Seurat** (R object format (.rds) for disease-specific training data and validation sets processed via pollock from_seurat)

## Examples

```
pollock train from_scanpy --module-filepath scRNAseq_generalized --scanpy-h5ad-filepath merged_training.h5ad --cell-type-key cell_type --epochs 20 --latent-dim 64 --middle-dim 512 --n-per-cell-type 500 --kl-scaler 0.001
```

## Evaluation signals

- Reproduced per-cancer-type F1 deltas fall within the reported spread: 8 of 12 cancer-type comparisons are within 0.30 F1 of the disease-specific counterpart
- PDAC scRNA-seq delta ≈ 0.03, BRCA snRNA-seq/snATAC-seq deltas ≈ 0.06/0.02, CESC scRNA-seq ≈ 0.31, GBM snRNA-seq/snATAC-seq ≈ 0.52/0.43
- Generalized training sets per modality contain no normal epithelial cells after filtering
- Prediction tables cover all cells of each diseased-held-out validation set and are loadable as tab-separated txt or embedded in output.rds/output.h5ad
- Generalized modules trained with default hyperparameters (20 epochs, latent-dim 64, encoder-out-dim 128, middle-dim 512, n-per-cell-type 500, kl-scaler 1e-3)

## Limitations

- Generalized models perform somewhat worse than disease-specific counterparts; some cancer types (e.g. CESC scRNA-seq, GBM snRNA-seq/snATAC-seq) generalize most poorly with F1 differences of 0.31 to 0.52
- snATAC-seq data must be represented as gene activity for classification
- If fewer than N_PER_CELL_TYPE (500) cells exist for a cell type, cells are oversampled to balance the training dataset, which may affect F1 comparability

## Evidence

- [results] Generalized models perform somewhat worse than disease-specific counterparts, but 8/12 classify most cell types correctly within an F1-score of 0.30: [evidence span withheld — anchor status: not_found]
- [methods] The models and training datasets can be found on Zenodo at https://zenodo.org/record/5895221 in pretrained_models.tar.gz and benchmarking_datasets.tar.gz: "The models and training datasets can be found on Zenodo at https://zenodo.org/record/5895221 in pretrained_models.tar.gz and benchmarking_datasets.tar.gz"
- [readme] N_PER_CELL_TYPE cells will be partitioned into training dataset for each cell type. If less than N_PER_CELL_TYPE cells exist for a cell type than cells are oversampled: "N_PER_CELL_TYPE cells will be partitioned into training dataset for each cell type. If less than N_PER_CELL_TYPE cells exist for a cell type than cells are oversampled to balance the training dataset."
- [readme] snATAC-seq data is represented as gene activity: "trained with disease-specific breast cancer dataset, snATAC-seq data is represented as gene activity"
- [readme] If included output will be written to a tab-seperated .txt file. Otherwise output will be saved in the metadata of the input seurat object (.rds) or scanpy anndata object (.h5ad): "If included output will be written to a tab-seperated .txt file. Otherwise output will be saved in the metadata of the input seurat object (.rds) or scanpy anndata object (.h5ad) depending on the"
