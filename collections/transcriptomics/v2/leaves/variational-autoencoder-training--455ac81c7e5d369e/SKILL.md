---
name: variational-autoencoder-training
description: Use when single-cell RNA-seq data with spliced and unspliced count layers
  (AnnData .h5ad) requires RNA velocity or trajectory inference in scenarios where
  linear steady-state/dynamical models fail, e.g. multiple lineages, transcriptional
  boosts, or reprogramming bifurcations.
when_to_use_negative:
- Data lacks unspliced counts (spliced-only scRNA-seq), since the model requires both
  spliced and unspliced layers for velocity estimation
- Sparse cell clusters with few transient connecting cells, where the assumption of
  a single initial latent state with continuous trajectories breaks down (e.g. disconnected
  microglia in the dentate gyrus getting incorrectly lowest latent times)
- A simple linear scVelo steady-state or dynamical model already suffices for a single-lineage
  dataset without batch effects or time-dependent rates
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: LatentVelo
  role: 'Core VAE framework: data prep recipes, VAE/AnnotVAE initialization, ltv.train,
    ltv.output_results, ltv.cell_trajectories'
  repo: https://github.com/Spencerfar/LatentVelo
- name: scVelo
  role: Post-hoc velocity graph computation and streamline visualization on the latent
    space (vkey='spliced_velocity')
  repo: https://github.com/Spencerfar/LatentVelo
- name: scanpy
  role: Data loading (sc.read) and AnnData handling for spliced/unspliced layers
- name: anndata
  role: AnnData object format holding spliced/unspliced layers consumed by the VAE
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:9b82968d3becac3eccf7cefb2834f5ff834bf8d49d08e0fe9d39af2cbac62806
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
  - anndata
  - scVelo
  - scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# variational-autoencoder-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a structured variational autoencoder (LatentVelo VAE) that embeds single-cell spliced/unspliced counts into a latent space and models differentiation dynamics with neural ODEs, enabling lineage-resolved RNA velocity, batch correction, and latent regulatory state (zr) inference.

## When to use

Use when single-cell RNA-seq data with spliced and unspliced count layers (AnnData .h5ad) requires RNA velocity or trajectory inference in scenarios where linear steady-state/dynamical models fail, e.g. multiple lineages, transcriptional boosts, or reprogramming bifurcations.

## When NOT to use

- Data lacks unspliced counts (spliced-only scRNA-seq), since the model requires both spliced and unspliced layers for velocity estimation
- Sparse cell clusters with few transient connecting cells, where the assumption of a single initial latent state with continuous trajectories breaks down (e.g. disconnected microglia in the dentate gyrus getting incorrectly lowest latent times)
- A simple linear scVelo steady-state or dynamical model already suffices for a single-lineage dataset without batch effects or time-dependent rates

## Inputs

- AnnData (.h5ad) object with 'spliced' and 'unspliced' count layers (e.g. fibroblast reprogramming dataset from Biddy et al., dentate gyrus, scEU-seq organoids, or dyngen synthetic bifurcation data)
- Optional batch key and celltype key columns in adata.obs
- Hyperparameters: number_of_genes, latent_dim, zr_dim, h_dim, batch_size, number_of_epochs

## Outputs

- Trained LatentVelo VAE model checkpoint (saved via name parameter of ltv.train)
- Training logs: epochs, validation autoencoder loss (val_ae), validation trajectory loss (val_traj)
- latent_adata AnnData with latent space, spliced_velocity, latent_time, and UMAP embedding
- Updated adata with decoded reconstructions and gene velocities (gene_velocity=True, decoded=True)
- Latent trajectories z_traj with times from ltv.cell_trajectories

## How to apply

Prepare the AnnData with ltv.utils.standard_clean_recipe (spliced_key='spliced', unspliced_key='unspliced', optional batch_key='batch'; use ltv.utils.anvi_clean_recipe when celltype labels are annotated). Initialize the VAE with ltv.models.VAE(observed=number_of_genes, latent_dim=latent_dimension, zr_dim=latent_regulation_dimension, h_dim=conditioning_dimension), enabling batch_correction=True with the batch count when needed; for cell-type-annotated data use ltv.models.AnnotVAE. Train with ltv.train(model, adata, batch_size=batch_size, epochs=number_of_epochs), monitoring the returned validation autoencoder (val_ae) and trajectory reconstruction (val_traj) losses to detect convergence or overfitting. Extract results with ltv.output_results(model, adata, gene_velocity=True, decoded=True, embedding='umap') into latent_adata, then compute the velocity graph with scv.tl.velocity_graph(latent_adata, vkey='spliced_velocity') for downstream stream plotting, trajectory inference (ltv.cell_trajectories), and fate analysis. The rationale is that embedding dynamics in a learned latent space with state-dependent regulation (zr) relaxes strict transcription/splicing assumptions and supports lineage-dependent kinetics.

## Related tools

- **LatentVelo** (Core VAE framework: data prep recipes, VAE/AnnotVAE initialization, ltv.train, ltv.output_results, ltv.cell_trajectories) — https://github.com/Spencerfar/LatentVelo
- **scVelo** (Post-hoc velocity graph computation and streamline visualization on the latent space (vkey='spliced_velocity')) — https://github.com/Spencerfar/LatentVelo
- **scanpy** (Data loading (sc.read) and AnnData handling for spliced/unspliced layers)
- **anndata** (AnnData object format holding spliced/unspliced layers consumed by the VAE)

## Examples

```
model = ltv.models.VAE(observed=2000, latent_dim=20, zr_dim=2, h_dim=2); epochs, val_ae, val_traj = ltv.train(model, adata, batch_size=512, epochs=100)
```

## Evaluation signals

- Validation autoencoder loss (val_ae) and validation trajectory reconstruction loss (val_traj) returned by ltv.train decrease and plateau over epochs
- latent_adata contains spliced_velocity, latent_time, and a UMAP embedding; scv.tl.velocity_graph runs without error on vkey='spliced_velocity'
- On the fibroblast reprogramming dataset, the latent regulatory state zr visually separates reprogrammed from dead-end cells on the latent UMAP, and a logistic regression on zr yields a predictive ROC curve
- ltv.cell_trajectories returns diverging z_traj paths toward reprogrammed and dead-end fates, while CellRank on scVelo velocities recovers only a single dead-end terminal state (expected contrast)
- With batch_correction=True, velocities are batch corrected and outperform autoencoder methods without dynamics-aware correction on benchmark metrics

## Limitations

- Assumes a single initial latent state from which all cell trajectories start, with each cell reachable by a continuous trajectory
- May fail on sparse cell clusters with few observed transient cells connecting trajectories; trajectories may not start from the expected initial states
- For some datasets it requires hyperparameter adjustment (larger latent dimension, cell-type-annotated AnnotVAE, or gene subsetting); only heuristics exist for hyperparameter choice
- Stochastic dynamics and normalization are not addressed; proposed future fixes are replacing latent ODEs with SDEs and normalization factors as latent variables as in scVI

## Evidence

- [abstract] VAE + neural ODE design rationale: [evidence span withheld — anchor status: not_found]
