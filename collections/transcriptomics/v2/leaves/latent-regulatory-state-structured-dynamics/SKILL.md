---
name: latent-regulatory-state-structured-dynamics
description: Use when building an RNA velocity model that must capture lineage-dependent
  or time-dependent kinetics on single-cell spliced/unspliced data, e.g. bifurcating
  developmental systems where steady-state or linear dynamical models fail.
when_to_use_negative:
- The dataset has a single dominant lineage with simple time-independent kinetics,
  where linear methods like scVelo's dynamical model already suffice
- Data lacks both spliced and unspliced count layers in AnnData, since LatentVelo
  requires these two layers
- Sparse cell clusters have very few observed transient cells connecting trajectories
  — latent trajectories may not start from expected initial states in such cases
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: LatentVelo
  role: Provides ltv.models.VAE/AnnotVAE with structured latent states (zu, zs, zr)
    and ltv.train for fitting the neural-ODE dynamics
  repo: https://github.com/Spencerfar/LatentVelo
- name: PyTorch
  role: Deep learning framework in which the VAE encoder/decoder and ODE dynamics
    networks are implemented (pytorch 1.11.0)
- name: torchdiffeq
  role: Integrates the structured latent states (zu, zs, zr) as neural ordinary differential
    equations (version 0.2.2)
- name: AnnData
  role: Holds the spliced/unspliced layers and batch/celltype annotations consumed
    by standard_clean_recipe and ltv.train (anndata 0.8.0)
- name: scanpy
  role: Reads the input .h5ad dataset and supports downstream analysis of the trained
    latent space
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:1c37fa31384868d69c66fcf69a39ca328c5180d54ff6206a43e2e87d4245f545
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Spencerfar/LatentVelo
  repo_url: https://github.com/Spencerfar/LatentVelo
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
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# latent-regulatory-state-structured-dynamics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implement LatentVelo's structured latent dynamics VAE, which embeds spliced/unspliced counts into latent states (zu, zs, zr) whose dynamics are integrated as neural ODEs, with a latent regulatory state zr and conditioning variable h enabling lineage-dependent dynamics.

## When to use

Use when building an RNA velocity model that must capture lineage-dependent or time-dependent kinetics on single-cell spliced/unspliced data, e.g. bifurcating developmental systems where steady-state or linear dynamical models fail.

## When NOT to use

- The dataset has a single dominant lineage with simple time-independent kinetics, where linear methods like scVelo's dynamical model already suffice
- Data lacks both spliced and unspliced count layers in AnnData, since LatentVelo requires these two layers
- Sparse cell clusters have very few observed transient cells connecting trajectories — latent trajectories may not start from expected initial states in such cases

## Inputs

- AnnData object with 'spliced' and 'unspliced' count layers (e.g. example_bifurcation.h5ad dyngen synthetic dataset)
- Optional batch_key and celltype_key annotations
- Hyperparameters: number_of_genes, latent_dim, zr_dim, h_dim, number_of_celltypes (AnnotVAE), number_of_batches (batch correction), batch_size, epochs

## Outputs

- Trained LatentVelo VAE/AnnotVAE model with structured latent states (zu, zs, zr)
- Validation autoencoder loss (val_ae) and trajectory reconstruction loss (val_traj) from ltv.train
- Model parameters saved to the folder given by the name argument

## How to apply

Load spliced/unspliced AnnData (e.g. example_bifurcation.h5ad from dyngen) and prepare it with ltv.utils.standard_clean_recipe using spliced_key='spliced' and unspliced_key='unspliced'. Instantiate ltv.models.VAE with observed = number_of_genes, latent_dim, zr_dim (latent regulation dimension), and h_dim (conditioning dimension); the h parameter is a constant state-dependent variable that conditions the latent ODEs governing zr to follow a particular lineage branch, and the structured latent states (zu, zs, zr) are integrated with torchdiffeq 0.2.2. If cell-type labels are available, use ltv.models.AnnotVAE with celltypes = number_of_celltypes instead; batch correction is enabled with batch_correction = True and batches = number_of_batches for either variant. Train with ltv.train(model, adata, batch_size, epochs, name), which returns validation autoencoder (val_ae) and trajectory reconstruction (val_traj) losses; the VAE objective (Equation 13) matches ODE-integrated latent states to encoder states.

## Related tools

- **LatentVelo** (Provides ltv.models.VAE/AnnotVAE with structured latent states (zu, zs, zr) and ltv.train for fitting the neural-ODE dynamics) — https://github.com/Spencerfar/LatentVelo
- **PyTorch** (Deep learning framework in which the VAE encoder/decoder and ODE dynamics networks are implemented (pytorch 1.11.0))
- **torchdiffeq** (Integrates the structured latent states (zu, zs, zr) as neural ordinary differential equations (version 0.2.2))
- **AnnData** (Holds the spliced/unspliced layers and batch/celltype annotations consumed by standard_clean_recipe and ltv.train (anndata 0.8.0))
- **scanpy** (Reads the input .h5ad dataset and supports downstream analysis of the trained latent space)

## Examples

```
model = ltv.models.VAE(observed=number_of_genes, latent_dim=latent_dimension, zr_dim=latent_regulation_dimension, h_dim=conditioning_dimension); epochs, val_ae, val_traj = ltv.train(model, adata, batch_size=batch_size, epochs=number_of_epochs, name=parameter_output_folder_name)
```

## Evaluation signals

- ltv.train runs end-to-end on the synthetic bifurcation dataset without errors
- Returned val_ae and val_traj are both finite numbers
- The trained model's ODE-integrated latent states match encoder states through the VAE objective (Equation 13)
- The inferred regulatory dynamics conditioned on h follow the expected lineage branch in the bifurcation dataset
- Model checkpoint files are written to the folder specified by the name argument

## Limitations

- Assumes a single initial latent state from which all cell trajectories start, with each observed cell reachable by a continuous trajectory
- May encounter issues with sparse cell clusters with few observed transient cells connecting trajectories; latent trajectories may not start from expected initial states even when velocities are generally accurate
- Some datasets require hyperparameter adjustment (larger latent state dimension, cell-type annotated AnnotVAE, or gene subsetting), and only heuristics exist for choosing hyperparameters
- Stochastic dynamics and normalization are not addressed; proposed future fixes are replacing latent ODEs with SDEs and modeling normalization factors as latent variables as in scVI

## Evidence
