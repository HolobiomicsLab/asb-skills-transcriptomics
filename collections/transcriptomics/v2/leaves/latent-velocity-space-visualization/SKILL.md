---
name: latent-velocity-space-visualization
description: Use when a trained LatentVelo model (VAE or AnnotVAE) needs its latent-space
  velocities and latent time visualized to verify differentiation trajectories, branching
  structure, or batch-disentangled dynamics.
when_to_use_negative:
- The goal is gene-space velocity visualization from scVelo's dynamical model alone
  — this skill visualizes the LatentVelo latent space via the spliced_velocity key
- The model has not been trained yet; ltv.output_results requires fitted model parameters
  saved by ltv.train
- Cells form sparse clusters with few transient connecting cells, where latent trajectories
  may not start from expected initial states
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: LatentVelo
  role: Exports model results to a latent-space AnnData (ltv.output_results) and infers
    latent trajectories (ltv.cell_trajectories) for visualization
  repo: https://github.com/Spencerfar/LatentVelo
- name: scVelo
  role: Computes the velocity graph (scv.tl.velocity_graph with vkey='spliced_velocity')
    and renders velocity embedding stream plots colored by latent_time
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:8d1136b962bd3069a966e573ed40a2c738fe189addf9bc0dc051faaa3560b038
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
  - GSEApy
  - LatentVelo
  - scVelo
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# latent-velocity-space-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Exports LatentVelo results to the learned latent space AnnData and visualizes inferred RNA velocity as 2D streamlines colored by latent_time, allowing inspection of differentiation topology (e.g., progenitor-to-terminal states in pancreas endocrinogenesis or dyngen bifurcations).

## When to use

Use when a trained LatentVelo model (VAE or AnnotVAE) needs its latent-space velocities and latent time visualized to verify differentiation trajectories, branching structure, or batch-disentangled dynamics.

## When NOT to use

- The goal is gene-space velocity visualization from scVelo's dynamical model alone — this skill visualizes the LatentVelo latent space via the spliced_velocity key
- The model has not been trained yet; ltv.output_results requires fitted model parameters saved by ltv.train
- Cells form sparse clusters with few transient connecting cells, where latent trajectories may not start from expected initial states

## Inputs

- Trained LatentVelo model (ltv.models.VAE or ltv.models.AnnotVAE)
- Prepared AnnData object (.h5ad) with spliced/unspliced layers, e.g. example_bifurcation.h5ad or pancreas endocrinogenesis data
- Embedding choice for output_results (e.g. embedding='umap')

## Outputs

- latent_adata AnnData object containing velocities and latent_time on the LatentVelo latent space UMAP
- 2D velocity embedding stream plot colored by latent_time
- Latent trajectories (z_traj, times) from ltv.cell_trajectories for overlay on latent UMAP

## How to apply

After training with ltv.train (recording val_ae and val_traj losses), call ltv.output_results(model, adata, gene_velocity=True, decoded=True, embedding='umap') to obtain latent_adata containing results projected onto the LatentVelo latent space. Then compute the velocity graph with scv.tl.velocity_graph(latent_adata, vkey='spliced_velocity'), which uses the spliced_velocity key exported by LatentVelo rather than scVelo's native velocity key. Visualize with scv.pl.velocity_embedding_stream(latent_adata, vkey='spliced_velocity', color='latent_time') to draw 2D streamlines colored by latent time, confirming the expected progenitor-to-terminal differentiation topology. For deeper trajectory inspection, ltv.cell_trajectories(model, adata) returns z_traj and times that can be overlaid on the latent UMAP.

## Related tools

- **LatentVelo** (Exports model results to a latent-space AnnData (ltv.output_results) and infers latent trajectories (ltv.cell_trajectories) for visualization) — https://github.com/Spencerfar/LatentVelo
- **scVelo** (Computes the velocity graph (scv.tl.velocity_graph with vkey='spliced_velocity') and renders velocity embedding stream plots colored by latent_time)

## Examples

```
latent_adata, adata = ltv.output_results(model, adata, gene_velocity=True, decoded=True, embedding='umap'); scv.tl.velocity_graph(latent_adata, vkey='spliced_velocity'); scv.pl.velocity_embedding_stream(latent_adata, vkey='spliced_velocity', color='latent_time')
```

## Evaluation signals

- latent_adata contains a 'spliced_velocity' vkey usable by scv.tl.velocity_graph without key errors
- Velocity stream plot on the latent UMAP reproduces the expected progenitor-to-terminal differentiation topology (e.g., Figure 2 pancreas topology, or bifurcation structure for dyngen data)
- latent_time coloring shows monotonically increasing values along inferred streamlines from initial to terminal states
- ltv.output_results returns both latent_adata and the original adata when gene_velocity=True and decoded=True
- ltv.cell_trajectories returns non-empty z_traj and times arrays that overlay coherently on the latent UMAP

## Limitations

- LatentVelo may encounter issues with sparse cell clusters with few observed transient cells connecting trajectories; latent trajectories do not start from expected initial states in such cases
- On the dentate gyrus dataset, LatentVelo incorrectly assigned the lowest latent times to the disconnected microglia cluster, so trajectories incorrectly start from it
- Some datasets require hyperparameter adjustment (larger latent_dim, AnnotVAE, or gene subsetting) before latent-space visualization is informative

## Evidence

- [methods] scv.tl.velocity_graph(latent_adata, vkey='spliced_velocity'): [evidence span withheld — anchor status: not_found]
- [results] confirm the training outputs validation autoencoder and trajectory losses, and that the velocity stream plot on the latent UMAP reproduces the expected progenitor-to-terminal differentiation topology: [evidence span withheld — anchor status: not_found]
