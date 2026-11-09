---
name: contrastive-loss-design-cell-and-cluster-level
description: 'Use when implementing or reproducing scCobra-style integration: you
  must attach contrastive heads computing l_cell, l_cluster, and combined L_c on latent
  embeddings alongside BCE reconstruction, KL regularization, and a domain discriminator.'
when_to_use_negative:
- Reference and query datasets come from different tissue types or share few cell
  types — scCobra's label transfer requires the same tissue type or similar cell types
  for optimal performance
- The goal is multi-omic integration with CITE-seq protein data, where only a few
  hundred shared features make contrastive optimization challenging
- The data are bulk RNA-seq or already batch-harmonized feature tables where no latent
  contrastive embedding is needed
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scCobra
  role: Encoder-decoder VAE-GAN whose contrastive heads (l_cell, l_cluster, L_c) and
    DSBN decoder produce the batch-corrected embeddings
  repo: https://github.com/mcgilldinglab/scCobra
- name: PyTorch
  role: Framework for implementing the contrastive heads, domain discriminator, and
    three-phase adversarial training
- name: Scanpy
  role: Loading and log1p preprocessing of the single-cell expression matrix with
    batch labels
- name: scIB
  role: Leiden clustering across resolutions 0.1–2.0 in 0.1 increments to evaluate
    batch mixing of the learned embeddings
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:215ab64159017788071edbcbf1a5bc075a05756be8df2fdc59262765f1bffc3a
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# contrastive-loss-design-cell-and-cluster-level

## Summary

Design of scCobra's combined contrastive loss, which sums a cell-level loss l_cell and a cluster-level loss l_cluster into L_c to shape batch-harmonized latent embeddings, trained jointly with VAE/GAN objectives in a three-phase optimization.

## When to use

Use when implementing or reproducing scCobra-style integration: you must attach contrastive heads computing l_cell, l_cluster, and combined L_c on latent embeddings alongside BCE reconstruction, KL regularization, and a domain discriminator.

## When NOT to use

- Reference and query datasets come from different tissue types or share few cell types — scCobra's label transfer requires the same tissue type or similar cell types for optimal performance
- The goal is multi-omic integration with CITE-seq protein data, where only a few hundred shared features make contrastive optimization challenging
- The data are bulk RNA-seq or already batch-harmonized feature tables where no latent contrastive embedding is needed

## Inputs

- Preprocessed single-cell expression matrix (log1p-normalized, 2000 highly variable genes, e.g. pancreas dataset with 16382 cells across 9 batches)
- Batch labels per cell
- Cell metadata

## Outputs

- Batch-corrected latent embeddings for all cells
- Decoder BN1 outputs in the original gene space
- sccobra_pancreas_outputs.h5ad with latent and BN1 layers plus batch/cell metadata

## How to apply

Implement the scCobra encoder-decoder VAE-GAN in PyTorch, placing the Domain-Specific Batch Normalization layer (BN1) in the decoder so corrected outputs are emitted in the original gene space. Attach the domain discriminator operating on latent embeddings and the contrastive heads computing the cell-level loss l_cell, the cluster-level loss l_cluster, and the combined contrastive loss L_c, alongside binary cross-entropy reconstruction loss and KL divergence regularization. Run the three-phase optimization: first train the domain discriminator, then train the generator against it, then jointly optimize the contrastive and VAE objectives. Apply this to log1p-normalized inputs restricted to 2000 highly variable genes with batch labels, then extract the batch-corrected latent embeddings and BN1 original-space outputs for all cells. The design avoids assuming a gene expression distribution (e.g., negative binomial as in scVI) and avoids the over-correction typical of MNN-based methods by enforcing both cell-level and cluster-level contrastive structure.

## Related tools

- **scCobra** (Encoder-decoder VAE-GAN whose contrastive heads (l_cell, l_cluster, L_c) and DSBN decoder produce the batch-corrected embeddings) — https://github.com/mcgilldinglab/scCobra
- **PyTorch** (Framework for implementing the contrastive heads, domain discriminator, and three-phase adversarial training)
- **Scanpy** (Loading and log1p preprocessing of the single-cell expression matrix with batch labels)
- **scIB** (Leiden clustering across resolutions 0.1–2.0 in 0.1 increments to evaluate batch mixing of the learned embeddings)

## Examples

```
git clone https://github.com/mcgilldinglab/scCobra.git && cd scCobra && jupyter notebook run_integration.ipynb
```

## Evaluation signals

- Output .h5ad exists and contains both 'latent' and 'BN1' layers whose row counts equal the number of input cells
- Latent embeddings show batch mixing: Leiden clustering at resolutions 0.1–2.0 (0.1 increments, scIB pipeline) recovers consistent cell-type structure across batches
- Contrastive loss components l_cell, l_cluster, and combined L_c decrease and converge during the joint optimization phase
- Domain discriminator performance during phase 1 vs. generator performance in phase 2 indicates adversarial equilibrium was reached
- Integrated embeddings preserve cell-type labels (e.g. 14 pancreas cell types) rather than collapsing or over-correcting batch-specific biology

## Limitations

- scCobra's shared encoder requires selecting shared features between modalities; CITE-seq protein data with only a few hundred features makes this challenging
- Multimodal integration relies on converting scATAC-seq peak matrices into gene activity matrices, which is contingent on the conversion algorithm and may lead to information loss
- Label transfer requires reference and query datasets to originate from the same tissue type or contain similar cell types
- Over-correction scores were derived from two-dimensional UMAP embeddings, introducing dependence on UMAP

## Evidence

- [other] contrastive heads and losses: [evidence span withheld — anchor status: not_found]
- [discussion] three-phase training: "optimization performed in three distinct phases"
- [readme] contrastive network rationale: "By leveraging a deep generative model that combines a contrastive neural network with domain adaptation, scCobra mitigates batch effects and minimizes over-correction"
- [other] encoder and DSBN decoder: [evidence span withheld — anchor status: not_found]
