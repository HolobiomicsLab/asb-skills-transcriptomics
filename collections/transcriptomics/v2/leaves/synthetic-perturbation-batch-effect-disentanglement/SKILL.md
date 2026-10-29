---
name: synthetic-perturbation-batch-effect-disentanglement
description: Use when integrating multiple batches of synthetic (dyngen) or real single-cell
  spliced/unspliced data where one batch carries a perturbation (e.g., gene-module
  knockout) and batch effects must be separated from perturbation effects in a joint
  latent space.
when_to_use_negative:
- Input is a single-batch dataset with no batch or perturbation structure — batch_correction=True
  and the batches argument are unnecessary
- Data lacks spliced/unspliced count layers required by the LatentVelo AnnData preparation
  recipes
- Cells form sparse clusters with few observed transient cells connecting trajectories,
  where LatentVelo's assumption of a single initial latent state may mis-assign trajectory
  origins
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: LatentVelo
  role: Batch-corrected variational autoencoder with latent neural ODEs that integrates
    WT and knockout batches in a joint latent space and disentangles batch effects
    from the perturbation
  repo: https://github.com/Spencerfar/LatentVelo
- name: scVelo
  role: Computes the velocity graph on the LatentVelo latent space (scv.tl.velocity_graph
    with vkey='spliced_velocity') and plots 2D velocity streamlines colored by latent_time
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:84db46b9fd25a8b9b714479c23e0e18ed69f6f330e00126d43890f7f67901b9c
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
  - LatentVelo
  - scVelo
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# synthetic-perturbation-batch-effect-disentanglement

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies LatentVelo's batch-corrected VAE to jointly integrate two batches of a dyngen bifurcation dataset where one batch carries a gene-module knockout, disentangling batch effects from the perturbation in a shared latent space while preserving velocity accuracy and integration metrics.

## When to use

Use when integrating multiple batches of synthetic (dyngen) or real single-cell spliced/unspliced data where one batch carries a perturbation (e.g., gene-module knockout) and batch effects must be separated from perturbation effects in a joint latent space.

## When NOT to use

- Input is a single-batch dataset with no batch or perturbation structure — batch_correction=True and the batches argument are unnecessary
- Data lacks spliced/unspliced count layers required by the LatentVelo AnnData preparation recipes
- Cells form sparse clusters with few observed transient cells connecting trajectories, where LatentVelo's assumption of a single initial latent state may mis-assign trajectory origins

## Inputs

- Two-batch dyngen bifurcation AnnData file (example_bifurcation.h5ad) with 'spliced' and 'unspliced' layers, a 'batch' key, and a 'celltype' key

## Outputs

- LatentVelo latent-space AnnData (latent_adata) with spliced_velocity, decoded values, and UMAP embedding
- Batch-correction and velocity metric comparison tables
- Figure panels (figure_4g_4l.png) showing zr latent regulatory dimensions vs latent_time for both batches and velocity streamlines

## How to apply

Load the two-batch dyngen bifurcation AnnData (example_bifurcation.h5ad) and prepare it with ltv.utils.standard_clean_recipe (or ltv.utils.anvi_clean_recipe for the celltype-annotated model) using spliced_key='spliced', unspliced_key='unspliced', batch_key='batch', celltype_key='celltype'. Initialize ltv.models.VAE (or ltv.models.AnnotVAE) with observed gene count, latent_dim, zr_dim, h_dim, batch_correction=True, and batches=2 so the dynamics-based embedding batch corrects both cell states and velocities. Train with ltv.train, recording epochs plus the validation autoencoder (val_ae) and trajectory reconstruction (val_traj) losses, and save parameters to the named parameter output folder. Export with ltv.output_results(gene_velocity=True, decoded=True, embedding='umap'), compute the velocity graph with scv.tl.velocity_graph(latent_adata, vkey='spliced_velocity'), and plot 2D streamlines colored by latent_time. Compile batch-correction and velocity metric tables and plot the latent regulatory dimensions (zr) against latent time for both batches to demonstrate that batch effects are disentangled from the knockout.

## Related tools

- **LatentVelo** (Batch-corrected variational autoencoder with latent neural ODEs that integrates WT and knockout batches in a joint latent space and disentangles batch effects from the perturbation) — https://github.com/Spencerfar/LatentVelo
- **scVelo** (Computes the velocity graph on the LatentVelo latent space (scv.tl.velocity_graph with vkey='spliced_velocity') and plots 2D velocity streamlines colored by latent_time)

## Examples

```
model = ltv.models.VAE(observed=n_genes, latent_dim=latent_dim, zr_dim=zr_dim, h_dim=h_dim, batch_correction=True, batches=2); epochs, val_ae, val_traj = ltv.train(model, adata, batch_size=batch_size); latent_adata, adata = ltv.output_results(model, adata, gene_velocity=True, decoded=True, embedding='umap')
```

## Evaluation signals

- Validation set autoencoder (val_ae) and trajectory reconstruction (val_traj) losses from ltv.train should be recorded and decrease to convergence
- Velocity accuracy metrics on the latent space should be quantitatively preserved relative to single-batch baselines
- Cell-state integration metrics (e.g., scIB-style batch mixing) should confirm batches are mixed in the latent space
- The zr latent regulatory dimensions plotted against latent_time should show batch effects separated from the knockout signal across both batches
- Velocity streamlines colored by latent_time should recover the expected bifurcation structure with correct transition directions

## Limitations

- LatentVelo assumes a single initial latent state from which all cell trajectories start, with each cell reachable by a continuous trajectory
- Sparse cell clusters with few observed transient cells connecting trajectories can cause latent trajectories to not start from expected initial states
- May require hyperparameter adjustment (larger latent_dim, AnnotVAE cell-type-annotated model, or gene subsetting) for some datasets
- Stochastic dynamics and normalization are not addressed; proposed fixes are latent SDEs and normalization factors as latent variables

## Evidence
