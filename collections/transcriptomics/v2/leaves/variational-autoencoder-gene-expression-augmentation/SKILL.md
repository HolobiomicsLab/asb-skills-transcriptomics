---
name: variational-autoencoder-gene-expression-augmentation
description: Use when training a spatial location recovery model on Visium-type ST
  data and the training sample is limited or the scRNA-seq query data is noisy, so
  synthetic ST replicates are needed to augment the training set.
when_to_use_negative:
- When the goal is direct 2D location or spatial domain prediction on the original
  ST data without the optional augmentation step — the VAE replicates are an optional
  addition, not a requirement.
- When the ST data has no spot-level spatial coordinates, since each gene's 1D expression
  vector cannot be reshaped into the 2D matrix the convolutional VAE requires.
- When the training ST data comes from a single-cell-resolution platform with a small
  gene panel (e.g., small MERFISH/Xenium panels), where the authors note CeLEry is
  suboptimal and augmentation is unlikely to compensate for limited gene coverage.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: CeLEry
  role: Implements the optional VAE-based data augmentation procedure that generates
    ST data replicates combined with the original data for model training
  repo: github.com/QihuangZhang/CeLEry
- name: PyTorch
  role: Deep learning framework used to build and train the convolutional variational
    autoencoder (encoder/decoder layers, embedding dimension 512, beta = 1e-5 KL weighting)
- name: Scanpy
  role: Used for two-step normalization of ST gene expression (spot-level log transformation
    of UMI counts followed by gene-level standardization)
- name: scikit-learn
  role: Provides K-means clustering of genes into groups with similar expression patterns,
    encoded as a one-hot matrix
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-39895-3
    title: Leveraging spatial transcriptomics data to recover cell locations in single-cell
      RNA-seq with CeLEry
  source_content_hash: sha256:4f7e1df8672e764ee79a003cd218befc0df9435f24bae70e941bc6777962e9b8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CeLEry
  - PyTorch
  - Scanpy
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# variational-autoencoder-gene-expression-augmentation

## Summary

Optional CeLEry data augmentation step that trains a convolutional variational autoencoder on the training spatial transcriptomics (ST) data to generate replicates whose 2D gene expression patterns resemble the original data while varying among replicates, improving robustness against scRNA-seq noise.

## When to use

Use when training a spatial location recovery model on Visium-type ST data and the training sample is limited or the scRNA-seq query data is noisy, so synthetic ST replicates are needed to augment the training set.

## When NOT to use

- When the goal is direct 2D location or spatial domain prediction on the original ST data without the optional augmentation step — the VAE replicates are an optional addition, not a requirement.
- When the ST data has no spot-level spatial coordinates, since each gene's 1D expression vector cannot be reshaped into the 2D matrix the convolutional VAE requires.
- When the training ST data comes from a single-cell-resolution platform with a small gene panel (e.g., small MERFISH/Xenium panels), where the authors note CeLEry is suboptimal and augmentation is unlikely to compensate for limited gene coverage.

## Inputs

- Preprocessed Visium ST data: spot-level log-transformed UMI counts, gene-level standardized across spots
- Per-gene 2D spatial expression matrices arranged by spot coordinates (unmeasured spots coded as 0)
- One-hot matrix of K-means gene cluster assignments
- Trained convolutional VAE encoder and decoder (channels 16/8/4, embedding dimension 512, beta = 1e-5)

## Outputs

- Augmented ST replicates (e.g., three) with 2D gene expression patterns similar to the original ST data
- Per-gene embedding mean and variance estimated from the trained encoder
- Comparison figure of generated versus original 2D expression maps for exemplar genes (RPS20, RPS29, CALB1)

## How to apply

Preprocess the mouse posterior brain Visium ST data with two-step normalization: spot-level log transformation of UMI counts, then gene-level standardization (subtract gene mean, divide by gene SD across spots). Reshape each gene's 1D expression vector into a 2D matrix arranged by spot spatial coordinates, coding unmeasured spots as 0. Run K-means to cluster genes with similar expression patterns and encode the assignments as a one-hot matrix. Train a convolutional VAE with three 2D convolutional encoder layers (16, 8, 4 channels), a mirrored decoder, an embedding dimension of 512, and a loss of reconstruction error plus beta-weighted KL divergence with beta = 1e-5. Estimate per-gene embedding mean and variance from the trained encoder, sample embeddings from N(mu, sigma^2), and decode them to reconstruct spatial gene expression matrices that serve as augmented ST replicates combined with the original data for model training; choose the number of replicates (the paper generated three) via cross-validation.

## Related tools

- **CeLEry** (Implements the optional VAE-based data augmentation procedure that generates ST data replicates combined with the original data for model training) — github.com/QihuangZhang/CeLEry
- **PyTorch** (Deep learning framework used to build and train the convolutional variational autoencoder (encoder/decoder layers, embedding dimension 512, beta = 1e-5 KL weighting))
- **Scanpy** (Used for two-step normalization of ST gene expression (spot-level log transformation of UMI counts followed by gene-level standardization))
- **scikit-learn** (Provides K-means clustering of genes into groups with similar expression patterns, encoded as a one-hot matrix)

## Examples

```
pip install CeLEryPy
```

## Evaluation signals

- Each generated replicate's 2D gene expression pattern should be visually similar to the original ST data while showing variation among replicates.
- Visual inspection of exemplar genes (RPS20, RPS29, CALB1): the generated versus original 2D expression maps should match in spatial pattern and be saved as a comparison panel figure.
- Decoder output matrices should have the same 2D spatial dimensions as the original reshaped spot-coordinate gene expression matrices.
- VAE training loss should reflect reconstruction error plus beta-weighted KL divergence with beta = 1e-5; divergence should remain small relative to reconstruction.
- Number of replicates should be validated as a hyperparameter via cross-validation rather than fixed arbitrarily.

## Limitations

- The number of replicates to be generated is a hyperparameter prespecified by users; the optimal value requires cross-validation rather than a fixed recommended setting.
- Augmentation is optional; the article states it improves robustness and helps overcome noise in scRNA-seq, but its benefit depends on the training ST data quality.
- CeLEry overall is suboptimal on single-cell-resolution platforms with small gene panels; the authors await ST data with single-cell resolution and transcriptome-wide coverage.

## Evidence

- [abstract] abstract: "CeLEry has an optional data augmentation procedure via a variational autoencoder, which improves the method's robustness and allows it to overcome noise in scRNA-seq data."
- [methods] methods: [evidence span withheld — anchor status: not_found]
- [discussion] discussion: "we recommend that the user explore the best number of replicates by performing cross-validation"
- [readme] readme: "CeLEry optionally generates replicates of the spatial transcriptomic data via variational autoencoder then includes them as the training data together with original spatial transcriptomic data."
