---
name: single-cell-cell-type-classification
description: Use when you have raw (unnormalized) single-cell count data (Seurat .rds,
  scanpy .h5ad, or 10x cellranger output) and annotated cell type labels to train
  on, or a pretrained Pollock module to assign cell type labels to new cells.
when_to_use_negative:
- Input expression matrix is normalized/log-transformed — Pollock requires raw unnormalized
  counts (e.g. the tutorial keeps a separate counts_adata copy before sc.pp.normalize_total/log1p).
- Goal is differential expression, clustering, or other unsupervised analysis rather
  than supervised cell type label assignment with an annotated reference or pretrained
  module.
- You need classification of organisms/cell types outside the available pretrained
  cancer/PBMC modules and have no annotated training data to train a new module.
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Pollock
  role: VAE-based deep learning classifier providing train, predict, and explain modes
    for cell type identification
  repo: github.com/ding-lab/pollock
- name: PyTorch
  role: Deep learning backend for training and inference of the Pollock VAE model
- name: scanpy
  role: Loads and writes .h5ad AnnData inputs/outputs and used in the conda environment
    for single-cell data handling
- name: captum
  role: Interpretability library installed in the Pollock environment supporting feature/explainability
    scores
- name: Conda
  role: Environment creation for Pollock installation (conda create -n pollock -c
    pytorch -c conda-forge ...)
- name: Seurat
  role: R workflow from which .RDS objects with raw counts and cell type metadata
    are supplied to Pollock
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_005
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbac028
    title: 'Pollock: fishing for cell states'
  source_content_hash: sha256:0767e0aefaf53066540e46f676f543ef1b1b111bc9b07b28883455fb3c688dc2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Conda
  - Pollock
  - PyTorch
  - Seurat
  - captum
  - scanpy
  edam_operation: http://edamontology.org/operation_1777
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# single-cell-cell-type-classification

## Summary

Classify single-cell types/states using Pollock, a VAE-based deep learning method with a linear classification head, via its train/predict CLI or pretrained modules covering scRNA-seq, snRNA-seq, and snATAC-seq (gene activity) cancer datasets.

## When to use

Use when you have raw (unnormalized) single-cell count data (Seurat .rds, scanpy .h5ad, or 10x cellranger output) and annotated cell type labels to train on, or a pretrained Pollock module to assign cell type labels to new cells.

## When NOT to use

- Input expression matrix is normalized/log-transformed — Pollock requires raw unnormalized counts (e.g. the tutorial keeps a separate counts_adata copy before sc.pp.normalize_total/log1p).
- Goal is differential expression, clustering, or other unsupervised analysis rather than supervised cell type label assignment with an annotated reference or pretrained module.
- You need classification of organisms/cell types outside the available pretrained cancer/PBMC modules and have no annotated training data to train a new module.

## Inputs

- Seurat .RDS object with raw counts in @assays$RNA@counts and cell type labels in metadata (e.g. 'cell_type')
- scanpy .h5ad AnnData object with raw counts in .X and cell type key in .obs
- 10x cellranger output (mtx directory or raw_feature_bc_matrix.h5), predict mode only
- Pretrained Pollock module directory from Zenodo record 5895221 (pretrained_models.tar.gz)
- CLI hyperparameters: epochs=20, batch size=64, latent-dim 64, encoder-out-dim 128, middle-dim 512, lr=1e-4, kl-scaler 1e-3, n-per-cell-type=500

## Outputs

- Per-cell predicted cell type labels saved to output.rds / output.h5ad metadata or a tab-separated .txt file
- Trained Pollock module directory (--module-filepath) for reuse in predict/explain modes
- UMAP embedding of VAE latent space incorporated into the predicted object (unless --no-umap)
- Macro-averaged and per-cell-type F1 scores when predictions are compared to annotated labels

## How to apply

