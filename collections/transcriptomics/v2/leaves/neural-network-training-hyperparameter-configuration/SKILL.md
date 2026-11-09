---
name: neural-network-training-hyperparameter-configuration
description: 'Use when building the VAE dimensionality-reduction stage of FAVA: input
  is a high-dimensional protein-by-condition omics matrix (TSV/CSV, AnnData-compatible)
  that must be compressed to a low-dimensional latent space before correlation-based
  network inference.'
when_to_use_negative:
- Input is not high-dimensional sparse omics data (FAVA targets sparse, redundant
  high-dimensional scRNA-seq/proteomics matrices; simple low-dimensional tables do
  not need VAE compression)
- The goal is direct co-expression analysis on the raw data without latent-space compression
  (the article states simple co-expression on raw data works poorly, but that is a
  different workflow than training the VAE)
- A pretrained vae_model.pt and latent_embeddings.tsv already exist and only the downstream
  cook/pairs_after_cutoff network-inference steps are needed
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: favapy
  role: Python package providing load_data, the VAE class, and the CLI entry point
    used to train the VAE and write latent embeddings
  repo: https://github.com/mikelkou/fava
- name: AnnData
  role: Input data structure used by favapy load_data for AnnData-compatible handling
    of the omics matrix
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:ce23ab28ea4bae6b617145b0fa912397c859b25530ccac0d8495879033292660
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
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# neural-network-training-hyperparameter-configuration

## Summary

Configures and trains FAVA's variational autoencoder that compresses high-dimensional scRNA-seq/proteomics data into a regularized Gaussian latent space, using a single-hidden-layer encoder-decoder with a 0.9 reconstruction / 0.1 KL-divergence weighted loss against a normal prior N(1, 0.1). Correct hyperparameter choice (Adam lr 1e-3, 50 epochs, batch size 32) is what enables dense latent-space correlations that improve protein functional association network inference.

## When to use

Use when building the VAE dimensionality-reduction stage of FAVA: input is a high-dimensional protein-by-condition omics matrix (TSV/CSV, AnnData-compatible) that must be compressed to a low-dimensional latent space before correlation-based network inference.

## When NOT to use

- Input is not high-dimensional sparse omics data (FAVA targets sparse, redundant high-dimensional scRNA-seq/proteomics matrices; simple low-dimensional tables do not need VAE compression)
- The goal is direct co-expression analysis on the raw data without latent-space compression (the article states simple co-expression on raw data works poorly, but that is a different workflow than training the VAE)
- A pretrained vae_model.pt and latent_embeddings.tsv already exist and only the downstream cook/pairs_after_cutoff network-inference steps are needed

## Inputs

- Protein-by-condition omics data file (TSV or CSV, per the -t option), AnnData-compatible, from scRNA-seq (Human Protein Atlas) or proteomics (PRIDE) sources
- Feature-scaled values in the 0-1 range (sigmoid decoder assumes 0-1 scaled inputs)

## Outputs

- Trained VAE model checkpoint saved as vae_model.pt
- Latent embeddings saved as latent_embeddings.tsv, one latent vector per input protein

## How to apply

Load the input data (TSV or CSV, per the -t option) with the favapy load_data routine using AnnData-compatible handling. Build the VAE with one intermediate/hidden layer (ReLU activations), linear mu and sigma output layers, and a sigmoid decoder output suited to 0-1 scaled inputs; latent dimensions default to a size derived from the hidden layer. Train with the Adam optimizer at learning rate 1e-3 for 50 epochs at batch size 32, minimizing a weighted loss of 0.9 reconstruction error plus 0.1 KL divergence against a normal prior with mean 1 and std 0.1. Encode all proteins through the trained VAE to obtain the regularized Gaussian latent space; downstream correlations in this latent space yield much improved networks compared to the original high-dimensional representation.

## Related tools

- **favapy** (Python package providing load_data, the VAE class, and the CLI entry point used to train the VAE and write latent embeddings) — https://github.com/mikelkou/fava
- **AnnData** (Input data structure used by favapy load_data for AnnData-compatible handling of the omics matrix)

## Examples

```
favapy <path-to-data-file> <path-to-save-output>
```

## Evaluation signals

- Structural check: model has exactly one hidden layer, ReLU activations in the hidden layer, linear activation on mu/sigma layers, and a sigmoid decoder output
- Loss check: total loss equals 0.9 * reconstruction error + 0.1 * KL divergence against a normal prior N(mean=1, std=0.1)
- Optimizer check: Adam is used with learning rate 1e-3, trained for 50 epochs at batch size 32
- Output schema check: latent_embeddings.tsv contains exactly one latent vector per input protein
- Downstream check: correlations computed in the latent space should yield improved protein association networks compared to correlations in the original high-dimensional representation

## Limitations

- Architecture defaults (hidden-layer size, latent dimensions) are derived from the input dimensions rather than tuned per dataset; non-default inputs may require adjusting these derived sizes
- The sigmoid decoder output assumes 0-1 scaled input data; unscaled or differently normalized omics matrices will not match the training assumptions
- Training is fixed at 50 epochs / batch size 32 / lr 1e-3 in the described workflow; very large datasets (the paper reports processing > 0.5 million conditions) may need more epochs or adjusted capacity

## Evidence

- [abstract] FAVA (Functional Associations using Variational Autoencoders), which compresses high-dimensional data into a low-dimensional space: "we have developed FAVA (Functional Associations using Variational Autoencoders), which compresses high-dimensional data into a low-dimensional space"
- [methods] pip install favapy: "You can install FAVA using pip:

.. code-block:: bash

   pip install favapy"
- [abstract] FAVA functions within the scverse ecosystem, employing AnnData as its input source: "FAVA functions within the scverse ecosystem, employing AnnData as its input source"
- [intro] Calculating correlations in this latent space results in much improved networks compared to the original representation: "Calculating correlations in this latent space results in much improved networks compared to the original representation"
- [methods] Run FAVA from the command line as follows:

.. code-block:: bash

   favapy <path-to-data-file> <path-to-save-output>: "Run FAVA from the command line as follows:

.. code-block:: bash

   favapy <path-to-data-file> <path-to-save-output>"
