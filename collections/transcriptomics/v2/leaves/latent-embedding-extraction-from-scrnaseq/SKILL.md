---
name: latent-embedding-extraction-from-scrnaseq
description: 'Use when you have log1p-normalized, multi-batch scRNA-seq data (e.g.,
  pancreas: 16382 cells, 14 cell types, 9 batches; 2000 highly variable genes with
  batch labels) and need harmonized latent embeddings plus corrected outputs in original
  gene space.'
when_to_use_negative:
- Input is single-batch scRNA-seq with no batch effects to mitigate — the domain discriminator
  and DSBN machinery target cross-batch harmonization
- Multi-omic integration where modalities share very few features, e.g., CITE-seq
  protein data with only a few hundred features, which makes it challenging to identify
  shared features and optimize the model
- Online label transfer where the reference and query datasets come from different
  tissue types — scCobra requires the same tissue type or similar cell types for optimal
  performance
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scCobra
  role: Deep generative contrastive VAE-GAN with DSBN decoder; its encoder produces
    the unified latent embeddings and its BN1 decoder emits original-gene-space corrected
    outputs
  repo: https://github.com/mcgilldinglab/scCobra
- name: Scanpy
  role: Loading and preprocessing the expression matrix (filtering, log1p normalization,
    HVG selection) and writing the output .h5ad
- name: PyTorch
  role: Implementing the encoder-decoder VAE-GAN, domain discriminator, and contrastive
    heads; running the three-phase adversarial optimization
- name: scIB
  role: Post-hoc evaluation of batch mixing via Leiden clustering across resolutions
    0.1–2.0 in 0.1 increments
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:714994701b3b3b0f9c3443000584440964e2913df905b10f49ae2ccb9ce636bf
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# latent-embedding-extraction-from-scrnaseq

## Summary

Extracts batch-corrected cell embeddings by training scCobra, a contrastive VAE-GAN whose encoder maps multi-batch scRNA-seq into a unified low-dimensional latent space and whose DSBN-equipped decoder (BN1) reconstructs outputs in the original gene space, without assuming a gene expression distribution.

## When to use

Use when you have log1p-normalized, multi-batch scRNA-seq data (e.g., pancreas: 16382 cells, 14 cell types, 9 batches; 2000 highly variable genes with batch labels) and need harmonized latent embeddings plus corrected outputs in original gene space.

## When NOT to use

- Input is single-batch scRNA-seq with no batch effects to mitigate — the domain discriminator and DSBN machinery target cross-batch harmonization
- Multi-omic integration where modalities share very few features, e.g., CITE-seq protein data with only a few hundred features, which makes it challenging to identify shared features and optimize the model
- Online label transfer where the reference and query datasets come from different tissue types — scCobra requires the same tissue type or similar cell types for optimal performance

## Inputs

- log1p-normalized scRNA-seq expression matrix with batch labels (e.g., 2000 highly variable genes, .h5ad via Scanpy)
- batch labels per cell (e.g., pancreas dataset: 16382 cells, 9 batches)
- cell metadata (e.g., cell types for downstream Leiden/scIB evaluation)

## Outputs

- batch-corrected latent embeddings per cell (unified low-dimensional space)
- decoder BN1 outputs in the original gene space (batch-corrected expression)
- output .h5ad file (e.g., sccobra_pancreas_outputs.h5ad) containing latent and BN1 layers plus batch and cell metadata

## How to apply

Preprocess with Scanpy: filter cells expressing fewer than 200 genes, genes detected in fewer than 3 cells, and cells with >5% mitochondrial fraction; normalize and log1p-transform, then retain 2000 highly variable genes ('vst' method per the Seurat v3.2.3 convention). Implement the scCobra encoder-decoder in PyTorch with the domain-specific batch normalization layer (BN1) in the decoder so corrected outputs can be emitted in the original gene space, and attach the domain discriminator on latent embeddings plus contrastive heads computing cell-level loss l_cell, cluster-level loss l_cluster, and combined L_c, alongside BCE reconstruction loss and KL divergence regularization. Optimize in three phases: first train the domain discriminator, then train the generator against it, then jointly optimize the contrastive and VAE objectives. Extract the batch-corrected latent embeddings and decoder BN1 original-space outputs for all cells and write them with batch/cell metadata to an .h5ad file (e.g., sccobra_pancreas_outputs.h5ad).

## Related tools

- **scCobra** (Deep generative contrastive VAE-GAN with DSBN decoder; its encoder produces the unified latent embeddings and its BN1 decoder emits original-gene-space corrected outputs) — https://github.com/mcgilldinglab/scCobra
- **Scanpy** (Loading and preprocessing the expression matrix (filtering, log1p normalization, HVG selection) and writing the output .h5ad)
- **PyTorch** (Implementing the encoder-decoder VAE-GAN, domain discriminator, and contrastive heads; running the three-phase adversarial optimization)
- **scIB** (Post-hoc evaluation of batch mixing via Leiden clustering across resolutions 0.1–2.0 in 0.1 increments)

## Examples

```
git clone https://github.com/mcgilldinglab/scCobra.git; pip install scanpy scib torch
```

## Evaluation signals

- Output file exists and contains both latent and BN1 layers
- Row counts of the latent and BN1 layers equal the number of input cells
- Latent embeddings from all batches occupy a unified low-dimensional space rather than batch-separated clusters
- Qualitative batch mixing confirmed by Leiden clustering across resolutions 0.1–2.0 (0.1 increments) under the scIB pipeline, as in the paper's integration evaluation
- Corrected outputs are emitted in the original gene space (2000 HVGs), confirming the decoder's BN1 placement works as intended

## Limitations

- Benchmark comparisons (Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER) were run with default parameters, which may not reflect their tuned performance
- Converting scATAC-seq peak matrices into gene activity matrices (via MAESTRO) for multimodal integration is contingent on the conversion algorithm and may lead to information loss
- Over-correction scores were derived from two-dimensional UMAP embeddings, introducing dependence on UMAP

## Evidence

- [methods] scCobra uses an encoder to map scRNA-seq data from different batches into a unified latent low-dimensional space, and a decoder equipped with a Domain-Specific Batch Normalization (DSBN) layer: [evidence span withheld — anchor status: not_found]
- [methods] optimization performed in three distinct phases: "optimization performed in three distinct phases"
- [readme] By leveraging a deep generative model that combines a contrastive neural network with domain adaptation, scCobra mitigates batch effects and minimizes over-correction without gene expression distribution assumptions.: "By leveraging a deep generative model that combines a contrastive neural network with domain adaptation, scCobra mitigates batch effects and minimizes over-correction without gene expression"
- [discussion] The scIB pipeline applied Leiden clustering across a range of resolutions from 0.1 to 2.0 (in increments of 0.1): "The scIB pipeline applied Leiden clustering across a range of resolutions from 0.1 to 2.0 (in increments of 0.1)"
