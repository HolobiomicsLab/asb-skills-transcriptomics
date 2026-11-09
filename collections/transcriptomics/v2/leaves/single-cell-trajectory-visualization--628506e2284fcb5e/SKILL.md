---
name: single-cell-trajectory-visualization
description: Use when a latent space model (LatentVelo) has produced spliced_velocity
  and latent trajectories and you need to visualize dynamics — e.g., streamlines colored
  by latent_time, diverging trajectories toward multiple fates, or terminal-state
  comparisons on a UMAP embedding.
when_to_use_negative:
- Input is raw gene-expression count data without a trained latent dynamics model
  — run LatentVelo training first
- You need quantitative terminal-state probabilities rather than qualitative visualization;
  use CellRank terminal-state analysis instead of relying only on stream plots
- Data has disconnected or sparse clusters; latent trajectories may incorrectly start
  from disconnected clusters (e.g., the microglia cluster in the dentate gyrus dataset)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: LatentVelo
  role: Produces the latent AnnData, spliced velocities, and latent trajectories that
    are visualized
  repo: https://github.com/Spencerfar/LatentVelo
- name: scVelo
  role: Computes scv.tl.velocity_graph on the latent space and renders scv.pl.velocity_embedding_stream
    streamlines
- name: scanpy
  role: Provides the UMAP embedding ('umap') used as the visualization canvas for
    latent space plots
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:e8d0029e6b494415010217c29b7e114edc5365c6475f7fbaa347778119d86b8d
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
  - scanpy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# single-cell-trajectory-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Visualize inferred single-cell differentiation dynamics by plotting scVelo velocity streamlines on the LatentVelo latent UMAP and rendering diverging latent cell trajectories, enabling qualitative assessment of fate separation (e.g., reprogrammed vs. dead-end cells).

## When to use

Use when a latent space model (LatentVelo) has produced spliced_velocity and latent trajectories and you need to visualize dynamics — e.g., streamlines colored by latent_time, diverging trajectories toward multiple fates, or terminal-state comparisons on a UMAP embedding.

## When NOT to use

- Input is raw gene-expression count data without a trained latent dynamics model — run LatentVelo training first
- You need quantitative terminal-state probabilities rather than qualitative visualization; use CellRank terminal-state analysis instead of relying only on stream plots
- Data has disconnected or sparse clusters; latent trajectories may incorrectly start from disconnected clusters (e.g., the microglia cluster in the dentate gyrus dataset)

## Inputs

- Trained LatentVelo VAE model (ltv.models.VAE or AnnotVAE)
- Cleaned AnnData object with spliced and unspliced layers (from ltv.utils.standard_clean_recipe)
- latent_adata AnnData output of ltv.output_results containing spliced_velocity and latent_time

## Outputs

- Velocity embedding stream plots on the latent UMAP colored by latent_time
- Latent cell trajectory arrays (z_traj, times) from ltv.cell_trajectories
- Composite figure panels showing diverging trajectories and fate separation (e.g., Figure 3 of the paper)

## How to apply

After training the LatentVelo VAE (ltv.models.VAE with zr_dim for the latent regulatory state), call ltv.output_results(model, adata, gene_velocity=True, decoded=True, embedding='umap') to obtain a latent AnnData object. Compute the velocity graph with scv.tl.velocity_graph(latent_adata, vkey='spliced_velocity') and plot velocity embedding streams colored by 'latent_time' via scv.pl.velocity_embedding_stream. Generate latent trajectories with z_traj, times = ltv.cell_trajectories(model, adata) and plot them on the latent space UMAP to show divergence toward distinct fates (e.g., reprogrammed vs. dead-end cells in the fibroblast reprogramming dataset). Validate by confirming the plots qualitatively reproduce expected figure panels, such as zr separating cell fates and trajectories diverging rather than collapsing to a single terminal state.

## Related tools

- **LatentVelo** (Produces the latent AnnData, spliced velocities, and latent trajectories that are visualized) — https://github.com/Spencerfar/LatentVelo
- **scVelo** (Computes scv.tl.velocity_graph on the latent space and renders scv.pl.velocity_embedding_stream streamlines)
- **scanpy** (Provides the UMAP embedding ('umap') used as the visualization canvas for latent space plots)

## Examples

```
latent_adata, adata = ltv.output_results(model, adata, gene_velocity=True, decoded=True, embedding='umap'); scv.tl.velocity_graph(latent_adata, vkey='spliced_velocity'); z_traj, times = ltv.cell_trajectories(model, adata)
```

## Evaluation signals

- scv.tl.velocity_graph runs without error on latent_adata using vkey='spliced_velocity', confirming the velocity key exists in the output AnnData
- Velocity stream plot shows coherent directional flow consistent with latent_time ordering (cells stream toward increasing latent time)
- Latent trajectories (z_traj) from ltv.cell_trajectories visibly diverge toward reprogrammed and dead-end fates rather than a single path
- Regenerated plots qualitatively match the reference figure panels (e.g., Figure 3K ROC/UMAP panels of the Biddy et al. reprogramming analysis)
- Plot streamlines distinguish fates when colored by cell-type or outcome labels, consistent with zr separating reprogrammed from dead-end cells

## Limitations

- Assumes a single initial latent state from which all cell trajectories start, with each cell reachable by a continuous trajectory
- Sparse cell clusters with few observed transient cells connecting trajectories cause latent trajectories to not start from expected initial states
- On the dentate gyrus dataset, the disconnected microglia cluster is estimated as having the lowest latent times, so inferred trajectories incorrectly start from it

## Evidence