Install Pollock from github.com/ding-lab/pollock in a conda environment (pytorch, captum, scanpy, umap-learn) and pip install the cloned repo. Download pretrained modules from Zenodo record 5895221 (pretrained_models.tar.gz) or train a new module with 'pollock train' from a Seurat .RDS or scanpy .h5ad object using --cell-type-key; the training/validation split is set by --n-per-cell-type (default 500, with oversampling of rarer cell types) or --val-ids, and hyperparameters default to 20 epochs, latent-dim 64, encoder-out-dim 128, middle-dim 512, kl-scaler 1e-3, zinb-scaler 0.5, clf-scaler 1.0. Supply raw unnormalized counts (Seurat @assays$RNA@counts or anndata .X); Pollock's VAE with linear classification head expects raw expression. Run 'pollock predict' to write per-cell predicted labels to output.rds/.h5ad metadata or a tab-separated txt file (with --txt-output); VAE embeddings are transformed via UMAP into 2D space by default (skip with --no-umap for speed). Assess quality with macro-averaged F1 against annotated labels — on the 12-dataset benchmark Pollock was within 0.02 F1 of the top tool in 11/12 datasets.

## Related tools

- **Pollock** (VAE-based deep learning classifier providing train, predict, and explain modes for cell type identification) — github.com/ding-lab/pollock
- **PyTorch** (Deep learning backend for training and inference of the Pollock VAE model)
- **scanpy** (Loads and writes .h5ad AnnData inputs/outputs and used in the conda environment for single-cell data handling)
- **captum** (Interpretability library installed in the Pollock environment supporting feature/explainability scores)
- **Conda** (Environment creation for Pollock installation (conda create -n pollock -c pytorch -c conda-forge ...))
- **Seurat** (R workflow from which .RDS objects with raw counts and cell type metadata are supplied to Pollock)

## Examples

```
pollock predict from_seurat --module-filepath <path_to_module_directory> --seurat-rds-filepath <filepath_to_RDS_object> --output-prefix output
```

## Evaluation signals

- On the 12 benchmarking datasets, reproduced Pollock F1 should be within 0.02 of the top-performing tool in 11 of 12 datasets.
- Per-cell predictions must cover the expected cells: e.g. the tutorial logs 'starting prediction of 2638 cells' with 12679 genes overlapping the model after filtering and 4859 genes missing.
- Pretrained PBMC model predicts into a defined 17-class label set (e.g. 'B cell', 'CD4 T cell', 'Monocyte', 'NK', 'Malignant'); predicted labels must fall within the module's cell type classes.
- Training/validation split via --n-per-cell-type 500 should oversample rarer cell types so the training set is balanced per cell type.
- Output file format matches the mode: metadata of input .rds/.h5ad by default, or tab-separated .txt when --txt-output is set.

## Limitations

- Raw (unnormalized) counts are required as input for train, predict, and explain modes; normalized data will not be handled correctly.
- kl-scaler trades off embedding noise vs generalizability: increasing it makes cell embeddings noisier but typically more generalizable, decreasing it reduces noise but generalizability.
- Explain mode has excessive runtimes for large cell numbers — ~100 cells with a background sample size of ~100 takes approximately 15 minutes; downsampling is recommended.
- UMAP transformation of VAE embeddings adds prediction time; use --no-umap to skip it.
- --min-genes-per-cell (default 10) filtering only applies in 10x mode.

## Evidence

- [methods] Pollock uses deep learning to make cell type predictions. At it's core, pollock is a variational autoencoder (VAE) with a linear classification head.: "Pollock uses deep learning to make cell type predictions. At it's core, pollock is a variational autoencoder (VAE) with a linear classification head."
- [readme] scanpy expression matrix (.X attribute in the anndata object) must be raw expression counts (i.e. not normalized): "scanpy expression matrix (.X attribute in the anndata object) must be raw expression counts (i.e. not normalized)"
- [readme] N_PER_CELL_TYPE cells will be partitioned into training dataset for each cell type. If less than N_PER_CELL_TYPE cells exist for a cell type than cells are oversampled to balance the training dataset. Default is 500.: "N_PER_CELL_TYPE cells will be partitioned into training dataset for each cell type. If less than N_PER_CELL_TYPE cells exist"
- [abstract] Pretrained models and datasets are available for download at https://zenodo.org/record/5895221.: "Pretrained models and datasets are available for download at https://zenodo.org/record/5895221."
- [methods] The models and training datasets can be found on Zenodo at https://zenodo.org/record/5895221 in pretrained_models.tar.gz and benchmarking_datasets.tar.gz: "The models and training datasets can be found on Zenodo at https://zenodo.org/record/5895221 in pretrained_models.tar.gz and benchmarking_datasets.tar.gz"
