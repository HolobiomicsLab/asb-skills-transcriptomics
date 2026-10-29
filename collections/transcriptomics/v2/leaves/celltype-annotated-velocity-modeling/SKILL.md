---
name: celltype-annotated-velocity-modeling
description: Use when RNA velocity must be inferred on spliced/unspliced scRNA-seq
  data (AnnData) with reliable cell-type annotations, especially when the default
  VAE requires hyperparameter adjustment (larger latent_dim, cell-type-annotated model,
  or gene subsetting).
when_to_use_negative:
- Cell-type labels are unavailable or unreliable — the anvi_clean_recipe celltype_key
  is mandatory for AnnotVAE, so use the standard ltv.models.VAE with standard_clean_recipe
  instead
- Data lacks spliced/unspliced layers in the AnnData object, which LatentVelo requires
- The dataset contains sparse cell clusters with few observed transient cells connecting
  trajectories, where the paper notes the model may still infer generally accurate
  velocity but latent trajectories do not start from expected initial states
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: LatentVelo
  role: Provides ltv.utils.anvi_clean_recipe for celltype-annotated data preparation,
    ltv.models.AnnotVAE model, ltv.train, ltv.output_results, and ltv.cell_trajectories
  repo: https://github.com/Spencerfar/LatentVelo
- name: scVelo
  role: Computes the velocity graph on the latent AnnData (vkey='spliced_velocity')
    and plots 2D velocity embedding streams colored by latent_time
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:bc437ca1669df2a8654c33f1073c594bf74803fd7077eee6b6df4fffa4ded679
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# celltype-annotated-velocity-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs LatentVelo with the celltype-annotated VAE (ltv.models.AnnotVAE) so that annotated cell-type labels condition the learned latent dynamics, improving RNA velocity and trajectory inference when standard hyperparameters underperform or cell types are known.

## When to use

Use when RNA velocity must be inferred on spliced/unspliced scRNA-seq data (AnnData) with reliable cell-type annotations, especially when the default VAE requires hyperparameter adjustment (larger latent_dim, cell-type-annotated model, or gene subsetting).

## When NOT to use

- Cell-type labels are unavailable or unreliable — the anvi_clean_recipe celltype_key is mandatory for AnnotVAE, so use the standard ltv.models.VAE with standard_clean_recipe instead
- Data lacks spliced/unspliced layers in the AnnData object, which LatentVelo requires
- The dataset contains sparse cell clusters with few observed transient cells connecting trajectories, where the paper notes the model may still infer generally accurate velocity but latent trajectories do not start from expected initial states

## Inputs

- AnnData object with spliced and unspliced count layers (e.g. spliced_key='spliced', unspliced_key='unspliced')
- Cell-type annotations referenced by celltype_key='celltype'
- Optional batch annotations (batch_key='batch') for batch-corrected runs
- Hyperparameters: number_of_genes, latent_dimension, latent_regulation_dimension (zr_dim), conditioning_dimension (h_dim), number_of_celltypes, batch_size, number_of_epochs, parameter_output_folder_name

## Outputs

- Trained AnnotVAE parameters saved to the named parameter output folder
- Training curves: epochs, val_ae, val_traj losses from ltv.train
- latent_adata AnnData on the LatentVelo latent space with spliced_velocity and latent_time (from ltv.output_results)
- 2D velocity streamline plots and cell trajectories (z_traj, times from ltv.cell_trajectories)

## How to apply

Prepare the AnnData with ltv.utils.anvi_clean_recipe instead of the standard recipe, which — unlike the optional keys in the standard recipe — must include a celltype_key. Initialize the model as ltv.models.AnnotVAE(observed=number_of_genes, latent_dim=..., zr_dim=latent_regulation_dimension, h_dim=conditioning_dimension, celltypes=number_of_celltypes), optionally adding batch_correction=True and batches=n for multi-batch data. Train with ltv.train, which returns epochs plus validation autoencoder (val_ae) and trajectory reconstruction (val_traj) losses, and save parameters to a named output folder. Export results with ltv.output_results (gene_velocity=True, decoded=True, embedding='umap') to a latent_adata, then compute the velocity graph with scv.tl.velocity_graph(latent_adata, vkey='spliced_velocity') and plot 2D velocity streamlines colored by latent_time. The rationale is that annotated cell-type labels are incorporated as additional conditioning information in the latent dynamics, which the paper notes can rescue cases needing hyperparameter adjustment.

## Related tools

- **LatentVelo** (Provides ltv.utils.anvi_clean_recipe for celltype-annotated data preparation, ltv.models.AnnotVAE model, ltv.train, ltv.output_results, and ltv.cell_trajectories) — https://github.com/Spencerfar/LatentVelo
- **scVelo** (Computes the velocity graph on the latent AnnData (vkey='spliced_velocity') and plots 2D velocity embedding streams colored by latent_time)

## Examples

```
ltv.utils.anvi_clean_recipe(adata, spliced_key='spliced', unspliced_key='unspliced', batch_key='batch', celltype_key='celltype'); model = ltv.models.AnnotVAE(observed=n_genes, latent_dim=20, zr_dim=2, h_dim=5, celltypes=n_celltypes); epochs, val_ae, val_traj = ltv.train(model, adata, batch_size=512, name='anvi_run')
```

## Evaluation signals

- val_ae and val_traj losses returned by ltv.train decrease and stabilize over epochs
- latent_adata contains a spliced_velocity vkey and latent_time covariate consumable by scv.tl.velocity_graph
- Velocity streamlines on the latent UMAP reproduce expected cell-type transitions ordered by latent_time
- For batch-corrected runs, batch effects are disentangled from cell state in the latent regulatory (zr) dimensions when plotted against latent time
- Model file for the run is written to the named parameter output folder passed to ltv.train

## Limitations

- Assumption of a single initial latent state from which all cell trajectories start, with each cell reachable by a continuous trajectory
- May encounter issues with sparse cell clusters with few observed transient cells connecting trajectories; inferred trajectories may not start from expected initial states (e.g. the disconnected microglia cluster in the dentate gyrus dataset got the lowest latent times)
- Some datasets require additional hyperparameter adjustment (larger latent dimension, cell-type-annotated model, or restricting to a subset of genes), and only heuristics exist for hyperparameter choice

## Evidence
