---
name: rna-velocity-inference
description: Use when single-cell spliced/unspliced AnnData layers are available and
  you need velocity fields, latent trajectories, batch-corrected dynamics embeddings,
  or fate separation across multiple lineages where steady-state RNA velocity assumptions
  fail.
when_to_use_negative:
- Data lacks unspliced/spliced count layers (LatentVelo requires AnnData with both
  layers), e.g., only a processed expression matrix
- Goal is cell-type annotation or de novo clustering rather than dynamics — scANVI/scTour-style
  tools or clustering is more appropriate
- Dataset contains sparse cell clusters with few transient cells connecting trajectories,
  where inferred trajectories may incorrectly start from disconnected clusters (e.g.,
  microglia in dentate gyrus)
edam_operation: http://edamontology.org/operation_2429
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: LatentVelo
  role: Core VAE + neural ODE model inferring latent velocity, zr regulatory state,
    latent time, and cell trajectories
  repo: https://github.com/Spencerfar/LatentVelo
- name: scVelo
  role: Computes velocity graph on latent_adata (vkey='spliced_velocity') and plots
    velocity embedding streams colored by latent_time
- name: AnnData
  role: Input/output data container holding spliced/unspliced layers and latent results
- name: GSEApy
  role: Gene Ontology enrichment on genes along inferred trajectories (e.g., beta-cell
    trajectory terms such as regulation of insulin secretion)
  repo: https://github.com/zqfang/GSEApy
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:f969f9bed1a79b29defc322cc978dc8b1cbe595f792f0a16ed1dd196ff06e3dc
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
  - scanpy
  edam_operation: http://edamontology.org/operation_2429
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# rna-velocity-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Infer RNA velocity and cell developmental trajectories using LatentVelo, a variational autoencoder that embeds spliced/unspliced counts into a dynamics-based latent space and models differentiation with neural ODEs, including a latent regulatory state (zr) for lineage-dependent dynamics.

## When to use

Use when single-cell spliced/unspliced AnnData layers are available and you need velocity fields, latent trajectories, batch-corrected dynamics embeddings, or fate separation across multiple lineages where steady-state RNA velocity assumptions fail.

## When NOT to use

- Data lacks unspliced/spliced count layers (LatentVelo requires AnnData with both layers), e.g., only a processed expression matrix
- Goal is cell-type annotation or de novo clustering rather than dynamics — scANVI/scTour-style tools or clustering is more appropriate
- Dataset contains sparse cell clusters with few transient cells connecting trajectories, where inferred trajectories may incorrectly start from disconnected clusters (e.g., microglia in dentate gyrus)

## Inputs

- AnnData object (.h5ad) with 'spliced' and 'unspliced' count layers
- Optional batch key (e.g., 'batch') and celltype key (e.g., 'celltype') in adata.obs
- Hyperparameters: number_of_genes, latent_dim, zr_dim, h_dim, batch_size, epochs, number_of_celltypes

## Outputs

- latent_adata AnnData with spliced_velocity, latent_time, and UMAP embedding of the LatentVelo latent space
- Gene velocities (gene_velocity=True) and decoder reconstructions (decoded=True) written back to adata
- Cell trajectories as z_traj and times from ltv.cell_trajectories
- Training log with epochs, val_ae (autoencoder), and val_traj (trajectory reconstruction) losses

## How to apply

Prepare the AnnData with ltv.utils.standard_clean_recipe (spliced_key='spliced', unspliced_key='unspliced', optional batch_key/celltype_key) or ltv.utils.anvi_clean_recipe when using the cell-type annotated model, which requires a celltype_key. Initialize ltv.models.VAE (or ltv.models.AnnotVAE with celltypes=n) specifying observed gene count, latent_dim, zr_dim (latent regulation dimension), and h_dim (conditioning dimension); set batch_correction=True with batches=n for batch correction. Train with ltv.train(model, adata, batch_size=..., epochs=...), monitoring the returned validation autoencoder (val_ae) and trajectory (val_traj) reconstruction losses, then export with ltv.output_results(model, adata, gene_velocity=True, decoded=True, embedding='umap'). Compute the velocity graph with scv.tl.velocity_graph(latent_adata, vkey='spliced_velocity') and visualize with scv.pl.velocity_embedding_stream colored by latent_time; infer developmental paths with ltv.cell_trajectories(model, adata) returning z_traj and times, and inspect the zr dimensions for lineage/fate separation.

## Related tools

- **LatentVelo** (Core VAE + neural ODE model inferring latent velocity, zr regulatory state, latent time, and cell trajectories) — https://github.com/Spencerfar/LatentVelo
- **scVelo** (Computes velocity graph on latent_adata (vkey='spliced_velocity') and plots velocity embedding streams colored by latent_time)
- **AnnData** (Input/output data container holding spliced/unspliced layers and latent results)
- **GSEApy** (Gene Ontology enrichment on genes along inferred trajectories (e.g., beta-cell trajectory terms such as regulation of insulin secretion)) — https://github.com/zqfang/GSEApy

## Examples

```
latent_adata, adata = ltv.output_results(model, adata, gene_velocity=True, decoded=True, embedding='umap'); scv.tl.velocity_graph(latent_adata, vkey='spliced_velocity')
```

## Evaluation signals

- Training returns non-NaN validation losses: val_ae (autoencoder) and val_traj (trajectory reconstruction) should decrease and converge over epochs
- Velocity embedding stream on the latent UMAP reproduces the expected progenitor-to-terminal differentiation topology (e.g., Figure 2 pancreatic endocrinogenesis layout)
- zr latent regulatory dimensions cluster the terminal endocrine states (alpha, beta, delta, epsilon) or separate reprogrammed vs. dead-end cells
- Logistic regression on zr predicts reprogramming outcome (ROC analysis, Figure 3K); CellRank on scVelo velocities recovers only a single (dead-end) terminal state as a control
- GO enrichment (GSEApy) of genes along inferred trajectories recovers expected terms, e.g., regulation of insulin secretion and beta cell development

## Limitations

- Assumes a single initial latent state from which all cell trajectories start, with each observed cell reachable by a continuous trajectory
- May fail on datasets with sparse cell clusters with few observed transient cells connecting trajectories; latent trajectories may not start from expected initial states (e.g., microglia assigned lowest latent times in dentate gyrus)
- Some datasets require hyperparameter adjustment (larger latent_dim, cell-type annotated AnnotVAE model, or gene subsetting); hyperparameter choice is heuristic only
- Stochastic dynamics and normalization are not addressed; proposed future fixes are replacing latent ODEs with SDEs and modeling normalization factors as latent variables as in scVI

## Evidence
