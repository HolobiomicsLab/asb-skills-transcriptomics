---
name: high-dimensional-omics-latent-embedding-generation
description: Use when input is a high-dimensional protein x condition omics matrix
  (TSV/CSV, AnnData-compatible) with sparseness and redundancy, and a dense low-dimensional
  per-protein embedding is needed before network inference.
when_to_use_negative:
- The data are already a low-dimensional or dense embedding — FAVA exists to compress
  high-dimensional, sparse, redundant data
- The goal is direct co-expression correlation on the raw high-dimensional matrix;
  the article states this approach works poorly due to sparseness and redundancy
- Input rows are conditions/cells rather than proteins, since the embedding is one
  latent vector per input protein
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3308
tools:
- name: favapy
  role: Python/CLI package that loads data, builds and trains the VAE, and emits the
    latent embeddings
  repo: https://github.com/mikelkou/fava
- name: AnnData
  role: Input data structure used by favapy's load_data for AnnData-compatible handling
    of the omics matrix
  repo: https://github.com/mikelkou/fava
- name: FAVA
  role: The VAE-based method implemented by favapy that performs the dimensionality
    reduction
  repo: https://github.com/mikelkou/fava
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:91e1e4ecac6c439f52ef1b213f4104a920ecb0ee3d40a6a15274fd69e15cf598
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
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# high-dimensional-omics-latent-embedding-generation

## Summary

Compress high-dimensional scRNA-seq/proteomics protein-condition matrices into a dense, regularized Gaussian latent space using FAVA's VAE (single hidden layer, ReLU encoder, sigmoid decoder, 0.9 reconstruction / 0.1 KL-to-N(1, 0.1) weighting, Adam at lr 1e-3). The latent embeddings are the basis from which downstream correlation-based functional association networks are inferred.

## When to use

Use when input is a high-dimensional protein x condition omics matrix (TSV/CSV, AnnData-compatible) with sparseness and redundancy, and a dense low-dimensional per-protein embedding is needed before network inference.

## When NOT to use

- The data are already a low-dimensional or dense embedding — FAVA exists to compress high-dimensional, sparse, redundant data
- The goal is direct co-expression correlation on the raw high-dimensional matrix; the article states this approach works poorly due to sparseness and redundancy
- Input rows are conditions/cells rather than proteins, since the embedding is one latent vector per input protein

## Inputs

- High-dimensional protein x condition matrix as TSV or CSV (selected via the -t option), AnnData-compatible
- Inputs scaled to the 0-1 range (matched to the sigmoid decoder output)

## Outputs

- vae_model.pt — trained VAE model checkpoint
- latent_embeddings.tsv — one regularized Gaussian latent vector per input protein

## How to apply

Load the input file (TSV or CSV per the -t option) with favapy's load_data, which uses AnnData-compatible handling and expects inputs scaled to 0-1. Build the VAE with one hidden layer (ReLU activations), linear mu and sigma output layers, and a sigmoid decoder output; the hidden and latent dimensionalities default to sizes derived from the input dimensions. Train for 50 epochs at batch size 32 with the Adam optimizer (learning rate 1e-3), minimizing a weighted loss of 0.9 reconstruction error plus 0.1 KL divergence against a normal prior N(1, 0.1). Encode all proteins through the trained encoder to obtain the regularized Gaussian latent space, then save the checkpoint as vae_model.pt and the embeddings as latent_embeddings.tsv (one latent vector per input protein). Correlations computed in this latent space yield improved association networks compared to the original high-dimensional representation, which suffers from sparseness and redundancy.

## Related tools

- **favapy** (Python/CLI package that loads data, builds and trains the VAE, and emits the latent embeddings) — https://github.com/mikelkou/fava
- **AnnData** (Input data structure used by favapy's load_data for AnnData-compatible handling of the omics matrix) — https://github.com/mikelkou/fava
- **FAVA** (The VAE-based method implemented by favapy that performs the dimensionality reduction) — https://github.com/mikelkou/fava

## Examples

```
favapy <path-to-data-file> <path-to-save-output>
```

## Evaluation signals

- Structural check: the VAE has exactly one hidden layer with ReLU activations, linear mu and sigma output layers, and a sigmoid decoder output
- Loss check: training minimizes 0.9 * reconstruction error + 0.1 * KL divergence against N(mean=1, std=0.1), with Adam at lr 1e-3
- Training config check: 50 epochs at batch size 32
- File check: latent_embeddings.tsv contains exactly one latent vector per input protein; vae_model.pt is written
- Latent space check: embeddings form a dense, low-dimensional regularized Gaussian latent space suitable for downstream correlation computation

## Limitations

- Latent and hidden dimensions default to sizes derived from the input rather than being fixed, so embedding dimensionality varies with the input matrix
- Inputs must be scaled to 0-1 to match the sigmoid decoder output; unscaled data are mismatched to the architecture
- Simple co-expression on the original high-dimensional representation works poorly; the benefit is contingent on the latent-space compression of sparse, redundant data

## Evidence

- [abstract] FAVA (Functional Associations using Variational Autoencoders), which compresses high-dimensional data into a low-dimensional space: "we have developed FAVA (Functional Associations using Variational Autoencoders), which compresses high-dimensional data into a low-dimensional space"
- [intro] Calculating correlations in this latent space results in much improved networks compared to the original representation: "Calculating correlations in this latent space results in much improved networks compared to the original representation"
- [methods] Run FAVA from the command line: [evidence span withheld — anchor status: not_found]
- [abstract] FAVA functions within the scverse ecosystem, employing AnnData as its input source: "FAVA functions within the scverse ecosystem, employing AnnData as its input source"
