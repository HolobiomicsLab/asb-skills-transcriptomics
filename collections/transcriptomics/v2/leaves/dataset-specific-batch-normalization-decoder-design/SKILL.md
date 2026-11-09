---
name: dataset-specific-batch-normalization-decoder-design
description: Use when building a coupled-VAE to integrate heterogeneous and/or unpaired
  single-cell multi-omics datasets (e.
when_to_use_negative:
- Cells from the datasets were profiled together and share an identical feature space
  without domain shift — a plain VAE or horizontal integration with common genes only
  (up.Run(adata_cm=adata_cm)) suffices
- Datasets require vertical integration of the same cells or diagonal integration
  without common genes where the DSBN/common-plus-specific-gene design assumptions
  do not hold
- The task is label transfer or spatial deconvolution using the OT plan directly,
  rather than learning a decoder-based latent embedding
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: uniPort
  role: Provides the coupled-VAE with DSBN decoder implementation (model.vae.VAE,
    model.layer.Encoder, model.layer.Decoder, model.layer.DSBatchNorm) and the uniport.Run
    training entry point with Minibatch-UOT loss
  repo: https://github.com/caokai1073/uniport
- name: PyTorch
  role: Framework in which the shared encoder, DSBN decoder and per-dataset decoders
    are constructed and trained
- name: scanpy
  role: Reads and preprocesses the AnnData inputs (adata1.h5ad, adata2.h5ad, adata_cm.h5ad)
    with scanpy.read_h5ad
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-35094-8
    title: A unified computational framework for single-cell data integration with
      optimal transport
  source_content_hash: sha256:a22d95b5007d85d3d21436ed81586cc4842b34507ffe3c2e9554afc3c8ef5213
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/caokai1073/uniport
  tools:
  - PyTorch
  - scanpy
  - uniPort
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# Dataset-specific batch normalization decoder design

## Summary

Design pattern from uniPort's coupled-VAE in which reconstruction is handled by a DSBatchNorm (DSBN) decoder with per-dataset normalization statistics, paired with a shared probabilistic encoder and dataset-specific decoders, so heterogeneous single-cell datasets can share a latent space while preserving modality- or batch-specific structure.

## When to use

Use when building a coupled-VAE to integrate heterogeneous and/or unpaired single-cell multi-omics datasets (e.g., scRNA-seq with scATAC-seq gene activity scores) where a shared latent embedding is needed but each dataset has distinct feature distributions requiring per-dataset decoder normalization.

## When NOT to use

- Cells from the datasets were profiled together and share an identical feature space without domain shift — a plain VAE or horizontal integration with common genes only (up.Run(adata_cm=adata_cm)) suffices
- Datasets require vertical integration of the same cells or diagonal integration without common genes where the DSBN/common-plus-specific-gene design assumptions do not hold
- The task is label transfer or spatial deconvolution using the OT plan directly, rather than learning a decoder-based latent embedding

## Inputs

- Preprocessed PBMC AnnData matrices with dataset-specific HVGs (adata1.h5ad, adata2.h5ad)
- AnnData matrix of common highly variable genes (adata_cm.h5ad)
- Domain/batch labels identifying which dataset each cell belongs to

## Outputs

- Trained coupled-VAE with DSBN decoder (model.vae.VAE, model.layer.DSBatchNorm weights)
- Shared latent cell embeddings stored in adata.obs['latent']
- Training loss curve (training_loss_curve.png) of total loss L = -L_ELBO* + gamma * L_UOT*

## How to apply

Build the network in PyTorch mirroring model.layer.Encoder, model.layer.Decoder and model.layer.DSBatchNorm from the uniPort repo: a shared probabilistic encoder with reparameterization maps cells into a common latent space, while the decoder applies dataset-specific batch normalization (DSBN) so each domain's activations are normalized with its own statistics, supplemented by per-dataset decoders. Feed both a dataset-specific HVG matrix (adata1.h5ad / adata2.h5ad pattern) and a common-HVG matrix (adata_cm.h5ad); lambda_s=0.5 balances the common-gene and specific-gene contributions. Train by minimizing the total loss L = -L_ELBO* + gamma * L_UOT*, with lambda_recon=1.0, lambda_kl=0.5, lambda_ot=1.0, for up to iteration=30000 via uniport.Run; the Minibatch-UOT cost C_ij = ||mu_x - mu_y||^2 + ||sigma_x - sigma_y||^2 over encoder Gaussian outputs feeds gradients back to the shared encoder to align datasets. The rationale is that shared encoder parameters learn cross-dataset structure while DSBN decoders absorb domain-specific distributional shifts, enabling leverage of both highly variable common genes and dataset-specific genes.

## Related tools

- **uniPort** (Provides the coupled-VAE with DSBN decoder implementation (model.vae.VAE, model.layer.Encoder, model.layer.Decoder, model.layer.DSBatchNorm) and the uniport.Run training entry point with Minibatch-UOT loss) — https://github.com/caokai1073/uniport
- **PyTorch** (Framework in which the shared encoder, DSBN decoder and per-dataset decoders are constructed and trained)
- **scanpy** (Reads and preprocesses the AnnData inputs (adata1.h5ad, adata2.h5ad, adata_cm.h5ad) with scanpy.read_h5ad)

## Examples

```
adata = up.Run(adatas=[adata1, adata2], adata_cm=adata_cm, lambda_s=0.5, lambda_recon=1.0, lambda_kl=0.5, lambda_ot=1.0, iteration=30000)
```

## Evaluation signals

- Logged total loss L = -L_ELBO* + gamma * L_UOT* decreases monotonically (on balance) over the 30000 training iterations on the PBMC data
- Latent embeddings for all datasets are stored in adata.obs['latent'] after up.Run and share a single common latent space
- Reconstruction quality is consistent across datasets, indicating the DSBN decoder's per-domain statistics are absorbing batch effects rather than one dataset dominating
- The OT cost C_ij between encoder Gaussian outputs (mu, sigma) shrinks across datasets, showing Minibatch-UOT alignment is improving

## Limitations

- Storing a dense global OT plan (save_OT=True) requires substantial memory; computing capacity with respect to memory should be ascertained beforehand
- Which hyperparameter combination from the searched grids (lambda in {0.5, 1.0, 5.0}, gamma in {0.5, 1.0}, lambda_s in {0.5, 1.0}) was selected for each dataset is not stated, so defaults may not be optimal
- uniPort's scATAC integration relies on gene activity scores rather than directly modeling regulatory signals

## Evidence

- [results] coupled-VAE with DSBN decoder and Minibatch-UOT loss training: [evidence span withheld — anchor status: not_found]
- [results] DSBN decoder architecture: [evidence span withheld — anchor status: not_found]
- [methods] loss function composition: [evidence span withheld — anchor status: not_found]
- [readme] Run defaults: "**lambda_s**: balanced parameter for common and specific genes. Default: 0.5
+ **lambda_recon**: balanced parameter for reconstruct term. Default: 1.0"
- [abstract] leveraging common and specific genes: "It leverages both highly variable common and dataset-specific genes for integration to handle the heterogeneity across datasets"
