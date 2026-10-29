---
name: single-cell-gene-expression-modeling
description: Use when the goal is to reproduce or verify the fixed architecture of
  Pollock's VAE-based cell-type classifier (encoder/decoder, 64-dim latent, classification
  head, combined loss) against a pretrained Zenodo checkpoint before reuse or retraining.
when_to_use_negative:
- You only need to run cell type prediction with an existing pretrained module on
  raw-count input (use pollock predict mode instead of rebuilding the network).
- Your aim is annotating a new dataset with an existing module, not reproducing or
  validating the network architecture itself.
- Input expression matrices are normalized rather than raw counts — pollock expects
  raw expression counts in .X or the Seurat counts assay.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: Pollock
  role: Defines the VAE + linear classification head design and the default dimensions
    (--middle-dim 512, --encoder-out-dim 128, --latent-dim 64) and loss scalers to
    reconstruct
  repo: https://github.com/ding-lab/pollock
- name: PyTorch
  role: Framework for implementing the encoder, mirrored ZINB decoder, and classification
    head, and for strictly loading the checkpoint state_dict
- name: scanpy
  role: Loads single-cell count matrices (e.g. read_10x_mtx of 10x pbmc3k data) used
    to establish the input gene dimension
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbac028
    title: 'Pollock: fishing for cell states'
  source_content_hash: sha256:71f1c72f6876615210a9b7815dcbc8fbd6458d2d6f70adc9e9244245eefa9f7f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ding-lab/pollock
  tools:
  - Pollock
  - PyTorch
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# single-cell-gene-expression-modeling

## Summary

Reconstruct Pollock's single-cell gene-expression model: a PyTorch variational autoencoder (two fully connected ReLU layers of 512 and 128, a 64-dim latent cell embedding, a ZINB decoder) with a linear classification head, then verify the reconstruction by strictly loading a pretrained checkpoint from Zenodo record 5895221.

## When to use

Use when the goal is to reproduce or verify the fixed architecture of Pollock's VAE-based cell-type classifier (encoder/decoder, 64-dim latent, classification head, combined loss) against a pretrained Zenodo checkpoint before reuse or retraining.

## When NOT to use

- You only need to run cell type prediction with an existing pretrained module on raw-count input (use pollock predict mode instead of rebuilding the network).
- Your aim is annotating a new dataset with an existing module, not reproducing or validating the network architecture itself.
- Input expression matrices are normalized rather than raw counts — pollock expects raw expression counts in .X or the Seurat counts assay.

## Inputs

- pretrained_models.tar.gz from Zenodo record 5895221 (module checkpoint, e.g. scRNAseq_generalized)
- Gene count / gene set used to size the input layer, read from the checkpoint (e.g. 32738 genes in the PBMC tutorial AnnData)
- Pollock source code from github.com/ding-lab/pollock for default hyperparameter values

## Outputs

- PyTorch VAE encoder/decoder with 64-dim latent plus linear classification head matching the checkpoint
- architecture_verification_report.txt with per-layer dimension comparison
- Strict state_dict load confirmation (no shape or key mismatches)

## How to apply

Download pretrained_models.tar.gz from Zenodo record 5895221 and extract one pretrained module checkpoint (e.g. scRNAseq_generalized). In PyTorch, implement the encoder as an input layer sized to the gene count (read from the checkpoint, since datasets differ, e.g. 32738 genes in the PBMC tutorial data), followed by fully connected layers of size 512 (--middle-dim default) and 128 (--encoder-out-dim default), each with ReLU, feeding a 64-dimensional latent cell embedding (--latent-dim default). Implement the mirrored decoder with a ZINB reconstruction loss weighted by --zinb-scaler, the KL regularization term weighted by --kl-scaler (default 1e-3), and the classification cross-entropy loss weighted by --clf-scaler, all summed into a single overall loss. Implement the classification head as linear + ReLU from the 64-dim latent to a Softmax over cell type classes (pretrained modules span 17 classes such as 'B cell', 'CD4 T cell', 'Monocyte', 'NK', 'Malignant'). Load the checkpoint state_dict and compare every layer's name and tensor shape against the reconstruction, then write architecture_verification_report.txt; acceptance is a strict state_dict load with no shape or key mismatches.

## Related tools

- **Pollock** (Defines the VAE + linear classification head design and the default dimensions (--middle-dim 512, --encoder-out-dim 128, --latent-dim 64) and loss scalers to reconstruct) — https://github.com/ding-lab/pollock
- **PyTorch** (Framework for implementing the encoder, mirrored ZINB decoder, and classification head, and for strictly loading the checkpoint state_dict)
- **scanpy** (Loads single-cell count matrices (e.g. read_10x_mtx of 10x pbmc3k data) used to establish the input gene dimension)

## Examples

```
pollock train from_scanpy --module-filepath out_module --scanpy-h5ad-filepath adata.h5ad --cell-type-key cell_type --middle-dim 512 --enc-out-dim 128 --latent-dim 64 --kl-scaler 1e-3
```

## Evaluation signals

- model.load_state_dict(checkpoint, strict=True) completes with no missing keys, unexpected keys, or shape mismatches
- architecture_verification_report.txt shows every reconstructed layer name and tensor shape identical to the checkpoint state_dict
- Reconstructed dimensions match CLI defaults: middle 512, encoder-out 128, latent 64
- Overall loss equals the weighted sum: ZINB reconstruction (zinb-scaler) + KL (kl-scaler, default 1e-3) + cross-entropy classification (clf-scaler)
- Input layer width equals the gene count stored in the checkpoint rather than a hardcoded value

## Limitations

- Explain mode has excessive runtimes for very large numbers of cells (~100 cells with ~100 background cells takes about 15 minutes), so downsample cells and background samples.
- The classification head and layer activations beyond what the README documents must be inferred from the checkpoint itself; strict-load verification is the arbiter of correctness.
- Pretrained modules are disease- and modality-specific (e.g. scRNAseq_brca, snRNAseq_gbm, snATACseq gene-activity models); a checkpoint's gene set differs from your dataset, as seen where 12679 of 17538 genes overlapped and 4859 were missing at prediction time.
- Prediction input must be raw (unnormalized) counts in the Seurat counts assay, the .h5ad .X attribute, or 10x cellranger output.

## Evidence

- [methods] Pollock uses deep learning to make cell type predictions. At it's core, pollock is a variational autoencoder (VAE) with a linear classification head.: "At it's core, pollock is a variational autoencoder (VAE) with a linear classification head."
- [readme] The models and training datasets can be found on Zenodo at https://zenodo.org/record/5895221 in pretrained_models.tar.gz and benchmarking_datasets.tar.gz: "The models and training datasets can be found on Zenodo at https://zenodo.org/record/5895221 in pretrained_models.tar.gz and benchmarking_datasets.tar.gz"
- [readme] --latent-dim LATENT_DIM * Size of hidden layer in the VAE. Default is 64.: "Size of hidden layer in the VAE. Default is 64."
- [readme] --middle-dim MIDDLE_DIM * Size of intermediate linear layers. Default is 512.: "Size of intermediate linear layers. Default is 512."
- [readme] --kl-scaler KL_SCALER * This parameter controls how regularized the VAE is. 1e-3 is the default.: "This parameter controls how regularized the VAE is. 1e-3 is the default."
- [results] AnnData object with n_obs × n_vars = 2700 × 32738: "AnnData object with n_obs × n_vars = 2700 × 32738"
- [readme] By default VAE embeddings are transformed via UMAP into 2D space and incorporated into predicted object.: "By default VAE embeddings are transformed via UMAP into 2D space and incorporated into predicted object."
