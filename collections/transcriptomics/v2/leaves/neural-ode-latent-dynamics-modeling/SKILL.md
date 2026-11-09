---
name: neural-ode-latent-dynamics-modeling
description: Use when estimating RNA velocity on datasets with multiple lineages,
  time-dependent transcription/splicing rates, or batch effects where strict steady-state/linear
  dynamical assumptions fail.
when_to_use_negative:
- When unspliced signal is reliable and a single lineage with steady-state or linear
  dynamics suffices — standard scVelo/Velocyto may be adequate
- When the dataset contains sparse cell clusters with few observed transient cells
  connecting trajectories, latent trajectories may not start from expected initial
  states
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: LatentVelo
  role: Provides ltv.models.VAE/AnnotVAE, ltv.utils.standard_clean_recipe, ltv.train,
    ltv.output_results, and ltv.cell_trajectories for latent neural-ODE dynamics modeling
  repo: https://github.com/Spencerfar/LatentVelo
- name: torchdiffeq
  role: Integrates the structured latent ODEs (torchdiffeq 0.2.2 pinned in the README)
- name: scVelo
  role: Computes and visualizes velocity streamlines on the latent space with scv.tl.velocity_graph
    using vkey='spliced_velocity'
- name: AnnData
  role: Holds the spliced/unspliced layers and carries LatentVelo output results
- name: scanpy
  role: Reads .h5ad inputs (e.g. sc.read('../synthetic_datasets/example_bifurcation.h5ad'))
    used for model fitting
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:ce013f5e29e4d924df8ba04e5f3e816335abbf329687a811fd1727100f1f29d2
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
  - scVelo
  - scanpy
  - torchdiffeq
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# neural-ode-latent-dynamics-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implement a structured latent dynamics VAE (LatentVelo) that embeds spliced/unspliced counts into latent states (zu, zs, zr) whose dynamics are integrated as neural ODEs, enabling lineage-dependent RNA velocity and latent trajectory inference.

## When to use

Use when estimating RNA velocity on datasets with multiple lineages, time-dependent transcription/splicing rates, or batch effects where strict steady-state/linear dynamical assumptions fail.

## When NOT to use

- When unspliced signal is reliable and a single lineage with steady-state or linear dynamics suffices — standard scVelo/Velocyto may be adequate
- When the dataset contains sparse cell clusters with few observed transient cells connecting trajectories, latent trajectories may not start from expected initial states

## Inputs

- AnnData object with spliced and unspliced count layers (e.g. .h5ad such as example_bifurcation.h5ad)
- Optional batch key and celltype key in adata.obs for batch correction or AnnotVAE conditioning

## Outputs

- Trained LatentVelo VAE/AnnotVAE model with structured latent states (zu, zs, zr) and neural-ODE dynamics
- Validation autoencoder loss (val_ae) and trajectory reconstruction loss (val_traj) from ltv.train
- latent_adata AnnData of LatentVelo latent-space results plus velocities (spliced_velocity) for scVelo plotting
- Latent cell trajectories via ltv.cell_trajectories returning z_traj and times

## How to apply

Load an AnnData object with spliced and unspliced layers (e.g. example_bifurcation.h5ad from dyngen) and prepare it with ltv.utils.standard_clean_recipe (spliced_key='spliced', unspliced_key='unspliced', optional batch_key/celltype_key). Instantiate ltv.models.VAE with observed=number_of_genes, latent_dim, zr_dim (latent regulation dimension), and h_dim (conditioning dimension); the latent state is decomposed into zu, zs, and a latent regulatory variable zr whose constant state-dependent parameter h conditions the latent ODEs (integrated with torchdiffeq 0.2.2) to follow a particular lineage branch. For cell-type-annotated conditioning use ltv.models.AnnotVAE with celltypes=number_of_celltypes, and enable batch_correction=True with batches=number_of_batches when needed. Match ODE-integrated latent states to encoder states through the VAE objective and train with ltv.train(model, adata, batch_size, epochs, name), which returns validation autoencoder (val_ae) and trajectory reconstruction (val_traj) losses; then output results with ltv.output_results and compute the velocity graph with scv.tl.velocity_graph on vkey='spliced_velocity'.

## Related tools

- **LatentVelo** (Provides ltv.models.VAE/AnnotVAE, ltv.utils.standard_clean_recipe, ltv.train, ltv.output_results, and ltv.cell_trajectories for latent neural-ODE dynamics modeling) — https://github.com/Spencerfar/LatentVelo
- **torchdiffeq** (Integrates the structured latent ODEs (torchdiffeq 0.2.2 pinned in the README))
- **scVelo** (Computes and visualizes velocity streamlines on the latent space with scv.tl.velocity_graph using vkey='spliced_velocity')
- **AnnData** (Holds the spliced/unspliced layers and carries LatentVelo output results)
- **scanpy** (Reads .h5ad inputs (e.g. sc.read('../synthetic_datasets/example_bifurcation.h5ad')) used for model fitting)

## Examples

```
model = ltv.models.VAE(observed=number_of_genes, latent_dim=latent_dimension, zr_dim=latent_regulation_dimension, h_dim=conditioning_dimension); epochs, val_ae, val_traj = ltv.train(model, adata, batch_size=batch_size, epochs=number_of_epochs, name=parameter_output_folder_name)
```

## Evaluation signals

- ltv.train runs end-to-end on the synthetic bifurcation dataset and returns finite val_ae and val_traj losses
- Model constructor exposes zr_dim and h_dim, confirming the structured zu/zs/zr decomposition is present
- The ODE-integrated latent states match encoder states under the VAE objective, yielding coherent latent trajectories from ltv.cell_trajectories
- scv.tl.velocity_graph computes successfully on latent_adata with vkey='spliced_velocity' and streamlines follow the expected lineage branch (e.g. bifurcation)

## Limitations

- Assumes a single initial latent state from which all cell trajectories start, with each observed cell reachable by a continuous trajectory
- Sparse cell clusters with few observed transient cells can cause latent trajectories to not start from expected initial states (e.g. the disconnected microglia cluster in the dentate gyrus dataset was assigned the lowest latent times)
- Some datasets require hyperparameter adjustment (larger latent dimension, cell-type annotated model, or gene subsetting), and only heuristics exist for choosing hyperparameters
- Stochastic dynamics and normalization are not addressed; potential future fixes are replacing latent ODEs with SDEs and treating normalization factors as latent variables as in scVI

## Evidence
