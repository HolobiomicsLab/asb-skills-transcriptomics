---
name: densenet-encoder-architecture-configuration
description: Use when training a self-supervised feature encoder on a preprocessed
  single-cell expression matrix of highly variable genes and a DenseNet-based MoCo
  architecture with contrastive queue settings must be configured.
when_to_use_negative:
- 'The dataset is small: Miscell demands a large sample size to obtain good performance,
  so deep-encoder training is inappropriate for small scRNA-seq datasets'
- 'Batch effects are present and uncorrected: Miscell does not include batch correction,
  so the input expression matrix must be batch-corrected beforehand'
- 'The input is raw unnormalized counts: the README recommends normalizing features
  before training the model'
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: Miscell
  role: Provides the DenseNet encoder architecture, MoCo training script (main.py),
    and checkpoint/feature-extraction utilities
  repo: github.com/lixiangchun/Miscell
- name: moco (facebookresearch)
  role: Source of the momentum-contrast training code copied into Miscell
  repo: facebookresearch/moco
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2021.103200
    title: 'Miscell: An efficient self-supervised learning approach for dissecting
      single-cell transcriptome'
  source_content_hash: sha256:2c7d9829fb50e2e58dde1ebcdc7069b94cb6b88444fcb2584a43c3c2a48a3cb9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/lixiangchun/Miscell
  tools:
  - Miscell
  - moco (facebookresearch)
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# densenet-encoder-architecture-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure a DenseNet latent feature encoder (densenet11/21/29/63 variants) with an MLP projection head for MoCo-style self-supervised representation learning on single-cell gene expression matrices, as implemented in Miscell.

## When to use

Use when training a self-supervised feature encoder on a preprocessed single-cell expression matrix of highly variable genes and a DenseNet-based MoCo architecture with contrastive queue settings must be configured.

## When NOT to use

- The dataset is small: Miscell demands a large sample size to obtain good performance, so deep-encoder training is inappropriate for small scRNA-seq datasets
- Batch effects are present and uncorrected: Miscell does not include batch correction, so the input expression matrix must be batch-corrected beforehand
- The input is raw unnormalized counts: the README recommends normalizing features before training the model

## Inputs

- Preprocessed/normalized highly-variable-gene expression matrix in .npz format (e.g., data/x_new_scanpy_HVG.npz with key 'x')
- Number of input features (in_features) equal to the number of HVGs
- Choice of DenseNet architecture variant: densenet11, densenet21, densenet29, or densenet63

## Outputs

- Trained encoder checkpoint file (e.g., checkpoint_0199.pth.tar, miscell_encoder.pth)
- 64-dimensional latent feature embeddings per cell (via utils.extract_features)
- Features usable for FIt-SNE visualization and DBSCAN/Leiden clustering

## How to apply

Initialize one of the DenseNet architecture variants (densenet11, densenet21, densenet29, or densenet63) with in_features set to the number of highly variable genes in the input matrix (e.g., in_features=1686 for the PBMC HVG matrix). Attach an MLP projection head (--mlp) and configure the momentum-contrast hyperparameters: contrastive queue size K=1024 (--moco-k), momentum coefficient 0.999 (--moco-m), and InfoNCE temperature tau=0.2 (--moco-t), with the latent dimension set via --moco-dim 64. Train by stochastic gradient descent with learning rate 0.24 and batch size 256; the encoder and MLP are updated end-to-end while the momentum (key) encoder and its MLP are updated as a moving average. Features are augmented by randomly zeroing and shuffling 20% of genes per cell to form positive pairs. Save checkpoints periodically (--save_frequency 5) and reload the checkpoint via utils.load_pretrained_model to embed cells into the 64-dimensional latent space for downstream clustering.

## Related tools

- **Miscell** (Provides the DenseNet encoder architecture, MoCo training script (main.py), and checkpoint/feature-extraction utilities) — github.com/lixiangchun/Miscell
- **moco (facebookresearch)** (Source of the momentum-contrast training code copied into Miscell) — facebookresearch/moco

## Examples

```
python main.py -a densenet21 -j 1 -b 256 --lr 0.24 --dist-url "tcp://localhost:10001" --world-size 1 --rank 0 --multiprocessing-distributed --outdir checkpoint --in_features 1686 --save_frequency 5 --mlp --moco-t 0.2 --moco-k 1024 --moco-m 0.999 --moco-dim 64 data/x_new_scanpy_HVG.npz
```

## Evaluation signals

- Checkpoint file exists and can be reloaded with utils.load_pretrained_model to embed cells into the latent feature space
- Loaded in_features matches the expression matrix width (assert in_features == X.shape[1])
- Latent features are 64-dimensional per cell, consistent with --moco-dim 64
- Downstream t-SNE + DBSCAN or scanpy Leiden clustering on the extracted features yields coherent cell clusters (e.g., PBMC cell types)
- Training converges on at least two GPUs without queue/temperature misconfiguration (moco-k 1024, moco-t 0.2, moco-m 0.999)

## Limitations

- At least two GPUs are required for training per the README
- Miscell demands a large sample size to obtain good performance
- Miscell does not internally include any batch-correction methods; users must correct batch effects before running it

## Evidence

- [methods] MoCo training details from the source card workflow: [evidence span withheld — anchor status: not_found]
