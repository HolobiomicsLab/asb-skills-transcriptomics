---
name: domain-adversarial-discriminator-optimization
description: Use when integrating multi-batch scRNA-seq data (e.g. 9-batch pancreas,
  10-batch immune) where batch effects must be removed while preserving inherent biological
  differences across batches, and over-correction by MNN-style methods is a concern.
when_to_use_negative:
- Input is not raw counts but an already batch-corrected or feature/embedding table
  — the three-phase VAE-GAN training expects raw preprocessed expression with batch
  labels
- Reference and query datasets for online label transfer come from different tissue
  types or lack similar cell types — the paper states label transfer requires same-tissue
  origin for optimal performance
- The modalities do not share enough features, e.g. CITE-seq protein data with only
  a few hundred features, where the shared-encoder design cannot be optimized effectively
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scCobra
  role: The deep generative VAE-GAN whose encoder, DSBN decoder, domain discriminator
    and contrastive heads are optimized in three phases
  repo: https://github.com/mcgilldinglab/scCobra
- name: PyTorch
  role: Framework used to implement and train the encoder-decoder, domain discriminator
    and contrastive heads across the three optimization phases
- name: Scanpy
  role: Used to load and log1p-normalize the preprocessed expression matrix with batch
    labels before training
- name: scIB
  role: 'Post-training validation: Leiden clustering across resolutions 0.1–2.0 (0.1
    increments) to confirm batch mixing'
  repo: https://scib.readthedocs.io/en/latest/index.html
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:5b289c80db6511b9136b39964bb23c5c3912315ea8da32f43e2ec3292eee2cd0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scCobra
  tools:
  - PyTorch
  - Scanpy
  - scCobra
  - scIB
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# domain-adversarial-discriminator-optimization

## Summary

Trains scCobra's VAE-GAN so a domain discriminator operating on latent embeddings is first optimized, then the generator is trained against it, and finally contrastive (l_cell, l_cluster, L_c) and VAE objectives (BCE reconstruction + KL divergence) are jointly optimized — removing batch effects without assuming a gene expression distribution.

## When to use

Use when integrating multi-batch scRNA-seq data (e.g. 9-batch pancreas, 10-batch immune) where batch effects must be removed while preserving inherent biological differences across batches, and over-correction by MNN-style methods is a concern.

## When NOT to use

- Input is not raw counts but an already batch-corrected or feature/embedding table — the three-phase VAE-GAN training expects raw preprocessed expression with batch labels
- Reference and query datasets for online label transfer come from different tissue types or lack similar cell types — the paper states label transfer requires same-tissue origin for optimal performance
- The modalities do not share enough features, e.g. CITE-seq protein data with only a few hundred features, where the shared-encoder design cannot be optimized effectively

## Inputs

- log1p-normalized scRNA-seq expression matrix restricted to 2000 highly variable genes, with per-cell batch labels (e.g. pancreas dataset: 16382 cells, 14 cell types, 9 batches)
- Preprocessed AnnData object from Scanpy with batch label column
- scCobra PyTorch model definition (encoder, decoder with DSBN layer BN1, domain discriminator, contrastive heads)

## Outputs

- Batch-corrected latent embeddings for all cells
- Decoder BN1 outputs in the original 2000-HVG gene space
- sccobra_pancreas_outputs.h5ad containing latent and BN1 layers plus batch and cell metadata

## How to apply

Preprocess the expression matrix per the paper's pipeline (filter cells expressing <200 genes and genes detected in <3 cells, remove cells with >5% mitochondrial fraction, Scanpy normalization with log1p, select 2000 highly variable genes) and retain batch labels. Implement the encoder-decoder VAE-GAN in PyTorch, placing the domain-specific batch normalization layer (BN1) in the decoder so corrected outputs are emitted in the original 2000-HVG gene space. Optimize in three distinct phases: first train the domain discriminator on latent embeddings, then train the generator against it, then jointly optimize the contrastive losses (cell-level l_cell, cluster-level l_cluster, combined L_c) with binary cross-entropy reconstruction loss and KL divergence regularization. The discriminator-then-generator ordering is the rationale for domain adaptation: it forces latent embeddings to be batch-invariant before the contrastive and reconstruction objectives refine biological structure. Extract the batch-corrected latent embeddings and decoder BN1 original-space outputs for all cells and write them with batch and cell metadata to an .h5ad file.

## Related tools

- **scCobra** (The deep generative VAE-GAN whose encoder, DSBN decoder, domain discriminator and contrastive heads are optimized in three phases) — https://github.com/mcgilldinglab/scCobra
- **PyTorch** (Framework used to implement and train the encoder-decoder, domain discriminator and contrastive heads across the three optimization phases)
- **Scanpy** (Used to load and log1p-normalize the preprocessed expression matrix with batch labels before training)
- **scIB** (Post-training validation: Leiden clustering across resolutions 0.1–2.0 (0.1 increments) to confirm batch mixing) — https://scib.readthedocs.io/en/latest/index.html

## Examples

```
git clone https://github.com/mcgilldinglab/scCobra.git && jupyter notebook run_integration.ipynb
```

## Evaluation signals

- Output .h5ad exists and contains latent and BN1 layers whose row counts equal the number of input cells
- Leiden clustering across resolutions 0.1–2.0 (0.1 increments) via the scIB pipeline shows batch mixing in the corrected embeddings
- Batch labels no longer separate the latent embeddings, while known cell-type structure is retained (no over-correction of inherent batch-associated biological differences)
- Latent embeddings are comparable across batches, which is prerequisite for the paper's online label transfer via a pre-trained KNN classifier

## Limitations

- Over-correction scores were derived from two-dimensional UMAP embeddings, introducing dependence on the UMAP method
- Benchmarked methods (Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER) were run with default parameters, so comparisons may not reflect tuned performance
- Multimodal integration with scATAC-seq relies on converting peak matrices to gene activity matrices via MAESTRO, contingent on the conversion algorithm and potentially lossy
