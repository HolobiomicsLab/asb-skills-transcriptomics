---
name: variational-autoencoder-latent-dynamics-inference
description: Use when scRNA-seq AnnData with spliced/unspliced layers exhibits multi-lineage
  or transient dynamics (bifurcation, trifurcation) where linear velocity models fail,
  and latent-space velocities or trajectories are needed.
when_to_use_negative:
- Datasets that are well described by the linear steady-state or dynamical model with
  a single lineage and time-independent rates, where scVelo suffices
- Situations requiring a single initial latent state assumption to be violated in
  a known way, e.g. disconnected sparse clusters like microglia that the model misassigns
  as trajectory start points
- Analyses requiring stochastic dynamics or count normalization, which LatentVelo
  explicitly does not address
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: LatentVelo
  role: Core library providing ltv.utils cleaning recipes, ltv.models.VAE/AnnotVAE,
    ltv.train, ltv.output_results, and ltv.cell_trajectories for latent dynamics modeling
  repo: https://github.com/Spencerfar/LatentVelo
- name: scVelo
  role: Used downstream to compute the velocity graph (scv.tl.velocity_graph, vkey='spliced_velocity')
    and plot velocity embedding streams; also serves as the linear baseline in benchmarks
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:1179a5c684827446b8a2db6922e8c58631a9b739f45328471e6dfe18853653b3
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# variational-autoencoder-latent-dynamics-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a LatentVelo variational autoencoder to embed cells into a dynamics-based latent space and model differentiation with neural ordinary differential equations, yielding latent velocities, latent trajectories, and gene velocities that outperform linear methods like scVelo on complex topologies.

## When to use

Use when scRNA-seq AnnData with spliced/unspliced layers exhibits multi-lineage or transient dynamics (bifurcation, trifurcation) where linear velocity models fail, and latent-space velocities or trajectories are needed.

## When NOT to use

- Datasets that are well described by the linear steady-state or dynamical model with a single lineage and time-independent rates, where scVelo suffices
- Situations requiring a single initial latent state assumption to be violated in a known way, e.g. disconnected sparse clusters like microglia that the model misassigns as trajectory start points
- Analyses requiring stochastic dynamics or count normalization, which LatentVelo explicitly does not address

## Inputs

- AnnData object with 'spliced' and 'unspliced' count layers (e.g. .h5ad files such as example_bifurcation.h5ad dyngen synthetic datasets or scEU_organoid.h5ad)
- Optional batch key and celltype key columns in adata.obs
- Hyperparameters: number_of_genes, latent_dimension, latent_regulation_dimension (zr_dim), conditioning_dimension (h_dim), number_of_batches, number_of_epochs

## Outputs

- Trained LatentVelo VAE model and training loss curves (epochs, val_ae, val_traj)
- latent_adata AnnData with spliced_velocity velocities on the latent UMAP embedding
- Original adata updated with gene velocities and decoder reconstructions
- Latent cell trajectories z_traj and times from ltv.cell_trajectories

## How to apply

Prepare the AnnData with ltv.utils.standard_clean_recipe (spliced_key='spliced', unspliced_key='unspliced'; optional batch_key/celltype_key, or ltv.utils.anvi_clean_recipe for the celltype-annotated model). Initialize the model as ltv.models.VAE (or AnnotVAE with celltypes) specifying observed gene count, latent_dim, zr_dim (latent regulation dimension), and h_dim (conditioning dimension); set batch_correction=True with the number of batches to enable batch correction. Train via ltv.train, monitoring validation autoencoder and trajectory reconstruction losses. Export results with ltv.output_results (gene_velocity=True, decoded=True, embedding='umap'), then compute the velocity graph with scv.tl.velocity_graph(latent_adata, vkey='spliced_velocity'). Optionally extract trajectories with ltv.cell_trajectories to obtain z_traj and latent times for plotting on the latent UMAP.

## Related tools

- **LatentVelo** (Core library providing ltv.utils cleaning recipes, ltv.models.VAE/AnnotVAE, ltv.train, ltv.output_results, and ltv.cell_trajectories for latent dynamics modeling) — https://github.com/Spencerfar/LatentVelo
- **scVelo** (Used downstream to compute the velocity graph (scv.tl.velocity_graph, vkey='spliced_velocity') and plot velocity embedding streams; also serves as the linear baseline in benchmarks)

## Examples

```
model = ltv.models.VAE(observed=n_genes, latent_dim=20, zr_dim=2, h_dim=2); epochs, val_ae, val_traj = ltv.train(model, adata, batch_size=512); latent_adata, adata = ltv.output_results(model, adata, gene_velocity=True); scv.tl.velocity_graph(latent_adata, vkey='spliced_velocity')
```

## Evaluation signals

- Training outputs validation autoencoder (val_ae) and trajectory reconstruction (val_traj) losses that should decrease and converge
- On dyngen synthetic datasets, cosine similarity between ground-truth and estimated velocities, plus CBDir and ICCoh scores on 50 PCs, should exceed scVelo dynamical and stochastic modes (Figure 6A-6C ordering)
- latent_adata must contain the 'spliced_velocity' key so scv.tl.velocity_graph runs without error
- Inferred latent times and trajectory starts should match known biology (e.g. granule lineage transitions on the dentate gyrus dataset); trajectories should not start from disconnected clusters
- Package versions should match the published environment (e.g. torchdiffeq 0.2.2, pytorch 1.11.0, scvelo 0.2.4, anndata 0.8.0)

## Limitations

- Assumes a single initial latent state from which all cell trajectories start and that each observed cell is reachable by a continuous trajectory
- Sparse cell clusters with few observed transient cells can cause latent trajectories to start from unexpected initial states (e.g. the disconnected microglia cluster in the dentate gyrus dataset is assigned the lowest latent times)
- Some datasets require hyperparameter adjustment (larger latent_dim, cell-type annotated AnnotVAE model, or gene subsetting), and only heuristics exist for choosing hyperparameters
- Stochastic dynamics and normalization are not addressed; the authors propose replacing latent ODEs with SDEs and modeling normalization factors as latent variables as in scVI

## Evidence
