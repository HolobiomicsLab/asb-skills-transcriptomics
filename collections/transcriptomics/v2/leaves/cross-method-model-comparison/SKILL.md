---
name: cross-method-model-comparison
description: Use when a new classifier (e.g. Pollock) must be evaluated head-to-head
  against multiple established methods on identical annotated single-cell datasets
  with predefined train/validation splits, using F1 as the comparison metric.
when_to_use_negative:
- The goal is to apply a pretrained Pollock module to new data rather than compare
  classification methods — use prediction mode instead
- Datasets lack annotated cell type labels, so no supervised train/validation evaluation
  with F1 is possible
- Only one classifier is of interest and no cross-method comparison is needed
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Pollock
  role: Method under evaluation; trained in train mode with paper hyperparameters
    on each of the 12 benchmarking datasets
  repo: github.com/ding-lab/pollock
- name: PyTorch
  role: Backend framework Pollock's VAE classifier is built and trained on
- name: scanpy
  role: Single-cell data handling for .h5ad inputs and baseline classifier context
    in the benchmark comparison
- name: Conda
  role: Environment setup for installing Pollock and its dependencies (pytorch, captum,
    scanpy, umap-learn, r-base)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbac028
    title: 'Pollock: fishing for cell states'
  source_content_hash: sha256:3ce21b2305dffa28f1e7de127344f3fad877e416bec442adac245e9644fb761b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Conda
  - Pollock
  - PyTorch
  - scanpy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cross-method-model-comparison

## Summary

Benchmarks Pollock's cell type classification (a VAE with a linear classification head) against established classifiers — Seurat, Scanpy, SingleCellNet, ACTINN, linear SVM, and MLP — on the same train/validation splits of 12 single-cell datasets, computing per-cell-type and macro-averaged F1 scores.

## When to use

Use when a new classifier (e.g. Pollock) must be evaluated head-to-head against multiple established methods on identical annotated single-cell datasets with predefined train/validation splits, using F1 as the comparison metric.

## When NOT to use

- The goal is to apply a pretrained Pollock module to new data rather than compare classification methods — use prediction mode instead
- Datasets lack annotated cell type labels, so no supervised train/validation evaluation with F1 is possible
- Only one classifier is of interest and no cross-method comparison is needed

## Inputs

- benchmarking_datasets.tar.gz (Zenodo record 5895221) containing 12 annotated single-cell datasets with train/validation splits
- Raw (unnormalized) count matrices for Pollock inputs (Seurat .RDS @assays$RNA@counts, scanpy .h5ad .X raw counts, or 10x cellranger matrices)

## Outputs

- benchmark_f1_comparison.csv: method-by-dataset F1 table (per-cell-type and macro-averaged)
- Trained Pollock and baseline classifier models on each dataset's train split
- Cell type predictions on each validation split from all seven methods

## How to apply

Download benchmarking_datasets.tar.gz from Zenodo record 5895221 and extract the 12 datasets (6 scRNA-seq, 3 snRNA-seq, 3 snATAC-seq gene activity) with annotated cell type labels and train/validation splits. Train the Pollock module in train mode with the paper hyperparameters (epochs=20, batch size=64, latent dim=64, encoder-out dim=128, middle dim=512, lr=1e-4, kl-scaler=1e-3, zinb-scaler=0.5, clf-scaler=1.0), splitting via n-per-cell-type=500; note Pollock inputs must be raw (unnormalized) counts. Train the six baseline classifiers (Seurat, Scanpy, SingleCellNet, ACTINN, linear SVM, MLP) on the same splits. Predict cell type labels on each validation split with every model, then compute per-cell-type and macro-averaged F1 scores per method per dataset and assemble the method-by-dataset table (benchmark_f1_comparison.csv).

## Related tools

- **Pollock** (Method under evaluation; trained in train mode with paper hyperparameters on each of the 12 benchmarking datasets) — github.com/ding-lab/pollock
- **PyTorch** (Backend framework Pollock's VAE classifier is built and trained on)
- **scanpy** (Single-cell data handling for .h5ad inputs and baseline classifier context in the benchmark comparison)
- **Conda** (Environment setup for installing Pollock and its dependencies (pytorch, captum, scanpy, umap-learn, r-base))

## Examples

```
pollock train from_scanpy --module-filepath <output_module> --scanpy-h5ad-filepath <dataset.h5ad> --cell-type-key cell_type --epochs 20 --batch-size 64 --latent-dim 64 --encoder-out-dim 128 --middle-dim 512 --lr 1e-4 --kl-scaler 1e-3 --zinb-scaler 0.5 --clf-scaler 1.0 --n-per-cell-type 500
```

## Evaluation signals

- Pollock is within 0.02 F1 of the top-performing tool in 11 out of 12 benchmarking datasets
- F1 scores are computed per cell type and macro-averaged for every method on each of the 12 datasets
- All methods are trained and evaluated on the identical benchmarking train/validation splits; Pollock receives raw (unnormalized) counts
- Pollock hyperparameters match the paper settings (epochs=20, kl-scaler=1e-3, zinb-scaler=0.5, clf-scaler=1.0, latent dim=64, n-per-cell-type=500)

## Limitations

- Benchmark covers only the 12 provided datasets (6 scRNA-seq, 3 snRNA-seq, 3 snATAC-seq gene activity); snATAC-seq data is represented as gene activity
- Training splits use n-per-cell-type=500, with oversampling when a cell type has fewer than N cells to balance the training dataset
- Pollock inputs must be raw counts; supplying normalized data would invalidate the comparison

## Evidence

- [abstract] Pollock performs comparably to existing classification methods, while offering easily deployable pretrained classification models across a wide variety of tissue and data types.: "Pollock performs comparably to existing classification methods, while offering easily deployable pretrained classification models across a wide variety of tissue and data types."
- [methods] Pollock is a VAE with linear classification head: "Pollock uses deep learning to make cell type predictions. At it's core, pollock is a variational autoencoder (VAE) with a linear classification head."
- [methods] Zenodo models and training datasets location: "The models and training datasets can be found on Zenodo at https://zenodo.org/record/5895221 in pretrained_models.tar.gz and benchmarking_datasets.tar.gz"
- [readme] Validation split methodology via n-per-cell-type: "N_PER_CELL_TYPE cells will be partitioned into training dataset for each cell type. If less than N_PER_CELL_TYPE cells exist for a cell type than cells are oversampled"
- [readme] Raw counts requirement for h5ad input: "scanpy expression matrix (.X attribute in the anndata object) must be raw expression counts (i.e. not normalized)"
