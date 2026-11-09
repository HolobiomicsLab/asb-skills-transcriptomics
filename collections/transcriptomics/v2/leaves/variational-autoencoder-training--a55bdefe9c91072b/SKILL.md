---
name: variational-autoencoder-training
description: Use when you have annotated raw single-cell counts (Seurat .RDS, scanpy
  .h5ad, or 10x matrix) and must train a new Pollock module from scratch or reproduce
  the paper's disease-specific/generalized/pan-immune models.
when_to_use_negative:
- Input expression matrix is already normalized/log-transformed — Pollock requires
  raw counts and will misbehave on normalized data
- Goal is only inference or interpretability with existing models — use predict or
  explain mode with the Zenodo pretrained_models.tar.gz modules instead of retraining
- Data is not annotated single-cell counts with a cell type label column — there is
  nothing to train the classification head against
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Pollock
  role: Command line tool providing the train mode that fits the VAE classification
    module on annotated single-cell data
  repo: github.com/ding-lab/pollock
- name: PyTorch
  role: Deep learning framework underlying the VAE implementation (encoder 512/128
    ReLU layers, 64-dim latent embedding, ZINB decoder, linear classification head)
- name: scanpy
  role: Provides from_scanpy input path; .h5ad anndata objects with raw counts in
    .X are used as training input
- name: Conda
  role: Creates the pollock environment with pytorch, captum, scanpy, umap-learn,
    and r-base needed to install and run Pollock
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbac028
    title: 'Pollock: fishing for cell states'
  source_content_hash: sha256:adc2c317119013431bafa8af01ae3d7b629da999c4b3ca318dc4f6af1c585dde
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Conda
  - Pollock
  - PyTorch
  - Seurat
  - scanpy
  - uniPort
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# variational-autoencoder-training

## Summary

Trains Pollock's cell type classification VAE — a variational autoencoder with a linear classification head and combined ZINB + KL + cross-entropy loss — on annotated single-cell datasets (scRNA-seq, snRNA-seq, or snATAC-seq gene activity) via the Pollock command line tool in train mode.

## When to use

Use when you have annotated raw single-cell counts (Seurat .RDS, scanpy .h5ad, or 10x matrix) and must train a new Pollock module from scratch or reproduce the paper's disease-specific/generalized/pan-immune models.

## When NOT to use

- Input expression matrix is already normalized/log-transformed — Pollock requires raw counts and will misbehave on normalized data
- Goal is only inference or interpretability with existing models — use predict or explain mode with the Zenodo pretrained_models.tar.gz modules instead of retraining
- Data is not annotated single-cell counts with a cell type label column — there is nothing to train the classification head against

## Inputs

- Seurat .RDS object with cell type labels in metadata and raw counts in @assays$RNA@counts
- scanpy .h5ad anndata object with cell type key in .obs and raw counts in .X
- 10x cellranger output (mtx directory or .h5) for predict mode
- Hyperparameter flags (epochs, latent-dim, encoder-out-dim, middle-dim, kl-scaler, zinb-scaler, clf-scaler, n-per-cell-type)

## Outputs

- Trained Pollock module directory at --module-filepath (loadable checkpoint for predict/explain modes)
- Cell type predictions on validation split (usable for per-cell-type and macro-averaged F1 computation)

## How to apply

Run `pollock train from_seurat/from_scanpy` with `--cell-type-key` pointing to the metadata column holding annotated cell type labels. Use the paper's default hyperparameters: epochs=20, batch size 64, latent-dim 64, encoder-out-dim 128, middle-dim 512, lr 1e-4 (kl-scaler 1e-3, zinb-scaler 0.5, clf-scaler 1.0 as in the benchmark). Validation is split via --n-per-cell-type 500: N_PER_CELL_TYPE cells per cell type are partitioned into the training set, and if fewer cells exist for a rare cell type they are oversampled to balance the training data. Inputs must be raw, unnormalized counts — the .h5ad .X attribute or Seurat @assays$RNA@counts — because Pollock models the ZINB count distribution. The trained module is saved to --module-filepath and can then be used in predict or explain mode.

## Related tools

- **Pollock** (Command line tool providing the train mode that fits the VAE classification module on annotated single-cell data) — github.com/ding-lab/pollock
- **PyTorch** (Deep learning framework underlying the VAE implementation (encoder 512/128 ReLU layers, 64-dim latent embedding, ZINB decoder, linear classification head))
- **scanpy** (Provides from_scanpy input path; .h5ad anndata objects with raw counts in .X are used as training input)
- **Conda** (Creates the pollock environment with pytorch, captum, scanpy, umap-learn, and r-base needed to install and run Pollock)

## Examples

```
pollock train from_scanpy --module-filepath ./pollock_module --scanpy-h5ad-filepath input.h5ad --cell-type-key cell_type --epochs 20 --n-per-cell-type 500
```

## Evaluation signals

- The trained checkpoint's state_dict loads under a strict load with no key or shape mismatches when compared layer-by-layer against the known architecture (input -> 512 -> 128 -> 64-dim latent, linear + ReLU classification head to Softmax over cell types)
- On the 12 benchmarking datasets, the trained module reaches macro-averaged F1 within 0.02 of the top-performing tool (Seurat, Scanpy, SingleCellNet, ACTINN, SVM-linear, MLP) in 11 of 12 datasets
- Training/validation split used n-per-cell-type=500 with oversampling of rarer cell types, and raw (unnormalized) counts were supplied as input
- Loss components match the specification: ZINB reconstruction weighted by --zinb-scaler, KL regularization weighted by --kl-scaler (default 1e-3), and classification cross-entropy weighted by --clf-scaler, summed into a single overall loss over 20 epochs

## Limitations

- kl-scaler trades off embedding quality: increasing it makes cell embeddings noisier but typically more generalizable, decreasing it makes them less noisy but less generalizable
- If fewer than n-per-cell-type cells exist for a cell type, cells are oversampled, which affects rare cell types in unbalanced datasets
- Training requires raw counts; supplying normalized .X data or normalized Seurat counts violates the input contract and is a failure mode

## Evidence

- [methods] Pollock uses deep learning to make cell type predictions. At it's core, pollock is a variational autoencoder (VAE) with a linear classification head.: "Pollock uses deep learning to make cell type predictions. At it's core, pollock is a variational autoencoder (VAE) with a linear classification head."
- [readme] Size of hidden layer in the VAE. Default is 64.: "--latent-dim LATENT_DIM
  * Size of hidden layer in the VAE. Default is 64."
- [readme] This parameter controls how regularized the VAE is. 1e-3 is the default.: "--kl-scaler KL_SCALER
  * This parameter controls how regularized the VAE is. 1e-3 is the default."
- [readme] If less than N_PER_CELL_TYPE cells exist for a cell type than cells are oversampled to balance the training dataset. Default is 500.: "If less than N_PER_CELL_TYPE cells exist for a cell type than cells are oversampled to balance the training dataset. Default is 500."
- [readme] The models and training datasets can be found on Zenodo at https://zenodo.org/record/5895221 in pretrained_models.tar.gz and benchmarking_datasets.tar.gz: "The models and training datasets can be found on Zenodo at https://zenodo.org/record/5895221 in pretrained_models.tar.gz and benchmarking_datasets.tar.gz"
