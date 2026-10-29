---
name: variational-autoencoder-architecture-implementation
description: Use when high-dimensional, sparse, redundant scRNA-seq or proteomics
  matrices (AnnData-compatible TSV/CSV) must be compressed into a dense low-dimensional
  latent space before correlation-based network inference.
when_to_use_negative:
- Input is already a low-dimensional embedding or precomputed protein association
  network — the VAE stage exists to compress raw high-dimensional condition/sample
  matrices
- The goal is direct co-expression analysis on the original high-dimensional matrix
  rather than correlation in the compressed latent space
- Downstream analysis requires reconstructing the exact input matrix rather than learning
  a dense latent representation for pair scoring
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0203
tools:
- name: favapy
  role: Python package providing the VAE class, load_data routine, and CLI entry point
    that execute this dimensionality reduction stage
  repo: https://github.com/mikelkou/fava
- name: FAVA
  role: Pipeline within which the VAE compresses high-dimensional scRNA-seq/proteomics
    data into the low-dimensional latent space used for network inference
  repo: https://github.com/mikelkou/fava
- name: AnnData
  role: Input data structure used by favapy load_data to hold the omics matrix
  repo: https://github.com/scverse/ecosystem-packages/tree/main/packages/favapy
- name: Python
  role: Implementation language of the fava.py module containing the VAE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:59588263a28dff23f32b42f6c5fb25b50ede3bb53bc28f9fa0ebf6e980da4516
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - AnnData
  - FAVA
  - Python
  - favapy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# variational-autoencoder-architecture-implementation

## Summary

Implements FAVA's VAE dimensionality reduction stage: a single-hidden-layer encoder-decoder (ReLU hidden layers, linear mu/sigma outputs, sigmoid decoder) that compresses high-dimensional omics data into a regularized Gaussian latent space, trained with a 0.9 reconstruction / 0.1 KL loss against a N(1, 0.1) prior.

## When to use

Use when high-dimensional, sparse, redundant scRNA-seq or proteomics matrices (AnnData-compatible TSV/CSV) must be compressed into a dense low-dimensional latent space before correlation-based network inference.

## When NOT to use

- Input is already a low-dimensional embedding or precomputed protein association network — the VAE stage exists to compress raw high-dimensional condition/sample matrices
- The goal is direct co-expression analysis on the original high-dimensional matrix rather than correlation in the compressed latent space
- Downstream analysis requires reconstructing the exact input matrix rather than learning a dense latent representation for pair scoring

## Inputs

- High-dimensional omics data file (TSV or CSV, selected via the -t option) with one row/column per protein, loadable through favapy load_data with AnnData-compatible handling

## Outputs

- vae_model.pt — trained VAE checkpoint
- latent_embeddings.tsv — one latent vector per input protein in the regularized Gaussian latent space

## How to apply

Load the TSV/CSV input via favapy's load_data with AnnData-compatible handling, then build a VAE with one intermediate/hidden layer using ReLU activations, linear mu and sigma output layers, and a sigmoid decoder output for 0-1 scaled inputs; latent dimensions default to a size derived from the hidden layer. Train with the Adam optimizer at learning rate 1e-3 for 50 epochs at batch size 32, minimizing a weighted loss of 0.9 reconstruction error plus 0.1 KL divergence against a normal prior with mean 1 and std 0.1. The 0.9/0.1 weighting reflects FAVA's choice of reconstruction-dominant training with mild Gaussian regularization, which addresses the sparseness and redundancy of omics data by producing a dense latent space. Encode all proteins through the trained model to obtain the latent embeddings, then save the checkpoint as vae_model.pt and embeddings as latent_embeddings.tsv.

## Related tools

- **favapy** (Python package providing the VAE class, load_data routine, and CLI entry point that execute this dimensionality reduction stage) — https://github.com/mikelkou/fava
- **FAVA** (Pipeline within which the VAE compresses high-dimensional scRNA-seq/proteomics data into the low-dimensional latent space used for network inference) — https://github.com/mikelkou/fava
- **AnnData** (Input data structure used by favapy load_data to hold the omics matrix) — https://github.com/scverse/ecosystem-packages/tree/main/packages/favapy
- **Python** (Implementation language of the fava.py module containing the VAE)

## Examples

```
favapy <path-to-data-file> <path-to-save-output>
```

## Evaluation signals

- Architecture check: exactly one hidden layer, ReLU activations on the hidden layer, linear activation on mu/sigma layers, and sigmoid decoder output (appropriate for 0-1 scaled inputs)
- Loss check: training minimizes 0.9 * reconstruction error + 0.1 * KL divergence against N(mean=1, std=0.1)
- Optimizer check: Adam with learning rate 1e-3, 50 epochs, batch size 32
- Output schema check: latent_embeddings.tsv contains one latent vector per input protein
- Downstream check: correlations computed in the resulting latent space should yield improved protein association networks versus the original high-dimensional representation

## Limitations

- A sigmoid decoder output assumes inputs are scaled to 0-1; unscaled or differently normalized omics matrices will violate this assumption
- The KL term anchors the latent space to a narrow N(1, 0.1) prior; deviations of the data distribution from this regime may degrade reconstruction
- The method is designed for high-dimensional, sparse, redundant condition/sample data — performance on small, dense datasets is not established in the provided context

## Evidence

- [other] FAVA combines its reconstruction and regularization losses as a weighted sum, with weights of 0.9 and 0.1 respectively, where the regularization loss is implemented as KL divergence against a normal prior (mean 1, std 0.1): [evidence span withheld — anchor status: not_found]
- [other] the model uses a single hidden layer with ReLU activations, linear activation on the mu/sigma layers, a sigmoid decoder output for 0-1 scaled inputs, and is trained with the Adam optimizer at a learning rate of 10^-3: [evidence span withheld — anchor status: not_found]
- [other] Train the model with the Adam optimizer (learning rate 1e-3) for 50 epochs at batch size 32, minimizing a weighted loss of 0.9 reconstruction error plus 0.1 KL divergence against a normal prior with mean 1 and std 0.1: [evidence span withheld — anchor status: not_found]
- [abstract] we have developed FAVA (Functional Associations using Variational Autoencoders), which compresses high-dimensional data into a low-dimensional space: "we have developed FAVA (Functional Associations using Variational Autoencoders), which compresses high-dimensional data into a low-dimensional space"
- [intro] Calculating correlations in this latent space results in much improved networks compared to the original representation: "Calculating correlations in this latent space results in much improved networks compared to the original representation"
- [other] Save the trained model checkpoint as vae_model.pt and the latent embeddings as latent_embeddings.tsv: [evidence span withheld — anchor status: not_found]
