---
name: vae-gan-neural-network-implementation
description: 'Use when multi-batch scRNA-seq count matrices (e.g., pancreas: 16382
  cells, 14 cell types, 9 batches) must be integrated via a contrastive VAE-GAN whose
  decoder emits batch-corrected outputs in the original gene space.'
when_to_use_negative:
- CITE-seq protein data with only a few hundred features, where identifying shared
  features for scCobra's shared encoder is challenging
- Label transfer to query datasets from a different tissue type or without similar
  cell types, which scCobra requires for optimal performance
- scATAC-seq peak matrices where conversion to a gene activity matrix (e.g., via MAESTRO)
  may lead to information loss
tools:
- name: scCobra
  role: The contrastive VAE-GAN model with DSBN decoder being implemented; source
    code cloned from GitHub
  repo: https://github.com/mcgilldinglab/scCobra
- name: Scanpy
  role: 'Preprocessing: cell/gene filtering, log1p normalization, and loading the
    batch-labeled expression matrix'
- name: PyTorch
  role: Framework for implementing the encoder-decoder, DSBN layer (BN1), domain discriminator,
    and contrastive heads
- name: scIB
  role: Leiden clustering across resolutions 0.1–2.0 (0.1 increments) to validate
    batch mixing qualitatively
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:7faba914dbb06a2647bd1d102180d2092b971dad6e3b88be4574510da006efeb
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# vae-gan-neural-network-implementation

## Summary

Reconstruct scCobra's VAE-GAN deep generative network — an encoder mapping multi-batch scRNA-seq into a unified latent space and a decoder with a Domain-Specific Batch Normalization (DSBN) layer — to batch-correct single-cell data without assuming a gene expression distribution.

## When to use

Use when multi-batch scRNA-seq count matrices (e.g., pancreas: 16382 cells, 14 cell types, 9 batches) must be integrated via a contrastive VAE-GAN whose decoder emits batch-corrected outputs in the original gene space.

## When NOT to use

- CITE-seq protein data with only a few hundred features, where identifying shared features for scCobra's shared encoder is challenging
- Label transfer to query datasets from a different tissue type or without similar cell types, which scCobra requires for optimal performance
- scATAC-seq peak matrices where conversion to a gene activity matrix (e.g., via MAESTRO) may lead to information loss

## Inputs

- log1p-normalized scRNA-seq expression matrix with 2000 highly variable genes and batch labels (Scanpy .h5ad), e.g., the pancreas dataset (16382 cells, 14 cell types, 9 batches)
- Batch labels per cell for DSBN and domain discrimination

## Outputs

- Batch-corrected latent embeddings for all cells
- Decoder BN1 outputs in the original gene space
- sccobra_pancreas_outputs.h5ad containing latent and BN1 layers plus batch and cell metadata

## How to apply

Preprocess with Scanpy: filter cells expressing fewer than 200 genes, genes detected in fewer than 3 cells, and cells with >5% mitochondrial fraction; normalize and apply log1p; select 2000 highly variable genes. Implement the scCobra encoder-decoder in PyTorch, placing the domain-specific batch normalization layer (BN1) in the decoder so corrected outputs can be emitted in the original gene space, and attach the domain discriminator on latent embeddings plus contrastive heads computing cell-level loss l_cell, cluster-level loss l_cluster, and the combined contrastive loss L_c, alongside binary cross-entropy reconstruction loss and KL divergence regularization. Optimize in three distinct phases: first train the domain discriminator, then train the generator against it, then jointly optimize the contrastive and VAE objectives. Extract the batch-corrected latent embeddings and the decoder's BN1 original-space outputs for all cells and write them with batch and cell metadata to an .h5ad file. The rationale is that scCobra mitigates batch effects and minimizes over-correction without gene expression distribution assumptions, avoiding the MNN over-correction and negative-binomial assumptions of alternatives like scVI.

## Related tools

- **scCobra** (The contrastive VAE-GAN model with DSBN decoder being implemented; source code cloned from GitHub) — https://github.com/mcgilldinglab/scCobra
- **Scanpy** (Preprocessing: cell/gene filtering, log1p normalization, and loading the batch-labeled expression matrix)
- **PyTorch** (Framework for implementing the encoder-decoder, DSBN layer (BN1), domain discriminator, and contrastive heads)
- **scIB** (Leiden clustering across resolutions 0.1–2.0 (0.1 increments) to validate batch mixing qualitatively)

## Examples

```
pip install scanpy scib; git clone https://github.com/mcgilldinglab/scCobra.git
```

## Evaluation signals

- Output .h5ad file exists and contains both latent and BN1 layers with row counts equal to the number of input cells
- Optimization ran in three distinct phases: discriminator trained first, then generator against it, then joint contrastive + VAE optimization
- Leiden clustering across resolutions 0.1–2.0 in 0.1 increments (scIB pipeline) shows qualitative batch mixing
- Latent embeddings organize by cell type rather than batch (contrastive loss L_c combining l_cell and l_cluster is minimized)
- Corrected outputs are emitted in the original gene space via the decoder's BN1 layer, preserving interpretability at the gene level

## Limitations

- Benchmarking methods (Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER) were compared with default parameters, which may not reflect their tuned performance
- Over-correction scores were derived from two-dimensional UMAP embeddings, introducing dependence on UMAP
- Label transfer requires reference and query datasets to originate from the same tissue type or contain similar cell types
- Multimodal integration of scATAC-seq relies on converting peak matrices to gene activity matrices, which is contingent on the conversion algorithm and may lead to information loss

## Evidence

- [readme] scCobra mitigates batch effects without distribution assumptions: "By leveraging a deep generative model that combines a contrastive neural network with domain adaptation, scCobra mitigates batch effects and minimizes over-correction without gene expression"
- [discussion] Filtering thresholds: "we first filtered out cells that expressed fewer than 200 genes and genes that were detected in fewer than 3 cells"
- [discussion] Normalization and HVG selection: "This was followed by a log1p transformation to stabilize the variance"
- [discussion] Three-phase optimization: "optimization performed in three distinct phases"
- [discussion] scIB validation via Leiden clustering: "The scIB pipeline applied Leiden clustering across a range of resolutions from 0.1 to 2.0 (in increments of 0.1)"
- [readme] Environment setup from README: "# Install scanpy scib
pip install scanpy scib"
