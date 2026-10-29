---
name: latent-ode-trajectory-inference
description: Use when a spliced/unspliced AnnData (h5ad) needs directional developmental
  trajectories or latent-time ordering, especially with multiple lineages, time-dependent
  rates, or batch effects that break linear RNA velocity.
when_to_use_negative:
- Data lacks an unspliced layer, since LatentVelo requires both spliced and unspliced
  count layers in AnnData
- Cells are unlikely to be reachable from a single initial state by continuous trajectories,
  e.g. disconnected populations like the microglia cluster in the dentate gyrus dataset
- Only steady-state, single-lineage velocity estimates are needed, where simpler linear
  methods such as scVelo stochastic mode may suffice
edam_operation: http://edamontology.org/operation_2437
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: LatentVelo
  role: 'Core model: VAE/AnnotVAE with neural ODE dynamics; provides ltv.train, ltv.output_results,
    and ltv.cell_trajectories for trajectory inference'
  repo: github.com/Spencerfar/LatentVelo
- name: scVelo
  role: Computes the velocity graph on the latent space (scv.tl.velocity_graph with
    vkey='spliced_velocity') and plots velocity embedding streams; also the stochastic/dynamical
    baseline
  repo: github.com/Spencerfar/LatentVelo
- name: scib
  role: Supplies benchmark metrics (CBDir cross-boundary directedness, ICCoh in-cluster
    coherence) used to validate the inferred dynamics
- name: GSEApy
  role: Runs GO enrichment on genes along inferred trajectories, e.g. validating beta-cell
    trajectories via insulin secretion regulation terms
  repo: zqfang/GSEApy
provenance:
  source_task_ids:
  - task_008
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:a894deba44b69f5f2fb5b8864755e9a447ba3eac198e34a8ad1f06f14bb41f80
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
  - GSEApy
  - LatentVelo
  - numpy
  - pandas
  - scVelo
  - scanpy
  - scib
  - scvi-tools
  - sklearn
  - torchdiffeq
  edam_operation: http://edamontology.org/operation_2437
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.47058823529411764
n_claims: 17
---

# latent-ode-trajectory-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Infer developmental cell trajectories with LatentVelo by embedding cells in a VAE latent space and modeling differentiation dynamics with neural ODEs, so that individual cells get inferred developmental paths (z_traj with times) rather than only local RNA velocity vectors.

## When to use

Use when a spliced/unspliced AnnData (h5ad) needs directional developmental trajectories or latent-time ordering, especially with multiple lineages, time-dependent rates, or batch effects that break linear RNA velocity.

## When NOT to use

- Data lacks an unspliced layer, since LatentVelo requires both spliced and unspliced count layers in AnnData
- Cells are unlikely to be reachable from a single initial state by continuous trajectories, e.g. disconnected populations like the microglia cluster in the dentate gyrus dataset
- Only steady-state, single-lineage velocity estimates are needed, where simpler linear methods such as scVelo stochastic mode may suffice

## Inputs

- AnnData (.h5ad) object with 'spliced' and 'unspliced' count layers
- Optional batch_key and celltype_key annotations (celltype_key required for AnnotVAE)
- Model hyperparameters: latent_dim, zr_dim, h_dim, number of celltypes, batch count

## Outputs

- Latent AnnData from ltv.output_results containing spliced_velocity, latent_time, and UMAP embedding
- Latent trajectories z_traj and times from ltv.cell_trajectories
- Velocity embedding stream plots on the latent UMAP colored by latent_time
- Training diagnostics: epochs, val_ae, val_traj losses

## How to apply

Prepare the AnnData with ltv.utils.standard_clean_recipe (or ltv.utils.anvi_clean_recipe when using the cell-type-annotated AnnotVAE, which requires a celltype_key) using spliced_key='spliced' and unspliced_key='unspliced'. Initialize ltv.models.VAE (or AnnotVAE) with observed gene count, latent_dim, zr_dim (latent regulatory state dimension), h_dim (conditioning dimension), optionally batch_correction=True with the number of batches. Train via ltv.train, recording the validation autoencoder (val_ae) and trajectory (val_traj) reconstruction losses, then run ltv.output_results(model, adata, gene_velocity=True, decoded=True, embedding='umap') to obtain a latent AnnData. Compute the velocity graph with scv.tl.velocity_graph(latent_adata, vkey='spliced_velocity') and extract developmental paths with ltv.cell_trajectories(model, adata), which returns z_traj and times for plotting on the latent UMAP.

## Related tools

- **LatentVelo** (Core model: VAE/AnnotVAE with neural ODE dynamics; provides ltv.train, ltv.output_results, and ltv.cell_trajectories for trajectory inference) — github.com/Spencerfar/LatentVelo
- **scVelo** (Computes the velocity graph on the latent space (scv.tl.velocity_graph with vkey='spliced_velocity') and plots velocity embedding streams; also the stochastic/dynamical baseline) — github.com/Spencerfar/LatentVelo
- **scib** (Supplies benchmark metrics (CBDir cross-boundary directedness, ICCoh in-cluster coherence) used to validate the inferred dynamics)
- **GSEApy** (Runs GO enrichment on genes along inferred trajectories, e.g. validating beta-cell trajectories via insulin secretion regulation terms) — zqfang/GSEApy

## Examples

```
z_traj, times = ltv.cell_trajectories(model, adata); scv.tl.velocity_graph(latent_adata, vkey='spliced_velocity')
```

## Evaluation signals

- ltv.train returns decreasing validation autoencoder (val_ae) and trajectory (val_traj) reconstruction losses
- Velocity embedding stream on the latent UMAP reproduces the expected progenitor-to-terminal differentiation topology (e.g. Figure 2 pancreatic endocrinogenesis)
- Latent trajectory start states match biology — trajectories should not originate from disconnected clusters such as dentate gyrus microglia
- Inferred trajectories validated downstream: GO enrichment of insulin secretion regulation and beta cell development terms, and Pax4-high beta vs Arx-low alpha paths
- On datasets with known directions, CBDir and ICCoh scores outperform scVelo stochastic and dynamical baselines

## Limitations

- Assumes a single initial latent state from which all cell trajectories start and that each cell is reachable by a continuous trajectory
- Sparse cell clusters with few observed transient cells connecting trajectories cause latent trajectories to not start from expected initial states
- On dentate gyrus, LatentVelo assigns lowest latent times to the disconnected microglia cluster, so inferred trajectories incorrectly start there
- Some datasets require hyperparameter adjustment (larger latent_dim, AnnotVAE, or gene subsetting); hyperparameter choice relies only on heuristics
- Stochastic dynamics and normalization are not addressed; proposed fixes are latent SDEs and normalization factors as latent variables as in scVI

## Evidence
