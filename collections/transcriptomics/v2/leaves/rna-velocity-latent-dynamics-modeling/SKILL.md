---
name: rna-velocity-latent-dynamics-modeling
description: Use when spliced/unspliced AnnData layers are available and the goal
  is RNA velocity on multi-lineage, time-dependent, or multi-batch data where strict
  steady-state assumptions of Velocyto/scVelo fail, or when batch-integrated velocities
  and latent trajectories are needed.
when_to_use_negative:
- Input lacks spliced/unspliced layers — LatentVelo requires AnnData with both count
  layers
- Cells form sparse clusters with few observed transient cells connecting trajectories,
  or disconnected clusters (e.g., microglia in dentate gyrus) that absorb the assumed
  single initial state
- Stochastic dynamics or normalization must be explicitly modeled — these challenges
  are not addressed by the current latent ODE formulation
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: LatentVelo
  role: Core VAE + neural ODE model for latent-space RNA velocity, batch correction,
    and trajectory inference
  repo: https://github.com/Spencerfar/LatentVelo
- name: scVelo
  role: Computes the velocity graph on the 'spliced_velocity' key and plots 2D velocity
    streamlines
- name: scib
  role: Computes kBET batch-mixing benchmark scores across sequencing batches
- name: scanpy
  role: Utilities for embedding, differential expression along trajectories, and figure
    assembly
- name: GSEApy
  role: GO enrichment analysis of differentially expressed genes along inferred endothelial,
    erythroid, mesenchyme, and neural trajectories
  repo: https://github.com/zqfang/GSEApy
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:5b316520a94041c29efc3579f040c75ae31dce0175d744f754b4a804ac91278b
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
  - GSEApy
  - LatentVelo
  - scVelo
  - scanpy
  - scib
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# rna-velocity-latent-dynamics-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimate RNA velocity in a learned latent space using LatentVelo, which embeds cells with a variational autoencoder and models differentiation dynamics with neural ordinary differential equations, enabling batch correction, cell-type-annotated modeling, and latent trajectory inference on multi-lineage single-cell datasets.

## When to use

Use when spliced/unspliced AnnData layers are available and the goal is RNA velocity on multi-lineage, time-dependent, or multi-batch data where strict steady-state assumptions of Velocyto/scVelo fail, or when batch-integrated velocities and latent trajectories are needed.

## When NOT to use

- Input lacks spliced/unspliced layers — LatentVelo requires AnnData with both count layers
- Cells form sparse clusters with few observed transient cells connecting trajectories, or disconnected clusters (e.g., microglia in dentate gyrus) that absorb the assumed single initial state
- Stochastic dynamics or normalization must be explicitly modeled — these challenges are not addressed by the current latent ODE formulation

## Inputs

- AnnData (.h5ad) object with 'spliced' and 'unspliced' count layers
- Batch annotation key (e.g., sequencing batch) and celltype annotation key
- Optional cell-type labels and experimental sequencing-time information for the AnnotVAE model

## Outputs

- Latent-space AnnData from ltv.output_results with 'spliced_velocity' velocities and UMAP embedding
- Gene velocities and decoded model/trajectory reconstructions (when gene_velocity=True, decoded=True)
- Velocity streamplots from scv.tl.velocity_graph / scv.pl.velocity_embedding_stream
- Latent trajectories (z_traj, times) from ltv.cell_trajectories
- Benchmark metrics: CBDir transition scores, kBET batch-mixing scores, NN velocity cosine similarity; GO enrichment panels

## How to apply

Prepare the AnnData with ltv.utils.standard_clean_recipe for the standard model or ltv.utils.anvi_clean_recipe (spliced_key='spliced', unspliced_key='unspliced', batch_key='batch', celltype_key='celltype') for the celltype-annotated model. Initialize ltv.models.VAE or ltv.models.AnnotVAE with latent_dim (e.g., 70 on mouse gastrulation), zr_dim (e.g., 6), and set batch_correction=True with batches=3 to regularize latent dynamics using sequencing-time information. Train with ltv.train, monitoring validation autoencoder (val_ae) and trajectory reconstruction (val_traj) losses, saving parameters under a named output folder. Export results with ltv.output_results (gene_velocity=True, decoded=True, embedding='umap') and build the velocity graph with scv.tl.velocity_graph on the 'spliced_velocity' key for UMAP streamplots. For large multi-lineage systems like the 20,000-cell mouse gastrulation subset, benchmark batch mixing with scib (kBET), cross-boundary directionness (CBDir), and nearest-neighbor velocity cosine similarity across sequencing batches, and extract trajectories with ltv.cell_trajectories followed by differential expression and GO enrichment with GSEApy.

## Related tools

- **LatentVelo** (Core VAE + neural ODE model for latent-space RNA velocity, batch correction, and trajectory inference) — https://github.com/Spencerfar/LatentVelo
- **scVelo** (Computes the velocity graph on the 'spliced_velocity' key and plots 2D velocity streamlines)
- **scib** (Computes kBET batch-mixing benchmark scores across sequencing batches)
- **scanpy** (Utilities for embedding, differential expression along trajectories, and figure assembly)
- **GSEApy** (GO enrichment analysis of differentially expressed genes along inferred endothelial, erythroid, mesenchyme, and neural trajectories) — https://github.com/zqfang/GSEApy

## Examples

```
model = ltv.models.AnnotVAE(observed=n_genes, latent_dim=70, zr_dim=6, h_dim=h, celltypes=n_celltypes, batch_correction=True, batches=3); epochs, val_ae, val_traj = ltv.train(model, adata, batch_size=512, name='gastrulation'); latent_adata, adata = ltv.output_results(model, adata, gene_velocity=True, decoded=True, embedding='umap')
```

## Evaluation signals

- Validation autoencoder (val_ae) and trajectory reconstruction (val_traj) losses decrease and plateau during ltv.train
- kBET batch-mixing score shows LatentVelo outperforms scVI, scANVI, and unintegrated data on the mouse gastrulation batches
- Nearest-neighbor velocity cosine similarity indicates velocities are highly aligned across the 3 sequencing batches
- CBDir cross-boundary directionness scores show transitions consistent with known gastrulation lineage ordering (endothelial, erythroid, mesenchyme, neural)
- Inferred latent trajectories start from expected initial states on the latent UMAP (watch for disconnected clusters capturing the earliest latent times)

## Limitations

- Assumes a single initial latent state from which all cell trajectories start and that each observed cell is reachable by a continuous trajectory
- May encounter issues with sparse cell clusters with few observed transient cells connecting trajectories; trajectories may not start from expected initial states (e.g., microglia in dentate gyrus)
- Some datasets require hyperparameter adjustment: larger latent dimension, cell-type annotated model (AnnotVAE), or gene subsetting; hyperparameter choice relies on heuristics
- Stochastic dynamics and normalization challenges are not addressed (future directions: latent SDEs; normalization factors as latent variables as in scVI)

## Evidence
