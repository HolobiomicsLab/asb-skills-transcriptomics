---
name: variational-autoencoder-architecture-implementation
description: Use when building a dynamics-based latent-space model of RNA velocity
  from spliced/unspliced AnnData layers, where lineage-dependent dynamics require
  a latent regulatory state (zr) with state-dependent conditioning h.
when_to_use_negative:
- The analysis only needs local RNA velocity vectors under the standard spliced-unspliced
  model — scVelo's linear/dynamical model suffices without a learned latent space
- The dataset lacks spliced/unspliced layers (e.g. only a feature expression matrix
  or already-computed velocities), so the VAE cannot observe the gene dynamics it
  models
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: LatentVelo
  role: Provides the VAE/AnnotVAE model classes (ltv.models.VAE, ltv.models.AnnotVAE),
    data recipes, and ltv.train used to implement and train the structured latent
    VAE
  repo: github.com/Spencerfar/LatentVelo
- name: PyTorch
  role: Deep-learning framework in which the VAE architecture and neural-ODE latent
    dynamics are implemented (run with pytorch 1.11.0)
- name: torchdiffeq
  role: Numerical ODE solver (version 0.2.2) used to integrate the structured latent
    states' dynamics as neural ODEs
- name: AnnData
  role: Annotated data object format (anndata 0.8.0) holding the spliced and unspliced
    count layers consumed by the model
- name: scanpy
  role: Used (sc.read) to load the .h5ad dataset into the AnnData object before data
    preparation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:9a9ddb6fb5d4f6b05e7efa87ceeee295a02360198b1db8b3e4bb3cdc37d9ade4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Spencerfar/LatentVelo
  tools:
  - AnnData
  - LatentVelo
  - PyTorch
  - scanpy
  - torchdiffeq
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# variational-autoencoder-architecture-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implement the structured latent VAE underlying LatentVelo: a PyTorch variational autoencoder with structured latent states (zu, zs, zr) whose dynamics are integrated with torchdiffeq neural ODEs, optionally extended as AnnotVAE with cell-type conditioning and batch correction.

## When to use

Use when building a dynamics-based latent-space model of RNA velocity from spliced/unspliced AnnData layers, where lineage-dependent dynamics require a latent regulatory state (zr) with state-dependent conditioning h.

## When NOT to use

- The analysis only needs local RNA velocity vectors under the standard spliced-unspliced model — scVelo's linear/dynamical model suffices without a learned latent space
- The dataset lacks spliced/unspliced layers (e.g. only a feature expression matrix or already-computed velocities), so the VAE cannot observe the gene dynamics it models

## Inputs

- AnnData object with spliced and unspliced count layers (e.g. .h5ad such as example_bifurcation.h5ad)
- Model hyperparameters: number of genes, latent_dim, zr_dim, h_dim, and optionally celltypes (AnnotVAE) and batches (batch_correction=True)

## Outputs

- Trained LatentVelo VAE/AnnotVAE model with structured latent states (zu, zs, zr) and neural-ODE dynamics
- Validation autoencoder loss (val_ae) and trajectory reconstruction loss (val_traj) returned by ltv.train
- Saved parameters via the name argument of ltv.train

## How to apply

Prepare the AnnData (e.g. example_bifurcation.h5ad from dyngen) with ltv.utils.standard_clean_recipe using spliced_key='spliced' and unspliced_key='unspliced'; use ltv.utils.anvi_clean_recipe if cell-type-annotated conditioning is planned. Implement a VAE class in PyTorch taking constructor arguments observed (number of genes), latent_dim, zr_dim (latent regulation dimension), and h_dim (conditioning dimension), with structured latent states (zu, zs, zr) integrated as neural ODEs using torchdiffeq 0.2.2. For cell-type annotation, add the AnnotVAE variant accepting celltypes = number_of_celltypes; enable batch correction via batch_correction=True with a batches argument in either variant. Match the ODE-integrated latent states to encoder states through the VAE objective (Equation 13) and train with ltv.train(model, adata, batch_size, epochs, name), capturing the returned val_ae (autoencoder) and val_traj (trajectory reconstruction) validation losses. The rationale is that embedding cells in a learned latent space and inferring structured dynamics there — rather than velocity in gene space with strict linear dynamics — enables lineage- and time-dependent transcriptional regulation and batch correction.

## Related tools

- **LatentVelo** (Provides the VAE/AnnotVAE model classes (ltv.models.VAE, ltv.models.AnnotVAE), data recipes, and ltv.train used to implement and train the structured latent VAE) — github.com/Spencerfar/LatentVelo
- **PyTorch** (Deep-learning framework in which the VAE architecture and neural-ODE latent dynamics are implemented (run with pytorch 1.11.0))
- **torchdiffeq** (Numerical ODE solver (version 0.2.2) used to integrate the structured latent states' dynamics as neural ODEs)
- **AnnData** (Annotated data object format (anndata 0.8.0) holding the spliced and unspliced count layers consumed by the model)
- **scanpy** (Used (sc.read) to load the .h5ad dataset into the AnnData object before data preparation)

## Examples

```
model = ltv.models.VAE(observed=number_of_genes, latent_dim=latent_dimension, zr_dim=latent_regulation_dimension, h_dim=conditioning_dimension); epochs, val_ae, val_traj = ltv.train(model, adata, batch_size=batch_size, epochs=number_of_epochs, name=parameter_output_folder_name)
```

## Evaluation signals

- ltv.train runs to completion on the synthetic bifurcation dataset and returns finite val_ae and val_traj losses
- Model instantiation requires all four structural arguments (observed, latent_dim, zr_dim, h_dim), confirming the zu/zs/zr structured latent decomposition is present
- ODE integration with torchdiffeq 0.2.2 produces latent states consistent with encoder states under the VAE objective (Equation 13)
- With batch_correction=True, the model accepts a batches argument and outputs batch-corrected latent cell states and velocities
- Latent trajectories from the trained model recover the expected branch structure of the dyngen bifurcation dataset

## Limitations

- Assumes a single initial latent state from which all cell trajectories start, with each observed cell reachable by a continuous trajectory
- May encounter issues with sparse cell clusters containing few observed transient cells connecting trajectories; latent trajectories may not start from expected initial states (e.g. on the dentate gyrus, the disconnected microglia cluster gets the lowest latent times)
- Some datasets require additional hyperparameter adjustment (larger latent state dimension, cell-type annotated model, or gene subsetting); hyperparameter choice remains heuristic
- Stochastic dynamics and normalization are not addressed; proposed future fixes are replacing latent ODEs with SDEs and modeling normalization factors as latent variables as in scVI

## Evidence
