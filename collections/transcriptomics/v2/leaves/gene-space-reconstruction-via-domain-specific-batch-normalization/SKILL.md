---
name: gene-space-reconstruction-via-domain-specific-batch-normalization
description: Use when integrating multi-batch scRNA-seq data (e.g., log1p-normalized
  pancreas matrix, 2000 HVGs) and you need corrected outputs in the original gene
  space instead of only latent embeddings, avoiding the interpretability loss of latent-space-only
  methods.
when_to_use_negative:
- When CITE-seq protein data with only a few hundred features is the second modality
  — scCobra's shared encoder struggles to identify shared features among so few features
- When input is bulk RNA-seq or already-integrated data with no batch labels — DSBN
  requires per-domain normalization across batches
- When you need multi-omic peak-level scATAC-seq integration without conversion —
  scCobra relies on gene activity matrices and may lose information in the conversion
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scCobra
  role: VAE-GAN encoder-decoder model with DSBN (BN1) layer in the decoder producing
    gene-space corrected outputs
  repo: github.com/mcgilldinglab/scCobra
- name: Scanpy
  role: Loading and preprocessing the log1p-normalized pancreas expression matrix
    with 2000 HVGs and batch labels
- name: PyTorch
  role: Implementation of the encoder-decoder, DSBN layer, domain discriminator, and
    contrastive heads
- name: scIB
  role: Validation of batch mixing via Leiden clustering across resolutions 0.1–2.0
    in 0.1 increments
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:095b834c542936f35d580a7609201da234087c50dce5d814a60eb59a7962d858
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyTorch
  - Scanpy
  - scCobra
  - scIB
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# gene-space-reconstruction-via-domain-specific-batch-normalization

## Summary

Reconstruct scRNA-seq data in the original gene space by placing a Domain-Specific Batch Normalization (DSBN) layer (BN1) in the scCobra VAE-GAN decoder, so batch-corrected outputs remain gene-interpretable rather than latent-only.

## When to use

Use when integrating multi-batch scRNA-seq data (e.g., log1p-normalized pancreas matrix, 2000 HVGs) and you need corrected outputs in the original gene space instead of only latent embeddings, avoiding the interpretability loss of latent-space-only methods.

## When NOT to use

- When CITE-seq protein data with only a few hundred features is the second modality — scCobra's shared encoder struggles to identify shared features among so few features
- When input is bulk RNA-seq or already-integrated data with no batch labels — DSBN requires per-domain normalization across batches
- When you need multi-omic peak-level scATAC-seq integration without conversion — scCobra relies on gene activity matrices and may lose information in the conversion

## Inputs

- Preprocessed single-cell expression matrix (log1p-normalized, 2000 highly variable genes) with batch labels, e.g. .h5ad loaded via Scanpy
- Batch label annotations per cell for the DSBN domains
- Optionally scATAC-seq converted to gene activity matrix (via MAESTRO) for multimodal integration

## Outputs

- Batch-corrected latent embeddings for all cells
- Decoder BN1 outputs in the original gene space (batch-corrected expression)
- sccobra_pancreas_outputs.h5ad containing latent and BN1 layers with batch and cell metadata

## How to apply

Implement the scCobra encoder-decoder VAE-GAN in PyTorch and place the domain-specific batch normalization layer (BN1) in the decoder so that corrected outputs are emitted in the original gene space. Load a preprocessed expression matrix (log1p-normalized, 2000 highly variable genes with batch labels) via Scanpy, encode all batches into a unified latent low-dimensional space, and run the three-phase optimization: train the domain discriminator, then train the generator against it, then jointly optimize contrastive (l_cell, l_cluster, L_c) and VAE objectives with BCE reconstruction loss and KL regularization. Extract both the batch-corrected latent embeddings and the decoder's BN1 original-space outputs for all cells, writing them with batch and cell metadata to an .h5ad file. The DSBN design is the rationale for preserving interpretability: corrections happen per-domain in the decoder while reconstruction stays anchored to gene-level features, unlike methods that cannot be applied in the original feature space.

## Related tools

- **scCobra** (VAE-GAN encoder-decoder model with DSBN (BN1) layer in the decoder producing gene-space corrected outputs) — github.com/mcgilldinglab/scCobra
- **Scanpy** (Loading and preprocessing the log1p-normalized pancreas expression matrix with 2000 HVGs and batch labels)
- **PyTorch** (Implementation of the encoder-decoder, DSBN layer, domain discriminator, and contrastive heads)
- **scIB** (Validation of batch mixing via Leiden clustering across resolutions 0.1–2.0 in 0.1 increments)

## Examples

```
git clone https://github.com/mcgilldinglab/scCobra.git && conda create -n scCobra conda-forge::python=3.10 && pip install scanpy scib && pip3 install torch torchvision torchaudio torch_optimizer
```

## Evaluation signals

- Output .h5ad file exists and contains latent and BN1 layers with row counts equal to the number of input cells
- BN1 outputs are in the original 2000-gene space (same gene dimensionality as input matrix)
- Leiden clustering at resolutions 0.1–2.0 (scIB pipeline) shows batch mixing in the corrected embeddings
- Latent embeddings from all batches occupy a unified low-dimensional space, verified by domain discriminator behavior after three-phase training
- Corrected gene-space output preserves biological signal (no over-correction of cell-type-specific differences such as healthy vs. disease states)

## Limitations

- Multimodal integration depends on converting scATAC-seq peak matrices to gene activity matrices, which is contingent on the conversion algorithm and may lead to information loss
- The shared encoder requires selecting shared features between modalities; CITE-seq protein data with only a few hundred features makes this challenging
- scCobra does not assume a gene expression distribution, but the DSBN decoder requires batch/domain labels to be known for each cell

## Evidence

- [results] DSBN decoder rationale: [evidence span withheld — anchor status: not_found]
- [methods] methods three-phase training: "git clone https://github.com/mcgilldinglab/scCobra.git"
- [intro] interpretability problem: "certain methods and their original implementations are challenging to apply in the original feature space8,14,17,18, leading to a loss of model interpretability"
- [discussion] clustering validation: "The scIB pipeline applied Leiden clustering across a range of resolutions from 0.1 to 2.0 (in increments of 0.1)"
- [discussion] preprocessing: "we identified 2000 highly variable genes for downstream analysis"
