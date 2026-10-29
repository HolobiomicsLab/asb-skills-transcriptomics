---
name: sgd-training-hyperparameter-configuration
description: Use when a normalized single-cell gene expression feature matrix (e.g.
  scanpy HVG .npz) must be embedded into a latent feature space via self-supervised
  MoCo contrastive training of a DenseNet encoder with SGD.
when_to_use_negative:
- The dataset is small — Miscell demands a large sample size to obtain good performance
  with deep neural network training
- The input is raw, unnormalized counts or lacks HVG preprocessing — features must
  be normalized and restricted to highly variable genes before training
- Only a single GPU (or CPU) is available — the model was trained on GPUs and at least
  two GPUs are required
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Miscell
  role: Provides main.py and utils for MoCo-style DenseNet encoder training, checkpointing,
    and feature extraction from single-cell transcriptome data
  repo: https://github.com/lixiangchun/Miscell
- name: MoCo (facebookresearch/moco)
  role: Source of the momentum-contrast training code copied into Miscell for SGD
    training of the encoder with a momentum-updated key encoder and contrastive queue
  repo: facebookresearch/moco
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2021.103200
    title: 'Miscell: An efficient self-supervised learning approach for dissecting
      single-cell transcriptome'
  source_content_hash: sha256:45f99a753d5ce1627f981741d4ee7ff20003a7a08ce11f713f520f9fda9dff10
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/lixiangchun/Miscell
  repo_url: https://github.com/lixiangchun/Miscell
  tools:
  - Miscell
  - MoCo (facebookresearch/moco)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# sgd-training-hyperparameter-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configures and runs stochastic gradient descent training of the Miscell momentum-contrast (MoCo-style) DenseNet latent feature encoder on scRNA-seq expression matrices, setting the InfoNCE contrastive hyperparameters (queue size 1024, momentum 0.999, temperature 0.2) and SGD learning rate. It turns a normalized highly-variable-gene matrix into a reusable encoder checkpoint for downstream latent-space clustering.

## When to use

Use when a normalized single-cell gene expression feature matrix (e.g. scanpy HVG .npz) must be embedded into a latent feature space via self-supervised MoCo contrastive training of a DenseNet encoder with SGD.

## When NOT to use

- The dataset is small — Miscell demands a large sample size to obtain good performance with deep neural network training
- The input is raw, unnormalized counts or lacks HVG preprocessing — features must be normalized and restricted to highly variable genes before training
- Only a single GPU (or CPU) is available — the model was trained on GPUs and at least two GPUs are required

## Inputs

- Normalized highly-variable-gene expression matrix in .npz format (e.g. data/x_new_scanpy_HVG.npz with key 'x', in_features = number of HVGs such as 1686)
- Preprocessed scRNA-seq counts (e.g. 43,095 PBMCs profiled with the 10x protocol) after batch correction/normalization
- Choice of DenseNet architecture: densenet11, densenet21, densenet29 or densenet63

## Outputs

- Trained encoder checkpoint files (e.g. checkpoint_0199.pth.tar in the checkpoint output directory; miscell_encoder.pth in the reference workflow)
- 64-dimensional latent feature embeddings of all cells extractable via utils.extract_features with return_feature=True

## How to apply

Load the preprocessed, normalized highly-variable-gene expression matrix (e.g. data/x_new_scanpy_HVG.npz with in_features equal to the number of HVGs, e.g. 1686) and initialize a DenseNet encoder (densenet11, densenet21, densenet29 or densenet63 — a 21-layer DenseNet in the reference workflow) plus an MLP projection head. Launch distributed SGD training via main.py with batch size 256, learning rate 0.24, and the MoCo contrastive settings --mlp --moco-t 0.2 --moco-k 1024 --moco-m 0.999 --moco-dim 64; training minimizes the InfoNCE loss between original cells and augmented views in which 20% of genes are randomly zeroed and shuffled, run for 500 epochs. During training, the encoder and MLP parameters are updated end-to-end by SGD while the momentum (key) encoder and MLP parameters are updated gradually in a moving-averaged manner (momentum 0.999) against a contrastive queue of size 1024. Save checkpoints (--outdir checkpoint, --save_frequency 5) so the trained encoder can be reloaded with utils.load_pretrained_model to embed cells into the 64-dimensional latent feature space. Training is GPU-based and requires at least two GPUs.

## Related tools

- **Miscell** (Provides main.py and utils for MoCo-style DenseNet encoder training, checkpointing, and feature extraction from single-cell transcriptome data) — https://github.com/lixiangchun/Miscell
- **MoCo (facebookresearch/moco)** (Source of the momentum-contrast training code copied into Miscell for SGD training of the encoder with a momentum-updated key encoder and contrastive queue) — facebookresearch/moco

## Examples

```
python main.py -a densenet11 -j 1 -b 256 --lr 0.24 --dist-url "tcp://localhost:10001" --world-size 1 --rank 0 --multiprocessing-distributed --outdir checkpoint --in_features 1686 --mlp --moco-t 0.2 --moco-k 1024 --moco-m 0.999 --moco-dim 64 data/x_new_scanpy_HVG.npz
```

## Evaluation signals

- Checkpoint file (e.g. miscell_encoder.pth or checkpoint_0199.pth.tar) exists and can be reloaded with utils.load_pretrained_model to embed cells into the latent feature space
- in_features argument equals X.shape[1] of the loaded .npz expression matrix (e.g. assert in_features == 1686)
- Training command uses the reference MoCo settings: --moco-t 0.2, --moco-k 1024, --moco-m 0.999, --moco-dim 64, batch size 256, lr 0.24
- Extracted feature matrix has one 64-dimensional row per cell and supports downstream clustering (FIt-SNE + DBSCAN or scanpy Leiden) with non-trivial cluster structure
- Loss decreases over the 500 SGD epochs and the momentum encoder tracks the base encoder (moving-average update with coefficient 0.999)

## Limitations

- Miscell demands a large sample size to obtain good performance, analogous to other deep neural network methods
- At least two GPUs are required for training
- Users must normalize features and correct batch effects themselves before running Miscell; batch correction is not included internally

## Evidence

- [methods] DenseNet encoder and MoCo initialization: [evidence span withheld — anchor status: not_found]
- [methods] Momentum-averaged key encoder update: [evidence span withheld — anchor status: not_found]
