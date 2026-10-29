---
name: minibatch-unbalanced-optimal-transport-implementation
description: Use when integrating unpaired or heterogeneous single-cell multi-omics
  datasets (e.g., scRNA-seq with scATAC-seq) whose heterogeneity must be handled with
  both common and dataset-specific HVGs via a coupled-VAE with an OT alignment loss.
when_to_use_negative:
- Datasets already share the same feature space and paired cells — vertical integration
  via simple autoencoders (e.g., DCCA, Cobolt) is more appropriate
- Global, dense alignment of the full data is required and memory is limited — storing
  the dense OT plan T needs substantial memory, so avoid save_OT=True
- The task is linear batch correction of homogeneous batches — methods like Harmony
  are designed for that scope, not nonlinear cross-modal OT alignment
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: uniPort
  role: Provides uniport.Run, model.vae.VAE, model.layer.Encoder/Decoder/DSBatchNorm,
    and model.loss.unbalanced_ot implementing the coupled-VAE + Minibatch-UOT training
    loop
  repo: https://github.com/caokai1073/uniport
- name: PyTorch
  role: Framework for building and training the shared probabilistic encoder and DSBN/per-dataset
    decoders with reparameterization
- name: scanpy
  role: Loads the preprocessed PBMC .h5ad AnnData inputs (scanpy.read_h5ad) that feed
    the training loop
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-35094-8
    title: A unified computational framework for single-cell data integration with
      optimal transport
  source_content_hash: sha256:616377dd288560af299e5b62abf375b31479bbd5431a4d863b4ed221ffc7247b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/caokai1073/uniport
  tools:
  - PyTorch
  - scanpy
  - uniPort
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# minibatch-unbalanced-optimal-transport-implementation

## Summary

Implements uniPort's Minibatch-UOT alignment: a shared probabilistic encoder embeds cells from heterogeneous single-cell datasets (e.g., PBMC scATAC-seq and scRNA-seq) into a latent space where the unbalanced optimal transport cost C_ij = ||mu_x - mu_y||^2 + ||sigma_x - sigma_y||^2 between encoder Gaussians is minimized, coupled to reconstruction via DSBN and per-dataset decoders.

## When to use

Use when integrating unpaired or heterogeneous single-cell multi-omics datasets (e.g., scRNA-seq with scATAC-seq) whose heterogeneity must be handled with both common and dataset-specific HVGs via a coupled-VAE with an OT alignment loss.

## When NOT to use

- Datasets already share the same feature space and paired cells — vertical integration via simple autoencoders (e.g., DCCA, Cobolt) is more appropriate
- Global, dense alignment of the full data is required and memory is limited — storing the dense OT plan T needs substantial memory, so avoid save_OT=True
- The task is linear batch correction of homogeneous batches — methods like Harmony are designed for that scope, not nonlinear cross-modal OT alignment

## Inputs

- Preprocessed AnnData matrices per dataset with dataset-specific HVGs (adata1.h5ad, adata2.h5ad)
- Preprocessed AnnData matrix with highly variable common genes (adata_cm.h5ad)
- Configuration: lambda_s, lambda_recon, lambda_kl, lambda_ot, iteration, batch_size, mode ('h'/'v'/'d')

## Outputs

- Trained coupled-VAE model with shared encoder and DSBN/per-dataset decoders
- Cell embeddings in the shared latent space (stored in adata.obs['latent'] by uniport.Run)
- Optional global OT plan matrix when save_OT=True
- Total loss trajectory over 30000 iterations (training_loss_curve.png)

## How to apply

Load preprocessed PBMC AnnData matrices following the adata1.h5ad / adata2.h5ad / adata_cm.h5ad pattern with scanpy, where adata_cm holds k=2000 highly variable common genes and each dataset holds kx=ky=2000 dataset-specific HVGs scaled to 0-1 with MaxAbsScaler. Build the coupled-VAE in PyTorch mirroring model.vae.VAE, model.layer.Encoder, model.layer.Decoder, and model.layer.DSBatchNorm: a shared encoder with reparameterization outputs Gaussian (mu, sigma) latents, while reconstruction passes through a DSBatchNorm decoder plus per-dataset decoders. Compute the Minibatch-UOT cost C_ij = ||mu_x - mu_y||^2 + ||sigma_x - sigma_y||^2 between encoder outputs as implemented in model.loss.unbalanced_ot, and run the IPOT plan update iterations via uniport.Run with defaults lambda_s=0.5, lambda_recon=1.0, lambda_kl=0.5, lambda_ot=1.0, iteration=30000. Log the total loss L = -L_ELBO* + gamma * L_UOT* per iteration and plot the trajectory (training_loss_curve.png) to verify the objective is minimized.

## Related tools

- **uniPort** (Provides uniport.Run, model.vae.VAE, model.layer.Encoder/Decoder/DSBatchNorm, and model.loss.unbalanced_ot implementing the coupled-VAE + Minibatch-UOT training loop) — https://github.com/caokai1073/uniport
- **PyTorch** (Framework for building and training the shared probabilistic encoder and DSBN/per-dataset decoders with reparameterization)
- **scanpy** (Loads the preprocessed PBMC .h5ad AnnData inputs (scanpy.read_h5ad) that feed the training loop)

## Examples

```
import uniport as up; adata = up.Run(adatas=[adata1, adata2], adata_cm=adata_cm, lambda_s=0.5, lambda_recon=1.0, lambda_kl=0.5, lambda_ot=1.0, iteration=30000)
```

## Evaluation signals

- Logged total loss L = -L_ELBO* + gamma * L_UOT* decreases monotonically enough over the 30000 training iterations on the PBMC data
- Resulting adata.obs['latent'] embeddings show aligned cell-type structure across datasets (e.g., via kNN label transfer with sklearn.neighbors.KNeighborsClassifier)
- IPOT plan update iterations (Eqs. 1-7) converge without divergence; hyperparameters stay within the searched grids lambda in {0.5, 1.0, 5.0}, gamma in {0.5, 1.0}, lambdas in {0.5, 1.0}
- Reconstruction through the DSBN decoder reproduces both common-HVG (adata_cm) and dataset-specific HVG structure for each domain

## Limitations

- Storing the dense full OT plan T (save_OT=True) requires substantial memory; users must verify memory capacity before using this option
- Which hyperparameter combination from the searched grids (lambda in {0.5, 1.0, 5.0}, gamma in {0.5, 1.0}, lambdas in {0.5, 1.0}) was selected for each dataset is not stated, so tuning may be needed
- uniPort's scATAC integration relies on gene activity scores rather than directly modeling regulatory signals, unlike GLUE

## Evidence

- [other] shared encoder embeds cells into a latent space where a Minibatch-UOT loss is minimized: [evidence span withheld — anchor status: not_found]
- [abstract] uniPort combines coupled-VAE and Minibatch-UOT: "Here we introduce uniPort, a unified single-cell data integration framework that combines a coupled variational autoencoder (coupled-VAE) and minibatch unbalanced optimal transport (Minibatch-UOT)."
- [readme] default hyperparameters and iterations of uniport.Run: "iteration: max iterations for training. Training one batch_size samples is one iteration. Default: 30000"
- [readme] latent output stored in AnnData: "# integration with both common and dataset-specific genes
# latent representation are stored in adata.obs['latent']"
