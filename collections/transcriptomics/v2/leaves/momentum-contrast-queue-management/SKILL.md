---
name: momentum-contrast-queue-management
description: Use when learning latent representations from a normalized highly-variable-gene
  scRNA-seq expression matrix (e.g. .npz of HVG profiles) via self-supervised momentum
  contrast instead of deep generative reconstruction.
when_to_use_negative:
- 'The dataset is small: Miscell demands a large sample size to obtain good performance,
  analogous to other deep-neural-network methods'
- 'Batch effects are present and uncorrected: Miscell does not internally include
  any batch-correction methods, so batch correction must be done beforehand'
- A decoder/reconstruction objective is required (e.g. denoising or imputation tasks),
  where deep generative approaches like scVI are the appropriate choice
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Miscell
  role: Provides main.py implementing the MoCo-style momentum-contrast training loop,
    augmentation (20% gene zeroing/shuffling), and checkpoint saving for the DenseNet
    encoder
  repo: github.com/lixiangchun/Miscell
- name: MoCo (facebookresearch)
  role: Upstream reference implementation whose source code was copied into Miscell
    for momentum encoder and queue management
  repo: https://github.com/facebookresearch/moco
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2021.103200
    title: 'Miscell: An efficient self-supervised learning approach for dissecting
      single-cell transcriptome'
  source_content_hash: sha256:7704680284d067c31bf4494b07f08fa7cc53279f297aefe559de4fdb3183a5f5
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
  - MoCo (facebookresearch)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# momentum-contrast-queue-management

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A MoCo-style self-supervised training scheme in which a DenseNet latent feature encoder for single-cell gene expression is trained against a momentum-updated key encoder and a contrastive queue, minimizing an InfoNCE loss between original and augmented (gene-zeroed/shuffled) expression profiles.

## When to use

Use when learning latent representations from a normalized highly-variable-gene scRNA-seq expression matrix (e.g. .npz of HVG profiles) via self-supervised momentum contrast instead of deep generative reconstruction.

## When NOT to use

- The dataset is small: Miscell demands a large sample size to obtain good performance, analogous to other deep-neural-network methods
- Batch effects are present and uncorrected: Miscell does not internally include any batch-correction methods, so batch correction must be done beforehand
- A decoder/reconstruction objective is required (e.g. denoising or imputation tasks), where deep generative approaches like scVI are the appropriate choice

## Inputs

- Normalized highly-variable-gene expression matrix as .npz (e.g. data/x_new_scanpy_HVG.npz with key 'x'), with in_features equal to the number of HVGs (e.g. 1686)
- Architecture choice (densenet11, densenet21, densenet29, densenet63) and MoCo hyperparameters (moco-k 1024, moco-t 0.2, moco-m 0.999, moco-dim 64, --mlp)

## Outputs

- Trained encoder checkpoint file (e.g. checkpoint_0199.pth.tar, or miscell_encoder.pth per the source-card protocol) saved every --save_frequency epochs
- Latent feature embeddings of cells (64-dimensional by default) loadable via utils.load_pretrained_model for downstream FIt-SNE/DBSCAN or scanpy Leiden clustering

## How to apply

Initialize a DenseNet latent feature encoder (e.g. densenet11, densenet21, densenet29, densenet63) with an MLP projection head plus a momentum-updated key encoder sharing the same architecture. Maintain a contrastive queue of size 1024 with momentum coefficient 0.999 and InfoNCE temperature tau 0.2, with the latent dimension set via --moco-dim (e.g. 64). For each cell, generate a positive pair by randomly zeroing out and shuffling 20% of its genes to serve as the augmented view against the original profile. Train end-to-end with stochastic gradient descent (README example: batch size 256, learning rate 0.24, workers 1) across at least two GPUs using --multiprocessing-distributed, while the momentum encoder/MLP parameters are updated gradually in a moving-averaged manner; train for many epochs (source card protocol: 500 epochs) and save periodic checkpoints. The rationale is that contrastive learning between original and augmented profiles avoids the difficult and costly reconstruction loss of deep generative models such as scVI and MARS.

## Related tools

- **Miscell** (Provides main.py implementing the MoCo-style momentum-contrast training loop, augmentation (20% gene zeroing/shuffling), and checkpoint saving for the DenseNet encoder) — github.com/lixiangchun/Miscell
- **MoCo (facebookresearch)** (Upstream reference implementation whose source code was copied into Miscell for momentum encoder and queue management) — https://github.com/facebookresearch/moco

## Examples

```
python main.py -a densenet11 -j 1 -b 256 --lr 0.24 --dist-url "tcp://localhost:10001" --world-size 1 --rank 0 --multiprocessing-distributed --outdir checkpoint --in_features 1686 --save_frequency 5 --mlp --moco-t 0.2 --moco-k 1024 --moco-m 0.999 --moco-dim 64 data/x_new_scanpy_HVG.npz
```

## Evaluation signals

- The checkpoint file (e.g. miscell_encoder.pth / checkpoint_0199.pth.tar) exists and can be reloaded via utils.load_pretrained_model to embed cells into the latent feature space
- MoCo hyperparameters in the saved checkpoint match the specification: queue size 1024 (moco-k), momentum 0.999 (moco-m), temperature 0.2 (moco-t), mlp head enabled, moco-dim 64
- Training ran distributed on at least two GPUs (--multiprocessing-distributed) as the model was trained on GPUs and at least two GPUs are required
- InfoNCE loss between original and augmented representations decreases over epochs under SGD (lr 0.24, batch 256)
- Extracted features support sensible downstream structure, e.g. DBSCAN clusters (eps=1, min_samples=5) on multi-perplexity FIt-SNE embedding (perplexity_list=[30, 36, 42, 48])

## Limitations

- Miscell demands a large sample size to obtain good performance
- Users are required to correct for batch effects if necessary before running Miscell, since no batch-correction is included internally
- Feature normalization before training is recommended ('We recommend the user to normalize the feature before training the model')
- The clinically significant latent features lack explainability, and the biological mechanisms underlying them remain unknown

## Evidence
