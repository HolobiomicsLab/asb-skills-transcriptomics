---
name: kl-divergence-loss-weighting-regularization
description: Use when compressing high-dimensional, sparse omics data (e.g., protein
  abundances scaled 0-1) into a low-dimensional latent space with a VAE, and you need
  an explicitly regularized Gaussian latent representation to avoid sparseness/redundancy
  artifacts before computing protein associations.
when_to_use_negative:
- The input data are already low-dimensional or dense — FAVA's motivation is compressing
  high-dimensional data where co-expression analysis fails due to sparseness and redundancy
- You need to reconstruct inputs rather than produce a regularized latent embedding
  — the 0.1 KL weight intentionally trades reconstruction fidelity for prior-constrained
  latent structure
- Downstream analysis requires a non-Gaussian latent prior — the KL term is defined
  against a N(1, 0.1) normal prior specific to this implementation
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3307
tools:
- name: favapy
  role: Python package implementing the FAVA VAE (VAE, cook, pairs_after_cutoff, load_data
    members); installed via pip and used to train the weighted-loss VAE and produce
    latent embeddings
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
  source_content_hash: sha256:4c8004b11e142688128cdf42a1d59d1c124ff866206a6cb5780e952f14b355e6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - AnnData
  - FAVA
  - Python
  - favapy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# KL divergence loss weighting regularization

## Summary

A VAE training scheme from FAVA that combines reconstruction error and KL-divergence regularization in a 0.9/0.1 weighted sum, regularizing the latent space toward a Gaussian prior N(1, 0.1) so that high-dimensional proteomics/scRNA-seq data are compressed into a dense, low-dimensional latent space suitable for downstream correlation-based network inference.

## When to use

Use when compressing high-dimensional, sparse omics data (e.g., protein abundances scaled 0-1) into a low-dimensional latent space with a VAE, and you need an explicitly regularized Gaussian latent representation to avoid sparseness/redundancy artifacts before computing protein associations.

## When NOT to use

- The input data are already low-dimensional or dense — FAVA's motivation is compressing high-dimensional data where co-expression analysis fails due to sparseness and redundancy
- You need to reconstruct inputs rather than produce a regularized latent embedding — the 0.1 KL weight intentionally trades reconstruction fidelity for prior-constrained latent structure
- Downstream analysis requires a non-Gaussian latent prior — the KL term is defined against a N(1, 0.1) normal prior specific to this implementation

## Inputs

- High-dimensional omics data matrix (TSV or CSV, loaded via favapy load_data with AnnData-compatible handling), values scaled to 0-1
- Protein-by-condition abundance matrix from sources such as the Human Protein Atlas (scRNA-seq) or PRIDE (proteomics)

## Outputs

- Trained VAE model checkpoint saved as vae_model.pt
- Latent embeddings table (latent_embeddings.tsv) with one latent vector per input protein

## How to apply

Build a VAE with a single hidden layer using ReLU activations, linear activation on the mu and sigma output layers, and a sigmoid decoder output (appropriate for inputs scaled to 0-1). Train with the Adam optimizer at learning rate 1e-3 for 50 epochs at batch size 32, minimizing a weighted loss of 0.9 * reconstruction error + 0.1 * KL divergence, where the KL term is computed against a normal prior with mean 1 and std 0.1. The rationale is that raw omics data suffer from high sparseness and redundancy, and the KL regularization pressure produces a dense, low-dimensional latent space in which correlations yield much improved networks compared to the original representation. After training, encode all proteins through the model to obtain one latent vector per protein.

## Related tools

- **favapy** (Python package implementing the FAVA VAE (VAE, cook, pairs_after_cutoff, load_data members); installed via pip and used to train the weighted-loss VAE and produce latent embeddings) — https://github.com/mikelkou/fava
- **AnnData** (Input data structure used by favapy load_data for AnnData-compatible handling of the omics matrix)

## Examples

```
pip install favapy && favapy <path-to-data-file> <path-to-save-output>
```

## Evaluation signals

- Structural check: the model has exactly one hidden layer with ReLU activations, linear mu/sigma layers, and a sigmoid decoder output
- Loss check: training minimizes 0.9 * reconstruction + 0.1 * KL divergence against a N(1, 0.1) prior, with Adam at lr 1e-3, 50 epochs, batch size 32
- latent_embeddings.tsv contains one latent vector per input protein (row count matches input protein count)
- vae_model.pt checkpoint is produced and reloadable by the favapy VAE class

## Limitations

- Latent dimensions and hidden-layer size default to values derived from the input data rather than fixed constants, so architecture must be adapted to dataset size
- Sigmoid decoder output assumes inputs are scaled to 0-1; unscaled inputs violate this assumption
- The described hyperparameters (0.9/0.1 weighting, N(1, 0.1) prior, 50 epochs, batch 32) are specific to the FAVA implementation and were validated on Human Protein Atlas and PRIDE-derived data

## Evidence

- [abstract] FAVA (Functional Associations using Variational Autoencoders), which compresses high-dimensional data into a low-dimensional space: "we have developed FAVA (Functional Associations using Variational Autoencoders), which compresses high-dimensional data into a low-dimensional space"
- [intro] Calculating correlations in this latent space results in much improved networks compared to the original representation: "Calculating correlations in this latent space results in much improved networks compared to the original representation"
- [intro] this approach does not work well due to high sparseness and redundancy: "this approach does not work well due to high sparseness (many transcripts/proteins are not consistently observed in each cell/sample) and redundancy"
- [methods] favapy CLI entry point: "Run FAVA from the command line as follows:"
- [readme] FAVA: Functional Associations using Variational Autoencoders: "# FAVA: Functional Associations using Variational Autoencoders"
